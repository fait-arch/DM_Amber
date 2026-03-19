---
title: Manual de instalación de GROMACS con Enroot en HPC CEDIA
author: Equipo DM_Amber
date: 2026-03-18
---

# Manual de instalación del sistema: GROMACS + Enroot (HPC CEDIA)

Este documento describe la instalación y puesta en marcha de un entorno reproducible para dinámica molecular usando **GROMACS en contenedor Enroot** con acceso a **GPU** en HPC CEDIA.

## 1. Objetivo

Configurar un entorno funcional para:

- Ejecutar GROMACS 2023.2 desde NGC.
- Usar GPU (A100/MIG) dentro del contenedor.
- Trabajar en una carpeta persistente del usuario.
- Continuar simulaciones largas con `checkpoint`.

---

## 2. Requisitos previos

- Acceso a HPC CEDIA: https://hpc.cedia.edu.ec/
- Enroot disponible en el nodo.
- Espacio de trabajo recomendado:

```bash
mkdir -p ~/DM/GROMACS/test/{inputs,outputs,logs}
cd ~/DM/GROMACS/test
```

- Archivos mínimos de entrada:
	- `inputs/receptor.pdb`
	- `inputs/ligand.pdb`

---

## 3. Verificación del nodo (host)

Antes de instalar/construir el entorno:

```bash
cat /etc/os-release
lscpu
cat /proc/meminfo | head
nvidia-smi
```

Si `nvidia-smi` muestra GPU disponible para tu sesión, puedes continuar.

---

## 4. Instalación del contenedor GROMACS con Enroot

### 4.1 Importar imagen oficial desde NGC

```bash
enroot import docker://nvcr.io#hpc/gromacs:2023.2
```

Resultado esperado: archivo `hpc+gromacs+2023.2.sqsh`.

### 4.2 Crear contenedor local

```bash
enroot create --name gmx2023 hpc+gromacs+2023.2.sqsh
```

### 4.3 Verificar que el contenedor existe

```bash
enroot list
```

Debe aparecer `gmx2023`.

---

## 5. Ingreso al entorno de trabajo

Desde tu carpeta de proyecto:

```bash
enroot start --rw --mount $(pwd):/workspace gmx2023 /bin/bash
```

Dentro del contenedor:

```bash
cd /workspace
gmx --version
nvidia-smi
```

---

## 6. Estructura recomendada de trabajo

Dentro de `/workspace`:

```text
inputs/      # PDB, ITP, GRO, MDP
outputs/     # trayectorias y estructuras finales
logs/        # logs de ejecución
```

---

## 7. Configuración base de ejecución (sin Slurm)

### 7.1 Monitoreo de GPU

```bash
watch -n 1 nvidia-smi
```

### ETAPA 1: Preparación y Construcción del Complejo
```
gmx pdb2gmx -f ./inputs/receptor.pdb -o receptor.gro -water tip3p -ignh
```

Copia el receptor base (está en el directorio actual)
```
head -n -1 receptor.gro > complex.gro
```

Pega el ligando (Asumiendo que ligand.gro YA está en /workspace)
```
grep " LIG " ligand.gro >> complex.gro
```

Cierra con la caja del receptor
```
tail -n 1 receptor.gro >> complex.gro
```

Cuenta líneas y actualiza el encabezado automáticamente
```
count=$(expr $(wc -l < complex.gro) - 3)
sed -i "2s/.*/$count/" complex.gro

gmx editconf -f complex.gro -o newbox.gro -c -d 1.2 -bt octahedron
gmx solvate -cp newbox.gro -cs spc216.gro -o solvated.gro -p topol.top
```

Crear configuración vacía
```
touch empty.mdp
```

Generar el archivo binario (ions.tpr)
Si aquí te da error de "ligand.itp not found", es que te falta subir ese archivo.
```
gmx grompp -f empty.mdp -c solvated.gro -p topol.top -o ions.tpr -maxwarn 2
```

---

### ETAPA 2: Minimización de Energia

**5. Definir Caja y Solvatar**
- **Input:** `complex.gro`
- **Output:** `solvated.gro`, `topol.top` (actualizado con agua)    

```
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
```

### ETAPA 2: Calentamiento
```
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
```

### ETAPA 3: Equilibrado de Presión (NPT)
```
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
```

### ETAPA 4: Producción
```
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
```


## 8. Cotinuar

### Continuar una simulación interrumpida

```bash
gmx mdrun -v -deffnm md_0_1 -nb gpu -cpi md_0_1.cpt
```

### 7.3 Ejecutar en background

```bash
nohup gmx mdrun -v -deffnm md_0_1 -nb gpu -cpi md_0_1.cpt > logs/md_run.log 2>&1 &
```

---

## 9. Parámetros objetivo del protocolo

- Temperatura: **310 K**
- Presión: **1 bar**
- Paso de integración: **2 fs**
- Producción: **1 µs**
- Pasos de producción: $5\times10^8$
- Agua: **TIP3P**
- Caja: **octaédrica truncada** con distancia mínima 12 Å
- Iones: neutralización + **0.15 M** NaCl

---

## 10. Problemas comunes y solución

### 10.1 Error de compatibilidad con checkpoint (`.cpt`)

Si cambiaste parámetros importantes (p. ej. barostato), el `checkpoint` anterior puede ser incompatible.

Solución:

1. Mantener consistencia de parámetros entre etapas.
2. Si cambiaste acoplamientos, recompilar desde la etapa anterior y generar nuevo `.cpt`.

### 10.2 Falta `ligand.itp`

Solución:

- Asegura que `ligand.itp` esté en el directorio de trabajo.
- Verifica inclusión en `topol.top`:

```text
#include "ligand.itp"
```

### 10.3 No se usa GPU

Solución:

- Verifica GPU con `nvidia-smi` dentro del contenedor.
- Ejecuta `mdrun` con `-nb gpu`.

---

## 11. Integración opcional con AmberTools (ligandos)

Cuando GROMACS no parametriza bien el ligando, usar AmberTools/ParmEd:

```bash
conda create -n amber_env -c conda-forge ambertools parmed -y
conda activate amber_env
tleap --version
python -c "import parmed; print(parmed.__version__)"
```

Luego convertir topologías AMBER a formato GROMACS según el flujo de trabajo del proyecto.

---

## 12. Checklist final

- [ ] `enroot list` muestra `gmx2023`
- [ ] `gmx --version` funciona dentro del contenedor
- [ ] `nvidia-smi` funciona dentro del contenedor
- [ ] Carpeta de trabajo montada en `/workspace`
- [ ] Entradas mínimas disponibles en `inputs/`
- [ ] Ejecución/continuación de `mdrun` funcional

Con este manual, el sistema queda instalado y listo para iniciar simulaciones de dinámica molecular en HPC CEDIA con Enroot + GROMACS.
