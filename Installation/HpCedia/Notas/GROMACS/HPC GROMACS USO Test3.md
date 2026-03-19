---
aliases:
  - 2026-01-10-01-42-27-HPC-GROMACS-USO-Test3
tags:
  - Enroot
  - HPC
  - GROMACS
  - GPUComputing
summary:
---
Ubicacion de prueba /home/ronie.martinez/DM/GROMACS/test3

Para ingresar 
```
enroot start --rw --mount $(pwd):/workspace3 gmx2023 /bin/bash
```

Comando para ingresar como root en el enroott
```
enroot start -r -w gmx2023
```

Comando para Vaciar test a lo basico
```
ls | grep -v "amber19sb_parmbsc1_opc.ff" | grep -v "inputs" | xargs rm -rf
```

	 Comando para Continuar Simualcion:
```
gmx mdrun -v -deffnm md_0_1 -nb gpu -cpi md_0_1.cpt
```

No logropasar de la topologia por alguna razon no logro llegar a la minimazacion 


```
gmx pdb2gmx -f ./inputs/receptor.pdb -o receptor.gro -water tip3p -ignh

# 1. Copia el receptor base (está en el directorio actual)
head -n -1 receptor.gro > complex.gro

# 2. Pega el ligando (Asumiendo que ligand.gro YA está en /workspace2)
grep " LIG " ligand.gro >> complex.gro

# 3. Cierra con la caja del receptor
tail -n 1 receptor.gro >> complex.gro


---


# Cuenta líneas y actualiza el encabezado automáticamente
count=$(expr $(wc -l < complex.gro) - 3)
sed -i "2s/.*/$count/" complex.gro


---


gmx editconf -f complex.gro -o newbox.gro -c -d 1.2 -bt octahedron
gmx solvate -cp newbox.gro -cs spc216.gro -o solvated.gro -p topol.top


---


# 1. Crear configuración vacía
touch empty.mdp

# 2. Generar el archivo binario (ions.tpr)
# Si aquí te da error de "ligand.itp not found", es que te falta subir ese archivo.
gmx grompp -f empty.mdp -c solvated.gro -p topol.top -o ions.tpr -maxwarn 2


---


# Seleccionamos el grupo 13 (SOL) automáticamente con 'echo'
# Nota: Si tu grupo de solvente no es el 13, mira la lista que sale y cambia el número en el echo.
# Usualmente el agua es el último grupo.
echo "SOL" | gmx genion -s ions.tpr -o solv_ions.gro -p topol.top -pname NA -nname CL -neutral -conc 0.15


---

# Minimización de Energia
cat << EOF > em.mdp
; Minimización de Energía
integrator  = steep     ; Descenso más pronunciado (robusto)
nsteps      = 10000     ; 10k pasos (según el paper)
emtol       = 1000.0    ; Tolerancia de fuerza (kJ/mol/nm)
emstep      = 0.01
pbc         = xyz       ; Condiciones periódicas

; Electrostática y Van der Waals
coulombtype = PME       ; Particle Mesh Ewald (largo alcance)
rcoulomb    = 1.0       ; Corte 1.0 nm
rvdw        = 1.0       ; Corte 1.0 nm
rlist       = 1.0

; Restricciones
constraints = none      ; Dejar que los enlaces vibren para minimizar
define      = -DPOSRES  ; Mantener la proteína 'sujeta' (Position Restraints)
EOF

gmx grompp -f em.mdp -c solv_ions.gro -r solv_ions.gro -p topol.top -o em.tpr -maxwarn 1

gmx mdrun -v -deffnm em


---


# Crear el archivo `nvt.mdp` (Calentamiento)
cat << EOF > nvt.mdp
title       = NVT Heating 0K to 310K
define      =           ; SIN restricciones (según tu paper)
integrator  = sd        ; Langevin thermostat (Paper request)
dt          = 0.002     ; 2 fs
nsteps      = 250000    ; 500 ps
pbc         = xyz

; Output control
nstxout     = 0         ; No guardar coordenadas .trr pesadas
nstvout     = 0
nstfout     = 0
nstlog      = 5000      ; Guardar log cada 10 ps
nstenergy   = 5000      ; Guardar energía cada 10 ps
nstxout-compressed = 5000 ; Guardar trayectoria ligera .xtc

; Rampa de Temperatura (Annealing)
annealing         = single
annealing_npoints = 2
annealing_time    = 0     500
annealing_temp    = 0     310   ; Sube de 0 a 310 K linealmente

; Temperature coupling (Langevin lo maneja, pero definimos grupos)
tc-grps     = System
tau_t       = 1.0       ; Paper: collision frequency 1 ps^-1
ref_t       = 310       ; Temperatura referencia final

; Pressure coupling is off
pcoupl      = no        ; En NVT no hay presión

; Bond constraints
constraints = h-bonds   ; Paper: SHAKE algorithm (H only)
continuation = no       ; Empezamos desde velocidades 0 (generadas)
gen_vel     = yes       ; Generar velocidades iniciales
gen_temp    = 0         ; Temperatura inicial 0 K
gen_seed    = -1

; Non-bonded
cutoff-scheme = Verlet
coulombtype   = PME
rcoulomb      = 1.1     ; Paper: 11 Angstroms
rvdw          = 1.1
EOF

gmx grompp -f md.mdp -c npt.gro -t npt.cpt -p topol.top -o md_0_1.tpr -maxwarn 1

gmx mdrun -v -deffnm md_0_1 -nb gpu


---


# Equilibrado de Presión (NPT)
cat << EOF > npt.mdp
title       = NPT Equilibration 500ps
define      =           ; SIN restricciones (según paper)
integrator  = sd        ; Langevin thermostat
dt          = 0.002     ; 2 fs
nsteps      = 250000    ; 500 ps
pbc         = xyz

; Output control
nstxout     = 0
nstvout     = 0
nstfout     = 0
nstlog      = 5000
nstenergy   = 5000
nstxout-compressed = 5000

; Temperature coupling
tc-grps     = System
tau_t       = 1.0
ref_t       = 310

; Pressure coupling (ACTIVADO)
pcoupl           = C-rescale    ; Estable para equilibrar (Berendsen moderno)
pcoupltype       = isotropic    ; Presión igual en todas direcciones
tau_p            = 5.0
ref_p            = 1.0          ; 1 bar
compressibility  = 4.5e-5
refcoord_scaling = com

; Continuación exacta (desde NVT)
continuation = yes      ; Continuar simulación previa
gen_vel      = no       ; No generar velocidades (leer del checkpoint)

; Constraints
constraints = h-bonds   ; SHAKE/LINCS
cutoff-scheme = Verlet
coulombtype   = PME
rcoulomb      = 1.1
rvdw          = 1.1
EOF

gmx grompp -f npt.mdp -c nvt.gro -r nvt.gro -t nvt.cpt -p topol.top -o npt.tpr -maxwarn 1

gmx mdrun -v -deffnm npt -nb gpu


---


# Crear archivo `md.mdp` (Producción)
cat << EOF > md.mdp
title       = Production MD
integrator  = sd        ; Langevin (según paper)
dt          = 0.002     ; 2 fs
nsteps      = 500000000
                        ; Pon 500000000 para el 1us real.

; Output control (Guardar cada 100 ps)
nstxout     = 0
nstvout     = 0
nstfout     = 0
nstlog      = 50000
nstenergy   = 50000
nstxout-compressed = 50000 ; Archivo .xtc (Trayectoria ligera)

; Temperature coupling
tc-grps     = System
tau_t       = 1.0
ref_t       = 310

; Pressure coupling (Parrinello-Rahman es mejor para Prod)
pcoupl           = Parrinello-Rahman
pcoupltype       = isotropic
tau_p            = 5.0
ref_p            = 1.0
compressibility  = 4.5e-5

; Constraints
constraints      = h-bonds
continuation     = yes    ; Continuar desde NPT
gen_vel          = no     ; No generar velocidades nuevas

; Non-bonded
cutoff-scheme = Verlet
coulombtype   = PME
rcoulomb      = 1.1
rvdw          = 1.1
EOF

gmx grompp -f md.mdp -c npt.gro -t npt.cpt -p topol.top -o md_0_1.tpr -maxwarn 1

gmx mdrun -v -deffnm md_0_1 -nb gpu


---


# Centrar la proteína y empaquetar la unidad
# Cuando pregunte, selecciona:
# 1 (Protein) -> para centrar
# 0 (System)  -> para la salida
echo "1 0" | gmx trjconv -s md_0_1.tpr -f md_0_1.xtc -o md_clean.xtc -pbc mol -center


---

# Análisis de Estabilidad (RMSD)
# Selecciona "Backbone" (4) para el ajuste (least squares fit)
# Selecciona "Backbone" (4) para el cálculo
echo "4 4" | gmx rms -s md_0_1.tpr -f md_clean.xtc -o rmsd.xvg -tu ns


---

# Análisis de Flexibilidad (RMSF)
# Selecciona "C-alpha" (3) para calcular la fluctuación por residuo
echo "3" | gmx rmsf -s md_0_1.tpr -f md_clean.xtc -o rmsf.xvg -res


---

# Puentes de Hidrógeno (H-Bonds)
# Necesitamos el índice de la proteína (1) y el del ligando.
# Al ejecutarlo, mira la lista. Asumamos que Ligando es el grupo 13 (verifica tu lista).
# Selecciona: 1 (Protein) y luego el número de tu Ligando.
gmx hbond -s md_0_1.tpr -f md_clean.xtc -num hbnum.xvg


---


# Extraer el fotograma final (.pdb)
# Extraer el último frame (tiempo 1000 ps)
echo "0" | gmx trjconv -s md_0_1.tpr -f md_clean.xtc -o final_structure.pdb -dump 1000


---


# RMSD vs Time (Solo C-alpha)
# Selecciona el grupo "C-alpha" (usualmente el 3) TANTO para el ajuste (fit) como para el cálculo.
echo "3 3" | gmx rms -s md_0_1.tpr -f md_clean.xtc -o rmsd_calpha.xvg -tu ns


---


# Radius of Gyration vs Time
# _Paper: "decrease in the radius of gyration (RoG) from 19.24 Å to 19.15 Å"_ Esto mide qué tan compacta es la proteína.
# Selecciona el grupo "Protein" (usualmente el 1)
echo "1" | gmx gyrate -s md_0_1.tpr -f md_clean.xtc -o gyrate.xvg


---


# RMSF vs Residue
# _Paper: "calculating root mean squared fluctuations (RMSF) for each residue"_ Esto mide qué partes de la proteína se mueven más.
# Selecciona el grupo "C-alpha" (3) para obtener un valor por residuo
echo "3" | gmx rmsf -s md_0_1.tpr -f md_clean.xtc -o rmsf_residue.xvg -res


---
```


| nsteps      | nsteps(s)  | nsteps(h) | nsteps(d) |
| ----------- | ---------- | --------- | --------- |
| 500 000 000 | 15 000 000 | 4167      | 25        |
| 100         | 3          |           |           |


gmx mdrun -v -deffnm md_0_1 -nb gpu -cpi md_0_1.cpt
nohup gmx mdrun -v -deffnm md_0_1 -nb gpu -cpi md_0_1.cpt > md_run.log 2>&1 &


### Cambios mínimos obligatorios

1. `integrator = md`    
2. Termostato Langevin explícito
3. Justificar campo de fuerza
4. Positional restraints reales
5. Escala temporal realista
6. Añadir clustering + H-bonds




## Paso 1: Instalar Miniconda
Si escribes `conda` en tu terminal y dice "command not found", ejecuta este bloque completo para instalarlo:
```
# 1. Descargar el instalador de Miniconda
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

# 2. Instalarlo (Sigue las instrucciones, di "yes" a todo)
bash Miniconda3-latest-Linux-x86_64.sh -b -p $HOME/miniconda

# 3. Inicializar conda
$HOME/miniconda/bin/conda init bash

# 4. **IMPORTANTE**: Cierra tu terminal y vuelve a abrirla,
# o ejecuta este comando para recargar la configuración:
source ~/.bashrc
```

## Paso 2: Crear un entorno e instalar AmberTools + ParmEd
Ahora que tienes conda, vamos a crear un "entorno" aislado llamado `amber_env` donde instalaremos las herramientas. Esto es mucho más seguro que instalarlo en el sistema general.
```
# 1. Crear el entorno e instalar AmberTools y ParmEd
# (Esto puede tardar unos minutos porque descarga ~500MB)
conda create -n amber_env -c conda-forge ambertools parmed -y

# 2. Activar el entorno para probarlo
conda activate amber_env

# 3. Verificar que funcionan
tleap --version
python -c "import parmed; print('ParmEd version:', parmed.__version__)"
```

## Paso 3: Cómo integrar esto en tu Script Automático
Este es el punto clave. Tu script de GROMACS (el que me mostraste antes) fallará si no sabe dónde buscar `tleap`.
```
#!/bin/bash

# --- BLOQUE DE INICIALIZACIÓN DE CONDA ---
source $HOME/miniconda/bin/activate amber_env



# 1. PREPARACIÓN CON AMBERTOOLS (ff19SB) ...
cat << EOF > build_protein.in
source leaprc.protein.ff19SB
source leaprc.water.opc
mol = loadPdb "./inputs/receptor.pdb"
# Guardamos los archivos intermedios de AMBER
saveAmberParm mol receptor_amber.prmtop receptor_amber.inpcrd
quit
EOF

# Ejecutar LEaP
tleap -f build_protein.in



# 2. CONVERSIÓN A GROMACS (ParmEd)
# Crear script de Python para convertir
cat << EOF > convert_to_gmx.py
import parmed as pmd
# Cargar archivos de AMBER
amber = pmd.load_file('receptor_amber.prmtop', 'receptor_amber.inpcrd')
# Guardar como GROMACS (.gro y .top)
amber.save('receptor.gro', overwrite=True)
amber.save('topol.top', overwrite=True)
EOF

# Ejecutar conversión
python convert_to_gmx.py



# 3. GENERAR RESTRICCIONES DE POSICIÓN (Importante)
# Generar índice para seleccionar la proteína (grupo 1 por defecto)
echo "q" | gmx make_ndx -f receptor.gro -o index.ndx

# Generar posre.itp para la proteína (Heavy atoms)
echo "1" | gmx genrestr -f receptor.gro -n index.ndx -o posre.itp -fc 1000 1000 1000

# Añadir la llamada a posre.itp dentro de topol.top
sed -i '/; Include Position restraint file/d' topol.top  # Borrar si ya existe
sed -i '/\[ molecules \]/i \; Include Position restraint file\n#ifdef POSRES\n#include "posre.itp"\n#endif\n' topol.top
```
