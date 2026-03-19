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
  - ComputaciónCientífica
  - Contenedores
  - GPUComputing
  - OpenMM
  - MMGBSA
  - Linux
  - Ubuntu
  - test
  - Bioinformatica
summary: El documento resume el uso de un entorno HPC en CEDIA para ejecutar simulaciones de dinámica molecular con GROMACS, describiendo el acceso al nodo de cómputo, la verificación de hardware de alto rendimiento (CPU AMD EPYC, gran memoria y GPU NVIDIA A100) y la creación de entornos reproducibles mediante contenedores Enroot sin privilegios root. Se detallan los pasos para importar y ejecutar contenedores CUDA y GROMACS, preparar archivos de entrada y definir protocolos de minimización, equilibración y producción hasta escalas de microsegundos. Además, se contextualiza el trabajo con un protocolo científico basado en literatura (MD, análisis estructural y MM/GBSA) y se discuten limitaciones prácticas de GROMACS para el manejo de campos de fuerza, proponiendo OpenMM como alternativa.
---
ubicacion de prueba /home/ronie.martinez/DM/GROMACS/test
ubicacion de prueba /home/ronie.martinez/DM/GROMACS/test

## Ingreso
 [Login HPC CEDIA](https://hpc.cedia.edu.ec/).
 
![[Pasted image 20251231004323.png]]


![[Pasted image 20251231004428.png]]

## Ejecución de Recursos del Sistema operativo 
```
ronie.martinez@compute-0-0:~$ cat /etc/os-release
NAME="Ubuntu"
VERSION="20.04.5 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.5 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal
ronie.martinez@compute-0-0:~$ lscpu
Architecture:                    x86_64
CPU op-mode(s):                  32-bit, 64-bit
Byte Order:                      Little Endian
Address sizes:                   43 bits physical, 48 bits virtual
CPU(s):                          256
On-line CPU(s) list:             0-255
Thread(s) per core:              2
Core(s) per socket:              64
Socket(s):                       2
NUMA node(s):                    8
Vendor ID:                       AuthenticAMD
CPU family:                      23
Model:                           49
Model name:                      AMD EPYC 7742 64-Core Processor
Stepping:                        0
Frequency boost:                 enabled
CPU MHz:                         3344.367
CPU max MHz:                     2250,0000
CPU min MHz:                     1500,0000
BogoMIPS:                        4492.09
Virtualization:                  AMD-V
L1d cache:                       4 MiB
L1i cache:                       4 MiB
L2 cache:                        64 MiB
L3 cache:                        512 MiB
NUMA node0 CPU(s):               0-15,128-143
NUMA node1 CPU(s):               16-31,144-159
NUMA node2 CPU(s):               32-47,160-175
NUMA node3 CPU(s):               48-63,176-191
NUMA node4 CPU(s):               64-79,192-207
NUMA node5 CPU(s):               80-95,208-223
NUMA node6 CPU(s):               96-111,224-239
NUMA node7 CPU(s):               112-127,240-255
Vulnerability Itlb multihit:     Not affected
Vulnerability L1tf:              Not affected
Vulnerability Mds:               Not affected
Vulnerability Meltdown:          Not affected
Vulnerability Mmio stale data:   Not affected
Vulnerability Retbleed:          Vulnerable
Vulnerability Spec store bypass: Mitigation; Speculative Store Bypass disabled via prctl and seccomp
Vulnerability Spectre v1:        Mitigation; usercopy/swapgs barriers and __user pointer sanitization
Vulnerability Spectre v2:        Mitigation; Retpolines, IBPB conditional, IBRS_FW, STIBP conditional, RSB filling, PBRSB-eIBRS Not affected
Vulnerability Srbds:             Not affected
Vulnerability Tsx async abort:   Not affected
Flags:                           fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm consta
                                 nt_tsc rep_good nopl nonstop_tsc cpuid extd_apicid aperfmperf pni pclmulqdq monitor ssse3 fma cx16 sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx f16c 
                                 rdrand lahf_lm cmp_legacy svm extapic cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw ibs skinit wdt tce topoext perfctr_core perfctr_nb bpext perfc
                                 tr_llc mwaitx cpb cat_l3 cdp_l3 hw_pstate sme ssbd mba sev ibrs ibpb stibp vmmcall fsgsbase bmi1 avx2 smep bmi2 cqm rdt_a rdseed adx smap clflushopt clw
                                 b sha_ni xsaveopt xsavec xgetbv1 xsaves cqm_llc cqm_occup_llc cqm_mbm_total cqm_mbm_local clzero irperf xsaveerptr wbnoinvd arat npt lbrv svm_lock nrip_
                                 save tsc_scale vmcb_clean flushbyasid decodeassists pausefilter pfthreshold avic v_vmsave_vmload vgif umip rdpid overflow_recov succor smca
ronie.martinez@compute-0-0:~$ cat /proc/meminfo | head
MemTotal:       1056649460 kB
MemFree:        933779980 kB
MemAvailable:   937249680 kB
Buffers:          156400 kB
Cached:          5481760 kB
SwapCached:            0 kB
Active:         97825036 kB
Inactive:        5600972 kB
Active(anon):   96814860 kB
Inactive(anon):    88064 kB
ronie.martinez@compute-0-0:~$ nvidia-smi
Wed Dec 31 05:20:11 2025       
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 535.161.08             Driver Version: 535.161.08   CUDA Version: 12.2     |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  NVIDIA A100-SXM4-40GB          On  | 00000000:07:00.0 Off |                   On |
| N/A   24C    P0              42W / 400W |     75MiB / 40960MiB |     N/A      Default |
|                                         |                      |              Enabled |
+-----------------------------------------+----------------------+----------------------+

+---------------------------------------------------------------------------------------+
| MIG devices:                                                                          |
+------------------+--------------------------------+-----------+-----------------------+
| GPU  GI  CI  MIG |                   Memory-Usage |        Vol|      Shared           |
|      ID  ID  Dev |                     BAR1-Usage | SM     Unc| CE ENC DEC OFA JPG    |
|                  |                                |        ECC|                       |
|==================+================================+===========+=======================|
|  0    5   0   0  |              25MiB /  9856MiB  | 28      0 |  2   0    1    0    0 |
|                  |               0MiB / 16383MiB  |           |                       |
+------------------+--------------------------------+-----------+-----------------------+
                                                                                         
+---------------------------------------------------------------------------------------+
| Processes:                                                                            |
|  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
|        ID   ID                                                             Usage      |
|=======================================================================================|
|  No running processes found                                                           |
+---------------------------------------------------------------------------------------+
```


```
nvidia-smi
```

> [!Observación]
> El sistema corresponde a Ubuntu 20.04.5 LTS y el usuario se encuentra ejecutando comandos directamente en un nodo de cómputo (compute-0-0), no en un nodo de login. El nodo dispone de hardware de alto rendimiento, incluyendo 2 sockets AMD EPYC 7742 con un total de 256 hilos de CPU, aproximadamente 1 TB de memoria RAM, y una GPU NVIDIA A100 de 40 GB con MIG habilitado, de la cual al usuario se le ha asignado una partición de ~10 GB de VRAM. Los recursos observados están mayormente libres, lo que indica que el nodo ha sido asignado de forma dedicada o controlada por el scheduler, siendo un entorno adecuado para simulaciones HPC intensivas como GROMACS.

para las pruebas trabajare en la ruta "/home/ronie.martinez/DM/GROMACS"

## Crear un sistema
Enroot: - Es un motor de contenedores ligero diseñado para HPC, **alternativa a Docker** en clusters donde no tienes permisos `root`. Permite ejecutar contenedores con **GPU y acceso al host** sin necesidad de privilegios elevados. Ideal para **aplicaciones científicas**, simulaciones, GROMACS, TensorFlow con CUDA, etc.


## Enroot
### Paso 1: Importar la imagen de Docker
`enroot import docker://nvcr.io#nvidia/cuda`

**Qué hace:**
1. `enroot import` → crea una imagen Enroot a partir de otra fuente.
2. `docker://nvcr.io#nvidia/cuda` → indica que la imagen proviene de **NVIDIA Container Registry (NCR)**, en este caso la imagen oficial de **CUDA**.  
3. Resultado → un archivo `.sqsh` (SquashFS), que es **el contenedor “inmutable”** listo para usar en HPC.

**Notas importantes:**
- No necesitas root para importar.    
- Esto **no instala CUDA en el sistema host**, solo dentro del contenedor.
- El contenedor contiene **todo el stack de CUDA** y drivers necesarios para GPU.    

---
### Paso 2: Crear un contenedor personal
`enroot create --name micudapersonal nvidia+cuda.sqsh`

**Qué hace:**
1. `enroot create` → crea un contenedor **editable a partir de la imagen** `.sqsh`.    
2. `--name micudapersonal` → asigna un nombre lógico para identificarlo.
3. `nvidia+cuda.sqsh` → la imagen base que importaste antes.

**Resultado:**
- Tienes un **contenedor personalizable**, no solo de solo lectura.
- Puedes instalar paquetes, librerías y software sin afectar el host.
- Todavía es un contenedor, pero ahora puedes **montarlo como RW (read-write)**.

---
### Paso 3: Listar contenedores
`enroot list`

**Qué hace:**
- Muestra **todos los contenedores Enroot disponibles en el usuario**, incluyendo:
    - Nombre (`micudapersonal`)
    - Estado (creado, corriendo, detenido)
    - Imagen base
- Es equivalente a `docker ps -a` pero adaptado a HPC sin root.

---
### Paso 4: Iniciar el contenedor con acceso root y RW
```
enroot start --root --rw gmx2023
```

**Qué hace cada opción:**
1. `--root` → abre el contenedor con **usuario root dentro del contenedor**
    - Importante: no tienes root en el host, pero dentro puedes instalar software.
2. `--rw` → permite que el contenedor **sea editable** (instalaciones, configuraciones, scripts).
3. `micudapersonal` → nombre del contenedor a iniciar.


**Qué significa en la práctica:**
- Ahora tienes un **entorno aislado pero completo**, con:
    - Acceso root dentro del contenedor
    - Posibilidad de instalar lo que necesites (GROMACS, Python, módulos científicos)
    - Acceso a GPU (CUDA funciona)
    - Ningún riesgo de modificar el HPC host

---
###  Beneficios de este enfoque
1. **Seguridad:** El host sigue intacto, incluso con root dentro del contenedor.
2. **GPU:** Puedes usar CUDA sin preocuparte por drivers del sistema.
3. **Portabilidad:** Tu contenedor puede moverse entre nodos o clusters HPC.
4. **Flexibilidad:** Instalas tu stack científico sin pedir root al administrador.



> [!Idea]
> Puedo usar enroot para crear un entorno para usar gromacs ?

## Crear Enroot GROMACS
Parce que si primero busque la imaginen de gromacs para la gpu en enroot
[GROMACS ENROOT](http://catalog.ngc.nvidia.com/orgs/hpc/containers/gromacs?version=2023.2).-

Ejemplo rapido de la ejecución de gromacs
[Ejemplo DM GROMACS](http://www.mdtutorials.com/gmx/complex/index.html).
### Paso 0: Acceder y Cargar Módulos
Esto fuerza a Enroot a entender que lo anterior es la dirección del registro.
```
enroot import docker://nvcr.io#hpc/gromacs:2023.2
```
> **Nota:** Fíjate en el `nvcr.io#hpc`. se crea un "hpc+gromacs+2023.2.sqsh" en la ruta /home/ronie.martinez/DM/GROMACS/imagen.

enroot import docker://nvcr.io#hpc/gromacs:2023.2


### Paso 1: Crear el Contenedor
Vas a "desempaquetar" ese archivo `.sqsh` y darle un nombre fácil de recordar (por ejemplo `gmx2023`).

Ejecuta este comando en la misma carpeta donde está el archivo `.sqsh`:
```
enroot create --name gmx2023 hpc+gromacs+2023.2.sqsh
```
> **Nota:** Verificación ejecuta `enroot list`. Deberías ver `gmx2023` en la lista.

> [!Comentario]
> JaJa si salio lol :D

### Paso 3: Preparar el Script de Ejecución (Slurm)
Para correr tu simulación en la HPC, no debes ejecutar el comando directamente en la terminal, sino enviar un trabajo a la cola.

1. Asegúrate de tener tus archivos de entrada (`.tpr`) en una carpeta.
2. Crea un archivo de texto llamado `job_gromacs.sh`:

### Paso 4: Descargar Ligand y receptor
```
wget --no-check-certificate "https://docs.google.com/uc?export=download&id=1sKr70cgHhaaME4Dn4R3TfhfbAnA8TKe-" -O ligand.pdb && \
wget --no-check-certificate "https://docs.google.com/uc?export=download&id=1_siYWXNvetxr023GiVsKSigQIOi_Z04U" -O receptor.pdb
wget --no-check-certificate "https://docs.google.com/uc?export=download&id=1BY2gTIq-Y6OtVHhZsN9Jp34TE1O88NHX" -O SYS_gaff2.prmtop && \
wget --no-check-certificate "https://docs.google.com/uc?export=download&id=15_TatRXoZQkFir-M1ATgDV-fKEamcBHO" -O SYS_gaff2.crd
```


> [!Idea]
> Creo que peudo hacer un script que ejecute odo dentro del enroot y guarde los resultados dentro del archivo 


por ahora cree este resumen para un llms
```
no es necesario crear el Script de Ejecución (Slurm) tengo u aceso al la por intreface grafica para probar pero primero mira esta orientacion "ronie.martinez@compute-0-0:~/DM/GROMACS/test$ pwd,/home/ronie.martinez/DM/GROMACS/test,ronie.martinez@compute-0-0:~/DM/GROMACS/test$ ls -la,total 4,drwxr-xr-x 3 ronie.martinez ronie.martinez  54 dic 31 08:15 .,drwxr-xr-x 4 ronie.martinez ronie.martinez  44 dic 31 07:17 ..,-rw-r--r-- 1 ronie.martinez ronie.martinez 180 dic 31 07:24 explicacion.md,drwxr-xr-x 2 ronie.martinez ronie.martinez  56 dic 31 08:31 inputs,ronie.martinez@compute-0-0:~/DM/GROMACS/test$ cat explicacion.md,Este será un test realizado utilizando como control positivo binimetinib, con código de sesión 75093661 y código SMILES: CN1C=NC2=C1C=C(C(=C2F)NC3=C(C=C(C=C3)Br)F)C(=O)NOCCO.,ronie.martinez@compute-0-0:~/DM/GROMACS/test$ ls -la inputs/
total 488drwxr-xr-x 2 ronie.martinez ronie.martinez     56 Jan  9 06:28 .drwxr-xr-x 3 ronie.martinez ronie.martinez     54 Jan  9 06:25 ..-rw-r--r-- 1 ronie.martinez ronie.martinez   4310 Jan  9 06:23 ligand.pdb-rw-r--r-- 1 ronie.martinez ronie.martinez 488554 Jan  9 06:23 receptor.pdb," y luego ten enc uent estos parametros "Dinámica molecular para un complejo proteína-ligando totalmente solvatado con un tamaño aproximado de 350,000 átomos, en condiciones fisiológicas (310 K y 1 bar), usando un paso de integración de 2 femtosegundos y aplicando control de temperatura y presión durante toda la producción. El objetivo es alcanzar una escala de tiempo de 1 microsegundo, lo que implica 500 millones de pasos de integración, almacenando periódicamente trayectorias y estados para un análisis estructural y energético detallado, con el fin de estudiar la estabilidad conformacional del complejo y la afinidad del ligando a lo largo del tiempo." en si tambien receurda que quiero basarme en esta simulacion:2.1.3 Molecular dynamics simulations.The initial structures of the ligand–receptor complexes for the MD simulation were obtained by molecular docking experiment. The ligands were parametrized in the AMBER 20 Antechamber module70 using the GAFF force field.71 For the kinase, AMBER ff19SB72 force field was employed. The PDB2PQR web-server73 was accessed to determine the protonation state of the side chains of the residues at physiological pH. The ligand–receptor complex was impregnated with pre-equilibrated TIP3P water molecules in a truncated octahedral periodic box. The minimum distance between the edges of the water box and the nearest atom of the complex was set at 12 Å. The system was neutralized by six Cl− anions, followed by the addition of Na+ and Cl− ions according to the recommendations of Machado and Pantano74 to achieve a neutral environment with a salt concentration of 0.15 M.,The minimization–heating–equilibration–production protocol was used. First, the system was subjected to geometry optimization with periodic boundary conditions in all directions. In 10[thin space (1/6-em)]000 optimization cycles (4000 steepest descent + 6000 conjugate gradient), both the protein and the ligand were constrained with harmonic potential k = 10.0 mol−1 Å−2. The system was heated stepwise from 0 K to 310 K in 500 ps without any constraints. After an equilibration period of 500 ps, a productive unconstrained molecular dynamics (MD) simulation of 1 μs duration was started, except for the drug dabrafenib, for which the total simulation time was 300 ns. A time step of 2 fs at constant pressure (1 atm) and temperature (310 K) was used. A langevine thermostat with a collision frequency of 1 ps−1 was used for temperature control. In all simulation protocols, hydrogen atoms were constrained by the SHAKE algorithm.75 The cut-off distance for non-bonded interaction was 11 Å, while for long-range electrostatic interactions, the particle mesh Ewald method76 was used. Periodic boundary conditions were employed in all directions. MD simulations were performed using the molecular dynamics package Amber.77 MD simulations were performed on the Isabella cluster of the University Computing Center, University of Zagreb, Croatia.,2.1.3.1 Binding free energy calculation.The free energies of binding (ΔGbind) between a series of potential drugs (ligands) and the hBRk were calculated according to well-established molecular mechanics/generalized Born surface area (MM/GBSA) protocol.78 The formula,within single-trajectory approach is implemented in the MMPSBA.py script of the AmberTools package. The sum of the bond, angle and dihedral energy (ΔEinternal), electrostatic (ΔEelectrostatic) and van der Waals (ΔEvdW) energies is ΔEMM, the change in MM energy contribution in the gas phase. ΔGsol is a change in solvation free energy, with a polar component (ΔGGB, electrostatic solvation energy) and a non-polar, non-electrostatic solvation contribution (ΔGSA). TΔS is the conformational entropy at binding. The production phase trajectory was divided into 20 segments of 50 ns length. Except for the drug dabrafenib (DAB), where the 300 ns trajectory was divided into 6 segments of 50 ns length. From each segment, 100 geometries were sampled at regular time steps and ΔGbind was calculated. The final ΔGbind was expressed as the mean ± standard deviation for all 20 segments. The calculated MM/GBSA free energies of binding were further decomposed into the specific contributions for each residue. In this way, the contributions of each amino acid side chain to ΔGbind were determined and the nature of the energy change in terms of interaction and solvation energies, or entropic contributions were identified.79 Since we are comparing the energies of the same receptor and of ligands with approximately the same size and number of rotatable bonds, entropic contributions (–TΔS) were neglected.,2.1.3.2 Cluster analysis.The structures of each complex were clustered based on the RMSD of the Cα atoms of each residue using the k-means algorithm. The maximum number of iterations was set to 1000, with randomized initial set of points and sieving set was equal to 10. The frames closest to the centroids of the clusters were identified and considered as representative structures of the conformations. The CPPTRAJ module80 was used to perform the cluster analysis.3.3.1 Molecular dynamics simulation study of compound T 160.In the first 900 ns of the simulation, the RMSD fluctuated around a mean value of 2.44 Å. Then an increase in RMSD was observed, and the mean value for the last 100 ns was 3.05 Å. These changes prompted us to perform a cluster analysis to identify relevant changes in the complex structure. We chose to use the k-means algorithm, where k ranges from 2 to 5. The results were analyzed using the Davies–Bouldin index (DBI), the pseudo-F statistic (pSF), and the ratio between the sum of squares of the regression and the sum of squares of error (SSR/SST) (Table S3†). The analysis confirmed the presence of three relevant conformations with selected parameters collected in Table S4.† For each conformation, a representative structure was identified as a frame closest to the cluster centroid. The distribution of the conformations of the T160:BRAF kinase (T160:hBRk) complex along the trajectory and their RMSD relative to the initial frame are shown in Fig. 6. Along with the increase in the mean RMSD values after the 900 ns mark, a slight decrease in the radius of gyration (RoG) from 19.24 Å to 19.15 Å was observed.In all three conformations, the secondary structure was generally well preserved. However, there were some minor changes in the secondary structure that could affect the geometry of the catalytic pocket Fig. 7. The first region includes residues that are in direct contact with the ligand – Arg462, Ile463, Gly464, and Ser465, while these residues do not have a defined secondary structure in the dominant conformation A, they form a β-sheet in conformations B and C. The second region with considerable changes is the region between Ile592 and Phe 635, which is almost completely without defined secondary structure. In the crystal structure (Fig. S5†)83 and in conformation A, only Pro622 to Ile625 from this region form an H4 α-helix. In conformation B, the H4 helix is absent but two new helices are formed (Glu611-Leu613 and Ile617-Trp619). The H4 helix and the Ile617–Trp619 helix are present in the C conformation.,The flexibility of the residues of the hBRk protein in the complex was examined by calculating root mean squared fluctuations (RMSF) for each residue (Fig. 8). High RMSF values indicated greater fluctuations and flexibility. The averaged RMSF value was 1.55 Å. Almost all of the first 50 residues were having higher than average RMSF values, indicating high flexibility of the N-terminus of the kinase. The highest RMSF value was having Met627, a residue whose side chain pointed toward the solvent, far from the binding site of the ligand. The previously mentioned unstructured region was having two fragments with RMSF values above 3 Å. These two fragments, Ser607–Ser614 and Ile625–Asn631, were not in direct contact with the ligand. It is important to note that the catalytically relevant residues Asp594, Phe595 and Gly596 have RMSF values below 1 Å.,Hydrogen bond analysis revealed that on average 2.11 hydrogen bonds were established between T160 and the kinase. This is on average only 0.02 less than in the complex with T183 and the hBRk. The predominant hydrogen bond formed in almost 75% of the simulation time was between the hydroxyl group of Thr529 (hydrogen acceptor) and the N4 atom of T160 with an average bond length of 1.89 ± 0.08 Å. Changes in the 3D structure of the protein were reflected in the hydrogen bonding patterns in different conformations. For example, in A, T160 formed four hydrogen bonds with three residues (two with Ser465 and one with Thr529 and Asp594). In addition, 15 residues had at least one heavy atom in a 4.0 Å zone away from the non-hydrogen atoms of T160. This illustrated the large number of stabilizing van der Waals interactions. In conformation B, only two hydrogen bonds were formed (with Lys483 and Thr529). Finally, conformation C exhibited three hydrogen bonds. The detailed interactions are shown in Fig. 9.Although the MD simulation for DAB was 300 ns long, some conclusions can be drawn. The flexibility of residues changes slightly when T160 is replaced in the catalytic pocket by dabrafenib, the drug approved for the treatment of melanoma with V600E or V600K mutation (Fig. S6†), with a slight increase in the flexibility of residues Ala544–Lys548 and a decrease in the flexibility of Ile625–Asn631. In addition, dabrafenib has on average one more hydrogen bond than T160, being predominantly (90% of the simulation time) bound to the Gly593 oxygen atom with an average hydrogen bond length of 1.84 Å, which is shorter than the hydrogen bond between Thr529 and T160. Numerous van der Waals interactions together with a higher number of hydrogen bonds could be responsible for the higher binding affinity to hBRk than T160.,3.3.2 Calculation of the free energy of binding.The binding affinity of the hit molecules and the reference drug dabrafenib was estimated using the MM/GBSA approach, in which the free energies of solvation were determined by solving the generalized Born equation. For the complex T160:hBRk, the free energy of binding without entropy contribution was −59.1 ± 2.6 kcal mol−1. T160 had the highest binding potential to hBRk of all the compounds studied. The MM/GBSA binding free energy decomposition was used to identify key residues with dominant contribution to protein–ligand binding. In the study of the SARS-CoV-2 virus, its main protease84 and the NS3 protease of Kyasanur forest disease virus,85 a threshold of −1.5 kcal mol−1 was set for the free energy of binding of a single residue to classify it as a residue with dominant contribution, and the same criteria was applied in the present study. Table 4 lists the residues with dominant contributions for T109, T126, T160, and T183 and dabrafenib. Fig. 10 shows the main interactions based on the decomposition of the binding free energy. In addition to the polar and uncharged Thr529 and the positively charged Lys483, four hydrophobic residues (Val471, Leu505, Leu514, Trp531, and Phe583) contributed significantly to the binding free energy of T160. This finding confirmed the results of previous analyzes on the importance of van der Waals interactions.As expected, the Gly593 residue, which forms the hydrogen bond, has the largest contribution to the binding of dabrafenib (DAB). Other stabilizing interactions are established with the electron-rich residues Phe583 and Trp531, the hydrophobic Val471, Leu514, and Ile527, and the polar Thr529 and Cys532. The free energy of binding for the DAB:hBRk complex was estimated to be −61.7 ± 1.0 kcal mol−1, slightly better than for T160:hBRk.,The results obtained by the analysis of Gaussian-based contour maps suggest that the introduction of the HBA group at the phenyl moiety should increase the activity of the pyrimidine–sulfonamide hybrid derivatives. Comparing compounds T160 and T126, which differ only in the presence of a nitro group at the para position of the phenyl moiety in T160, one can observe a drastic change in the binding free energy for these two compounds. While the binding free energy for T160 is −59.1 ± 2.6 kcal mol−1, it is only −39.7 ± 2.2 kcal mol−1 for T126 (vide infra). The MD simulations revealed that the nitro group sits in a hydrophobic pocket and is surrounded by hydrogen atoms. To gain a deeper insight into the nature of why some structural modifications predicted by Gaussian-based contour maps contribute to higher selectivity, additional MD simulations are needed, including non-substituted pyrimidine–sulfonamide scaffolds, but this is beyond the scope of the present manuscript.,3.3.3 Compound T109, T126 and T183 molecular dynamics simulations study.The same protocol used for the analysis of the T160:hBRk complex was applied to the analysis of the T109:hBRk, T126:hBRk, and T183:hBRk complexes. The stability of the complexes was confirmed by tracking RMSD and RoG along the trajectory (Fig. S7†).,All complexes remained stable but had higher averaged RMSD values compared to the T160:hBRk complex. The RoG values were very similar and range from 18.93 Å to 19.29 Å. The existence of multiple conformations was confirmed by cluster analysis. The cluster analysis data are summarized in Tables S5–S10.† Three complexes shared the presence of an initial short-lived conformation that converts to another conformation within 100 ns. The relative ratios between the populations of conformations A and B are approximately 0.6 to 0.3.,The primary geometric difference between the conformations of T109:hBRk and T126:hBRk, just as with T160:hBRk, was in unstructured fragments. However, in T183:hBRk, conformations A and B showed differences in secondary structure motifs compared to the representative structure of the initial conformation C (Fig. 11). For example, a shift of the H6 α-helix axis was observed. Slightly less pronounced was the shift of the H1 α-helix axis, but it played a larger role in the geometry of the binding site. While the C-terminus of the H1 α-helix had van der Waals contacts with the catalytic Phe595 residue, the N-terminus is tilted toward the unstructured loop region, reducing the volume of the cleft.hBRk residues had higher average RMSF values, 1.81 Å and 1.80 Å, when T183 and T126 were bound in the active pocket, respectively, compared with T109 (1.55 Å). The RMSF diagram (Fig. S8†) shows similar flexibility patterns. Again, the N-terminus and unstructured regions exhibited above average flexibility, while the conserved triad, which was important for catalysis, exhibited low flexibility. T126 had the lowest average number of hydrogen bonds (0.95) between ligand and the receptor. The hydrogen bond connecting the hydroxyl oxygen of Thr529 and the NH group of T126 was present only during 23% of the simulation time, with an average length of 1.91 Å. For comparison, the analogous bond in T160:hBRk was present for about 75% of the simulation time. In T183:hBRk, Gly593 acted as a hydrogen bond acceptor, and the 1.83 ± 0.10 Å long bond was present during 82% of the trajectory. In addition to hydrogen bonding, van der Waals interactions also contributed to free energy of binding. While the free energy of binding of T109 and T183 was almost within 1 kcal mol−1 (−55.5 ± 3.1 kcal mol−1 and −56.6 ± 3.0 kcal mol−1, respectively), the value for T126 is only 39.7 ± 2.2 kcal mol−1. T109, T160, and T183 can serve as excellent starting points for introducing chemical modifications to optimize their potential to inhibit BRAF kinase.
```

## Trabajo 
### Paso 1: Ingresar al Contenedor en Modo Interactivo
En lugar de decirle a Enroot que ejecute `gmx mdrun` y se salga, le diremos que nos abra una terminal (`bash`) dentro del contenedor para trabajar allí.

Ejecuta esto en tu terminal:
```
# --rw: Lectura/escritura
# --mount: Montamos tu carpeta actual en /workspace
# /bin/bash: Esto es clave, le decimos que abra la terminal
enroot start --rw --mount .:/workspace gmx2023 /bin/bash
```

![[Pasted image 20251231041552.png]]
estos son los paso a tomar para la ejecion resutla que gromacs no el campo de fuerza para procesar el receptor y tampoco puede procesar al lingando por que no tiene las herrmaientas nativas, me lleva...
pienso usar el script de openMM de mi vercion de google colab y partir de eso a ver que puedo hacer ocn eso






---
### Paso 2: Crear configuracion

#### 1. Crear el archivo de Minimización (`inputs/em.mdp`)
```
cat <<EOF > inputs/em.mdp
; Minimization parameters
integrator  = steep
nsteps      = 10000
emtol       = 1000.0
emstep      = 0.01
pbc         = xyz
coulombtype = PME
rcoulomb    = 1.0
vdwtype     = Cut-off
rvdw        = 1.0
constraints = h-bonds
EOF
```

#### 2. Crear el archivo de Equilibrio (`inputs/eq.mdp`)
```
cat <<EOF > inputs/eq.mdp
; Equilibration parameters
integrator              = sd
dt                      = 0.002
nsteps                  = 500000    ; 1 ns total
nstout                  = 5000
coulombtype             = PME
rcoulomb                = 1.0
vdwtype                 = Cut-off
rvdw                    = 1.0
tcoupl                  = no
tc-grps                 = System
tau_t                   = 1.0
ref_t                   = 310
pcoupl                  = Berendsen
pcoupltype              = isotropic
tau_p                   = 2.0
ref_p                   = 1.0
compressibility         = 4.5e-5
constraints             = h-bonds
constraint_algorithm    = LINCS
gen_vel                 = yes
gen_temp                = 0
annealing               = single
annealing_npoints       = 2
annealing_time          = 0 500
annealing_temp          = 0 310
EOF
```

#### 3. Crear el archivo de Producción (`inputs/md.mdp`)
```
cat <<EOF > inputs/md.mdp
; Production parameters 1us
integrator              = sd
dt                      = 0.002
nsteps                  = 500000000
nstxout                 = 0
nstvout                 = 0
nstfout                 = 0
nstenergy               = 50000
nstlog                  = 50000
nstxout-compressed      = 50000
compressed-x-grps       = System
coulombtype             = PME
rcoulomb                = 1.0
vdwtype                 = Cut-off
rvdw                    = 1.0
tcoupl                  = no
tc-grps                 = System
tau_t                   = 1.0
ref_t                   = 310
pcoupl                  = Parrinello-Rahman
pcoupltype              = isotropic
tau_p                   = 2.0
ref_p                   = 1.0
constraints             = h-bonds
constraint_algorithm    = LINCS
EOF
```

---

#### 4. Verificar y Ejecutar

Ahora verifica que existen. Ejecuta:

Bash

```
ls -lh inputs/*.mdp
```

_(Deberías ver `em.mdp`, `eq.mdp` y `md.mdp` en la lista)._


### Solución: Aumentar el límite de advertencias (`-maxwarn`)

Simplemente repite el comando `grompp` agregando `-maxwarn 3` al final.

Copia y pega este bloque:
```
# 1. Ensamblar ignorando el error de redondeo de carga
gmx grompp -f inputs/eq.mdp -c inputs/em.gro -p inputs/system.top -o inputs/eq.tpr -maxwarn 3

# 2. Ejecutar el Calentamiento (Ahora sí debería correr)
gmx mdrun -v -deffnm inputs/eq -nb gpu
```

**Lo que sucederá:** Verás que el comando `grompp` volverá a imprimir las advertencias, pero al final dirá algo como _"Too many warnings, but -maxwarn set... generating tpr file"_. Eso significa que generó el archivo `.tpr` exitosamente.

Inmediatamente después, el comando `mdrun` empezará a calentar tu sistema. 
