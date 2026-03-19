---
aliases:
  - 2026-01-07-16-20-18-HPC-CEDIA-with-GROMACS
tags:
  - GROMACS
  - HPC
  - CEDIA
  - Enroot
  - NVIDIAA100
  - DinamicaMolecular
  - ProteniaLigando
  - test
summary:
---
Para ingresar 
```
cd /home/ronie.martinez/DM/GROMACS/test2
enroot start --rw --mount $(pwd):/workspace2 gmx2023 /bin/bash
```

Para monitorear la GPU dentro de tu contenedor
```
watch -n 1 nvidia-smi
```

Para poner continuar con la simulación
gmx mdrun -v -deffnm md_0_1 -nb gpu -cpi md_0_1.cpt