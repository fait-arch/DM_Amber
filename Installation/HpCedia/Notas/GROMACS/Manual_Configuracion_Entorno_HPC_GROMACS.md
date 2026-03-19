---
aliases:
  - Manual configuración entorno HPC GROMACS
tags:
  - HPC
  - CEDIA
  - GROMACS
  - Enroot
  - GPU
  - AMBER
summary: Manual práctico para configurar un entorno reproducible de dinámica molecular proteína-ligando en HPC CEDIA usando Enroot, contenedor GROMACS (NVIDIA NGC) y herramientas opcionales de AmberTools.
---

# Manual de Configuración de Entorno (HPC CEDIA + GROMACS)

> Fecha base: 18 de marzo de 2026  
> Basado en notas de trabajo de `Notas/GROMACS`.

## 1) Objetivo

Dejar listo un entorno reproducible para ejecutar dinámica molecular proteína-ligando con:

- Contenedor `gmx2023` (GROMACS 2023.2)
- Acceso GPU en nodo de cómputo
- Estructura de carpetas estándar
- Validación rápida de funcionamiento

---

## 2) Supuestos

- Ya tienes acceso a HPC CEDIA.
- Trabajas en nodo de cómputo (no login) cuando ejecutes simulaciones.
- Existe `enroot` en el sistema.
- Tienes archivos de entrada mínimos:
  - `inputs/receptor.pdb`
  - `inputs/ligand.pdb`

---

## 3) Estructura recomendada de proyecto

```bash
mkdir -p ~/DM/GROMACS/test_env/{inputs,outputs,logs}
cd ~/DM/GROMACS/test_env
```

Copia tus entradas en `inputs/`.

---

## 4) Validación del nodo y GPU

Ejecuta antes de configurar:

```bash
cat /etc/os-release
lscpu
cat /proc/meminfo | head
nvidia-smi
```

Si `nvidia-smi` muestra una partición MIG activa de A100, el entorno GPU está disponible para tu sesión.

---

## 5) Configuración de Enroot con imagen de GROMACS

### 5.1 Importar imagen desde NVIDIA NGC

```bash
enroot import docker://nvcr.io#hpc/gromacs:2023.2
```

Salida esperada: archivo similar a `hpc+gromacs+2023.2.sqsh`.

### 5.2 Crear contenedor local

```bash
enroot create --name gmx2023 hpc+gromacs+2023.2.sqsh
```

### 5.3 Verificar contenedor

```bash
enroot list
```

Debe aparecer `gmx2023`.

---

## 6) Entrar al entorno de trabajo dentro del contenedor

Desde tu carpeta de proyecto:

```bash
enroot start --rw --mount $(pwd):/workspace gmx2023 /bin/bash
```

Esto monta tu directorio host actual en `/workspace` dentro del contenedor.

Valida dentro del contenedor:

```bash
cd /workspace
gmx --version
nvidia-smi
```

---

## 7) Configuración mínima reproducible de parámetros MD

Crea estos archivos en `/workspace/inputs/`.

### 7.1 `em.mdp` (minimización)

```ini
integrator  = steep
nsteps      = 10000
emtol       = 1000.0
emstep      = 0.01
pbc         = xyz
coulombtype = PME
rcoulomb    = 1.1
rvdw        = 1.1
constraints = none
```

### 7.2 `nvt.mdp` (calentamiento hasta 310 K)

```ini
integrator              = sd
dt                      = 0.002
nsteps                  = 250000
pbc                     = xyz
annealing               = single
annealing_npoints       = 2
annealing_time          = 0 500
annealing_temp          = 0 310
tc-grps                 = System
tau_t                   = 1.0
ref_t                   = 310
pcoupl                  = no
constraints             = h-bonds
gen_vel                 = yes
gen_temp                = 0
cutoff-scheme           = Verlet
coulombtype             = PME
rcoulomb                = 1.1
rvdw                    = 1.1
```

### 7.3 `npt.mdp` (equilibración a 1 bar)

```ini
integrator              = sd
dt                      = 0.002
nsteps                  = 250000
pbc                     = xyz
tc-grps                 = System
tau_t                   = 1.0
ref_t                   = 310
pcoupl                  = C-rescale
pcoupltype              = isotropic
tau_p                   = 5.0
ref_p                   = 1.0
compressibility         = 4.5e-5
constraints             = h-bonds
continuation            = yes
gen_vel                 = no
cutoff-scheme           = Verlet
coulombtype             = PME
rcoulomb                = 1.1
rvdw                    = 1.1
```

### 7.4 `md.mdp` (producción)

```ini
integrator              = sd
dt                      = 0.002
nsteps                  = 500000000
nstlog                  = 50000
nstenergy               = 50000
nstxout-compressed      = 50000
tc-grps                 = System
tau_t                   = 1.0
ref_t                   = 310
pcoupl                  = Parrinello-Rahman
pcoupltype              = isotropic
tau_p                   = 5.0
ref_p                   = 1.0
compressibility         = 4.5e-5
constraints             = h-bonds
continuation            = yes
gen_vel                 = no
cutoff-scheme           = Verlet
coulombtype             = PME
rcoulomb                = 1.1
rvdw                    = 1.1
```

> Nota: Para pruebas rápidas, reduce `nsteps` en `md.mdp`.

---

## 8) Validación funcional rápida del entorno

Dentro de `/workspace` ejecuta:

```bash
ls -lh inputs/*.mdp
```

Si se listan correctamente, el entorno está listo para fase de preparación/ejecución.

Monitoreo recomendado durante ejecución:

```bash
watch -n 1 nvidia-smi
```

Continuar una corrida interrumpida:

```bash
gmx mdrun -v -deffnm md_0_1 -nb gpu -cpi md_0_1.cpt
```

---

## 9) Componente AMBER (opcional, cuando falte parametrización del ligando)

Úsalo cuando GROMACS no pueda generar correctamente topología del ligando.

```bash
# En host o contenedor con conda
conda create -n amber_env -c conda-forge ambertools parmed -y
conda activate amber_env
tleap --version
python -c "import parmed; print(parmed.__version__)"
```

Flujo recomendado:

1. Parametrizar ligando con GAFF/GAFF2 (AmberTools/Antechamber o ACPYPE).
2. Exportar topología compatible (`ligand.itp`, `ligand.gro` o conversión equivalente).
3. Incluir en `topol.top` y continuar con GROMACS.

---

## 10) Errores comunes y solución

### Error A: conflicto con checkpoint al pasar a producción

Síntoma: falla `grompp` usando `-t npt.cpt` por cambios de acoplamiento/barostato.

Solución:

- Mantén consistencia entre etapas (termóstato/baróstato).
- Si cambias parámetros estructurales de acoplamiento, recompila etapa previa y regenera checkpoint compatible.

### Error B: `ligand.itp not found`

Solución:

- Verifica que `ligand.itp` esté en el directorio de trabajo.
- Confirma `#include "ligand.itp"` en `topol.top`.

### Error C: no hay uso de GPU

Solución:

- Comprueba `nvidia-smi` dentro del contenedor.
- Ejecuta `gmx mdrun ... -nb gpu`.

---

## 11) Checklist final

- [ ] `enroot list` muestra `gmx2023`
- [ ] `gmx --version` funciona dentro del contenedor
- [ ] `nvidia-smi` visible dentro del contenedor
- [ ] Existe carpeta `inputs/` con receptor y ligando
- [ ] Existen `em.mdp`, `nvt.mdp`, `npt.mdp`, `md.mdp`
- [ ] Se puede iniciar/continuar `mdrun` con GPU

---

## 12) Referencia de parámetros objetivo del estudio

- Temperatura: 310 K
- Presión: 1 bar
- Paso de integración: 2 fs
- Duración objetivo producción: 1 µs
- Total de pasos producción: $5\times10^8$
- Caja: octaédrica truncada, distancia mínima 12 Å
- Solvente: TIP3P
- Iones: neutralización + 0.15 M NaCl

Con esto el entorno queda configurado para reproducir el protocolo de trabajo descrito en tus notas.
