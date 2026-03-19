---
aliases:
  - 2026-01-23-10-41-30-Sin-título
tags:
  - 
summary: Escribe un resumen aquí...
---
## Ejecucion
Para ingresar 
```
enroot start --rw --mount $(pwd):/workspace2 gmx2023 /bin/bash
```

Comando para Continuar Simualcion:
```
gmx mdrun -v -deffnm md_0_1 -nb gpu -cpi md_0_1.cpt
```


## Monitoreo
Para monitorear la GPU dentro de tu contenedor
```
watch -n 1 nvidia-smi
```



## Evidencia

### Vista sin ejecución
![[Notas/GROMACS/img/Pasted image 20260123104416.png]]

### Vista con ejecución
![[Notas/GROMACS/img/Pasted image 20260123104724.png]]