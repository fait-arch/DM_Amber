---
aliases:
  - 2026-01-09-00-34-15-HPC-GROMACS-USO-TEST2
tags:
  - GROMACS
  - AMBER
  - DinamicaMolecular
  - ProteinaLigando
  - HPC
  - Bioinformatica
  - QuimicaComputacional
  - TIP3P
  - NPT
  - ParrinelloRahman
  - MMGBSA
  - RMSD
  - Linux
  - GPUComputing
  - Enroot
  - CEDIA
  - ProteniaLigando
  - test
summary: Se intenta ejecutar una simulación de dinámica molecular de 1 µs de un complejo proteína-ligando (~350 000 átomos) con GROMACS y GPU en /home/ronie.martinez/DM/GROMACS/test2. Aunque el sistema fue minimizado y equilibrado correctamente (eq.gro, eq.cpt), la producción falla por un conflicto de compatibilidad al cambiar el barostato de Berendsen a Parrinello-Rahman. La solución propuesta es reiniciar el flujo de trabajo, siguiendo un protocolo reproducible AMBER/GROMACS para analizar estabilidad y afinidad del complejo.Escribe un resumen aquí...Se intenta ejecutar una simulación de dinámica molecular de 1 µs de un complejo proteína-ligando (~350 000 átomos) con GROMACS y GPU en /home/ronie.martinez/DM/GROMACS/test2. Aunque el sistema fue minimizado y equilibrado correctamente (eq.gro, eq.cpt), la producción falla por un conflicto de compatibilidad al cambiar el barostato de Berendsen a Parrinello-Rahman. La solución propuesta es reiniciar el flujo de trabajo, siguiendo un protocolo reproducible AMBER/GROMACS para analizar estabilidad y afinidad del complejo.
---
ubicacion de prueba /home/ronie.martinez/DM/GROMACS/test2

Para ingresar 
```
enroot start --rw --mount $(pwd):/workspace2 gmx2023 /bin/bash
```

Para monitorear la GPU dentro de tu contenedor
```
watch -n 1 nvidia-smi
```
- `watch -n 1` ejecuta el comando cada 1 segundo    
- Así ves el uso de GPU en **tiempo real**

Como inciar el sistema:

Estoy en la **Fase de Producción** (la simulación real de 1 microsegundo) para estudiar la estabilidad de mi complejo proteína-ligando de 350,000 átomos. Ya logré **minimizar y equilibrar** el sistema exitosamente, y tengo los archivos `eq.gro` y `eq.cpt` listos, lo cual fue el paso más difícil. El problema ocurre porque **GROMACS detecta un conflicto** cuando intento usar el archivo de punto de control (`-t eq.cpt`) junto con los nuevos parámetros de producción (`md.mdp`). Esto sucede porque cambié el barostato (de Berendsen a Parrinello-Rahman), lo que hace que el archivo de control previo sea incompatible con la nueva configuración, bloqueando la creación del archivo de arranque (`.tpr`).



![[Pasted image 20251231041552.png]]


> [!Idea]
> Empesare desde cero creadnolas cosas y ojo en la ruta /home/ronie.martinez/DM/GROMACS/test2

```
Crea una **guía paso a paso para preparar y ejecutar una simulación de dinámica molecular de un complejo ligando-receptor usando GROMACS y AMBER**, detallando en cada etapa los **archivos de entrada y salida**. Todo está organizado para que el flujo de trabajo sea claro y reproducible.
Ten en cuenta que no es necesario crear el Script de Ejecución (Slurm) tengo u aceso al la por intreface grafica para probar pero primero mira esta orientacion "ronie.martinez@compute-0-0:~/DM/GROMACS/test$ pwd,/home/ronie.martinez/DM/GROMACS/test,ronie.martinez@compute-0-0:~/DM/GROMACS/test$ ls -la,total 4,drwxr-xr-x 3 ronie.martinez ronie.martinez  54 dic 31 08:15 .,drwxr-xr-x 4 ronie.martinez ronie.martinez  44 dic 31 07:17 ..,-rw-r--r-- 1 ronie.martinez ronie.martinez 180 dic 31 07:24 explicacion.md,drwxr-xr-x 2 ronie.martinez ronie.martinez  56 dic 31 08:31 inputs,ronie.martinez@compute-0-0:~/DM/GROMACS/test$ cat explicacion.md,Este será un test realizado utilizando como control positivo binimetinib, con código de sesión 75093661 y código SMILES: CN1C=NC2=C1C=C(C(=C2F)NC3=C(C=C(C=C3)Br)F)C(=O)NOCCO.,ronie.martinez@compute-0-0:~/DM/GROMACS/test$ ls -la inputs/,total 488,drwxr-xr-x 2 ronie.martinez ronie.martinez     56 dic 31 08:31 .,drwxr-xr-x 3 ronie.martinez ronie.martinez     54 dic 31 08:15 ..,-rw-r--r-- 1 ronie.martinez ronie.martinez   4310 dic 31 07:43 ligand.pdb,-rw-r--r-- 1 ronie.martinez ronie.martinez 488554 dic 31 07:34 receptor.pdb,ronie.martinez@compute-0-0:~/DM/GROMACS/test$," y luego ten enc uent estos parametros "Dinámica molecular para un complejo proteína-ligando totalmente solvatado con un tamaño aproximado de 350,000 átomos, en condiciones fisiológicas (310 K y 1 bar), usando un paso de integración de 2 femtosegundos y aplicando control de temperatura y presión durante toda la producción. El objetivo es alcanzar una escala de tiempo de 1 microsegundo, lo que implica 500 millones de pasos de integración, almacenando periódicamente trayectorias y estados para un análisis estructural y energético detallado, con el fin de estudiar la estabilidad conformacional del complejo y la afinidad del ligando a lo largo del tiempo." en si tambien receurda que quiero basarme en esta simulacion:2.1.3 Molecular dynamics simulations.The initial structures of the ligand–receptor complexes for the MD simulation were obtained by molecular docking experiment. The ligands were parametrized in the AMBER 20 Antechamber module70 using the GAFF force field.71 For the kinase, AMBER ff19SB72 force field was employed. The PDB2PQR web-server73 was accessed to determine the protonation state of the side chains of the residues at physiological pH. The ligand–receptor complex was impregnated with pre-equilibrated TIP3P water molecules in a truncated octahedral periodic box. The minimum distance between the edges of the water box and the nearest atom of the complex was set at 12 Å. The system was neutralized by six Cl− anions, followed by the addition of Na+ and Cl− ions according to the recommendations of Machado and Pantano74 to achieve a neutral environment with a salt concentration of 0.15 M.,The minimization–heating–equilibration–production protocol was used. First, the system was subjected to geometry optimization with periodic boundary conditions in all directions. In 10[thin space (1/6-em)]000 optimization cycles (4000 steepest descent + 6000 conjugate gradient), both the protein and the ligand were constrained with harmonic potential k = 10.0 mol−1 Å−2. The system was heated stepwise from 0 K to 310 K in 500 ps without any constraints. After an equilibration period of 500 ps, a productive unconstrained molecular dynamics (MD) simulation of 1 μs duration was started, except for the drug dabrafenib, for which the total simulation time was 300 ns. A time step of 2 fs at constant pressure (1 atm) and temperature (310 K) was used. A langevine thermostat with a collision frequency of 1 ps−1 was used for temperature control. In all simulation protocols, hydrogen atoms were constrained by the SHAKE algorithm.75 The cut-off distance for non-bonded interaction was 11 Å, while for long-range electrostatic interactions, the particle mesh Ewald method76 was used. Periodic boundary conditions were employed in all directions. MD simulations were performed using the molecular dynamics package Amber.77 MD simulations were performed on the Isabella cluster of the University Computing Center, University of Zagreb, Croatia.,2.1.3.1 Binding free energy calculation.The free energies of binding (ΔGbind) between a series of potential drugs (ligands) and the hBRk were calculated according to well-established molecular mechanics/generalized Born surface area (MM/GBSA) protocol.78 The formula,within single-trajectory approach is implemented in the MMPSBA.py script of the AmberTools package. The sum of the bond, angle and dihedral energy (ΔEinternal), electrostatic (ΔEelectrostatic) and van der Waals (ΔEvdW) energies is ΔEMM, the change in MM energy contribution in the gas phase. ΔGsol is a change in solvation free energy, with a polar component (ΔGGB, electrostatic solvation energy) and a non-polar, non-electrostatic solvation contribution (ΔGSA). TΔS is the conformational entropy at binding. The production phase trajectory was divided into 20 segments of 50 ns length. Except for the drug dabrafenib (DAB), where the 300 ns trajectory was divided into 6 segments of 50 ns length. From each segment, 100 geometries were sampled at regular time steps and ΔGbind was calculated. The final ΔGbind was expressed as the mean ± standard deviation for all 20 segments. The calculated MM/GBSA free energies of binding were further decomposed into the specific contributions for each residue. In this way, the contributions of each amino acid side chain to ΔGbind were determined and the nature of the energy change in terms of interaction and solvation energies, or entropic contributions were identified.79 Since we are comparing the energies of the same receptor and of ligands with approximately the same size and number of rotatable bonds, entropic contributions (–TΔS) were neglected.,2.1.3.2 Cluster analysis.The structures of each complex were clustered based on the RMSD of the Cα atoms of each residue using the k-means algorithm. The maximum number of iterations was set to 1000, with randomized initial set of points and sieving set was equal to 10. The frames closest to the centroids of the clusters were identified and considered as representative structures of the conformations. The CPPTRAJ module80 was used to perform the cluster analysis.3.3.1 Molecular dynamics simulation study of compound T 160.In the first 900 ns of the simulation, the RMSD fluctuated around a mean value of 2.44 Å. Then an increase in RMSD was observed, and the mean value for the last 100 ns was 3.05 Å. These changes prompted us to perform a cluster analysis to identify relevant changes in the complex structure. We chose to use the k-means algorithm, where k ranges from 2 to 5. The results were analyzed using the Davies–Bouldin index (DBI), the pseudo-F statistic (pSF), and the ratio between the sum of squares of the regression and the sum of squares of error (SSR/SST) (Table S3†). The analysis confirmed the presence of three relevant conformations with selected parameters collected in Table S4.† For each conformation, a representative structure was identified as a frame closest to the cluster centroid. The distribution of the conformations of the T160:BRAF kinase (T160:hBRk) complex along the trajectory and their RMSD relative to the initial frame are shown in Fig. 6. Along with the increase in the mean RMSD values after the 900 ns mark, a slight decrease in the radius of gyration (RoG) from 19.24 Å to 19.15 Å was observed.In all three conformations, the secondary structure was generally well preserved. However, there were some minor changes in the secondary structure that could affect the geometry of the catalytic pocket Fig. 7. The first region includes residues that are in direct contact with the ligand – Arg462, Ile463, Gly464, and Ser465, while these residues do not have a defined secondary structure in the dominant conformation A, they form a β-sheet in conformations B and C. The second region with considerable changes is the region between Ile592 and Phe 635, which is almost completely without defined secondary structure. In the crystal structure (Fig. S5†)83 and in conformation A, only Pro622 to Ile625 from this region form an H4 α-helix. In conformation B, the H4 helix is absent but two new helices are formed (Glu611-Leu613 and Ile617-Trp619). The H4 helix and the Ile617–Trp619 helix are present in the C conformation.,The flexibility of the residues of the hBRk protein in the complex was examined by calculating root mean squared fluctuations (RMSF) for each residue (Fig. 8). High RMSF values indicated greater fluctuations and flexibility. The averaged RMSF value was 1.55 Å. Almost all of the first 50 residues were having higher than average RMSF values, indicating high flexibility of the N-terminus of the kinase. The highest RMSF value was having Met627, a residue whose side chain pointed toward the solvent, far from the binding site of the ligand. The previously mentioned unstructured region was having two fragments with RMSF values above 3 Å. These two fragments, Ser607–Ser614 and Ile625–Asn631, were not in direct contact with the ligand. It is important to note that the catalytically relevant residues Asp594, Phe595 and Gly596 have RMSF values below 1 Å.,Hydrogen bond analysis revealed that on average 2.11 hydrogen bonds were established between T160 and the kinase. This is on average only 0.02 less than in the complex with T183 and the hBRk. The predominant hydrogen bond formed in almost 75% of the simulation time was between the hydroxyl group of Thr529 (hydrogen acceptor) and the N4 atom of T160 with an average bond length of 1.89 ± 0.08 Å. Changes in the 3D structure of the protein were reflected in the hydrogen bonding patterns in different conformations. For example, in A, T160 formed four hydrogen bonds with three residues (two with Ser465 and one with Thr529 and Asp594). In addition, 15 residues had at least one heavy atom in a 4.0 Å zone away from the non-hydrogen atoms of T160. This illustrated the large number of stabilizing van der Waals interactions. In conformation B, only two hydrogen bonds were formed (with Lys483 and Thr529). Finally, conformation C exhibited three hydrogen bonds. The detailed interactions are shown in Fig. 9.Although the MD simulation for DAB was 300 ns long, some conclusions can be drawn. The flexibility of residues changes slightly when T160 is replaced in the catalytic pocket by dabrafenib, the drug approved for the treatment of melanoma with V600E or V600K mutation (Fig. S6†), with a slight increase in the flexibility of residues Ala544–Lys548 and a decrease in the flexibility of Ile625–Asn631. In addition, dabrafenib has on average one more hydrogen bond than T160, being predominantly (90% of the simulation time) bound to the Gly593 oxygen atom with an average hydrogen bond length of 1.84 Å, which is shorter than the hydrogen bond between Thr529 and T160. Numerous van der Waals interactions together with a higher number of hydrogen bonds could be responsible for the higher binding affinity to hBRk than T160.,3.3.2 Calculation of the free energy of binding.The binding affinity of the hit molecules and the reference drug dabrafenib was estimated using the MM/GBSA approach, in which the free energies of solvation were determined by solving the generalized Born equation. For the complex T160:hBRk, the free energy of binding without entropy contribution was −59.1 ± 2.6 kcal mol−1. T160 had the highest binding potential to hBRk of all the compounds studied. The MM/GBSA binding free energy decomposition was used to identify key residues with dominant contribution to protein–ligand binding. In the study of the SARS-CoV-2 virus, its main protease84 and the NS3 protease of Kyasanur forest disease virus,85 a threshold of −1.5 kcal mol−1 was set for the free energy of binding of a single residue to classify it as a residue with dominant contribution, and the same criteria was applied in the present study. Table 4 lists the residues with dominant contributions for T109, T126, T160, and T183 and dabrafenib. Fig. 10 shows the main interactions based on the decomposition of the binding free energy. In addition to the polar and uncharged Thr529 and the positively charged Lys483, four hydrophobic residues (Val471, Leu505, Leu514, Trp531, and Phe583) contributed significantly to the binding free energy of T160. This finding confirmed the results of previous analyzes on the importance of van der Waals interactions.As expected, the Gly593 residue, which forms the hydrogen bond, has the largest contribution to the binding of dabrafenib (DAB). Other stabilizing interactions are established with the electron-rich residues Phe583 and Trp531, the hydrophobic Val471, Leu514, and Ile527, and the polar Thr529 and Cys532. The free energy of binding for the DAB:hBRk complex was estimated to be −61.7 ± 1.0 kcal mol−1, slightly better than for T160:hBRk.,The results obtained by the analysis of Gaussian-based contour maps suggest that the introduction of the HBA group at the phenyl moiety should increase the activity of the pyrimidine–sulfonamide hybrid derivatives. Comparing compounds T160 and T126, which differ only in the presence of a nitro group at the para position of the phenyl moiety in T160, one can observe a drastic change in the binding free energy for these two compounds. While the binding free energy for T160 is −59.1 ± 2.6 kcal mol−1, it is only −39.7 ± 2.2 kcal mol−1 for T126 (vide infra). The MD simulations revealed that the nitro group sits in a hydrophobic pocket and is surrounded by hydrogen atoms. To gain a deeper insight into the nature of why some structural modifications predicted by Gaussian-based contour maps contribute to higher selectivity, additional MD simulations are needed, including non-substituted pyrimidine–sulfonamide scaffolds, but this is beyond the scope of the present manuscript.,3.3.3 Compound T109, T126 and T183 molecular dynamics simulations study.The same protocol used for the analysis of the T160:hBRk complex was applied to the analysis of the T109:hBRk, T126:hBRk, and T183:hBRk complexes. The stability of the complexes was confirmed by tracking RMSD and RoG along the trajectory (Fig. S7†).,All complexes remained stable but had higher averaged RMSD values compared to the T160:hBRk complex. The RoG values were very similar and range from 18.93 Å to 19.29 Å. The existence of multiple conformations was confirmed by cluster analysis. The cluster analysis data are summarized in Tables S5–S10.† Three complexes shared the presence of an initial short-lived conformation that converts to another conformation within 100 ns. The relative ratios between the populations of conformations A and B are approximately 0.6 to 0.3.,The primary geometric difference between the conformations of T109:hBRk and T126:hBRk, just as with T160:hBRk, was in unstructured fragments. However, in T183:hBRk, conformations A and B showed differences in secondary structure motifs compared to the representative structure of the initial conformation C (Fig. 11). For example, a shift of the H6 α-helix axis was observed. Slightly less pronounced was the shift of the H1 α-helix axis, but it played a larger role in the geometry of the binding site. While the C-terminus of the H1 α-helix had van der Waals contacts with the catalytic Phe595 residue, the N-terminus is tilted toward the unstructured loop region, reducing the volume of the cleft.hBRk residues had higher average RMSF values, 1.81 Å and 1.80 Å, when T183 and T126 were bound in the active pocket, respectively, compared with T109 (1.55 Å). The RMSF diagram (Fig. S8†) shows similar flexibility patterns. Again, the N-terminus and unstructured regions exhibited above average flexibility, while the conserved triad, which was important for catalysis, exhibited low flexibility. T126 had the lowest average number of hydrogen bonds (0.95) between ligand and the receptor. The hydrogen bond connecting the hydroxyl oxygen of Thr529 and the NH group of T126 was present only during 23% of the simulation time, with an average length of 1.91 Å. For comparison, the analogous bond in T160:hBRk was present for about 75% of the simulation time. In T183:hBRk, Gly593 acted as a hydrogen bond acceptor, and the 1.83 ± 0.10 Å long bond was present during 82% of the trajectory. In addition to hydrogen bonding, van der Waals interactions also contributed to free energy of binding. While the free energy of binding of T109 and T183 was almost within 1 kcal mol−1 (−55.5 ± 3.1 kcal mol−1 and −56.6 ± 3.0 kcal mol−1, respectively), the value for T126 is only 39.7 ± 2.2 kcal mol−1. T109, T160, and T183 can serve as excellent starting points for introducing chemical modifications to optimize their potential to inhibit BRAF kinase.
```



---

### 📝 Resumen del Proceso

El flujo de trabajo comienza preparando la **topología** de la proteína (AMBER99SB-ILDN) y del ligando (GAFF2, generado externamente). Se **fusionan** ambas estructuras en un complejo, se define una **caja octaédrica** para simular un entorno infinito y se llena de **agua** (TIP3P). Luego, se reemplazan moléculas de agua por **iones** (Na+ y Cl-) para neutralizar la carga y alcanzar una concentración fisiológica de 0.15 M. Una vez armado el sistema, se realiza una **Minimización de Energía** para eliminar choques estéricos. Posteriormente, se **calienta** el sistema a 310 K (NVT) y se **equilibra la presión** a 1 bar (NPT) usando la GPU para acelerar el cálculo. Finalmente, se ejecuta la **Producción** de Dinámica Molecular (la simulación real), se corrige la trayectoria visualmente y se extraen datos de **estabilidad (RMSD)**, **compacidad (RoG)** y **flexibilidad (RMSF)** para graficarlos.

---

### ⚠️ Requisito Previo (Archivos Externos)

Debes tener en tu carpeta `/workspace2`:

1. `inputs/receptor.pdb`
2. `ligand.itp` (Generado en ACPYPE Web)
3. `ligand.gro` (Generado por GROMACS abajo o ACPYPE)

---

### ETAPA 1: Preparación y Construcción del Complejo

**1. Topología del Receptor**
- **Input:** `inputs/receptor.pdb`
- **Output:** `receptor.gro`, `topol.top`

```
# Selección: 6 (AMBER99SB-ILDN)
gmx pdb2gmx -f inputs/receptor.pdb -o receptor.gro -water tip3p -ignh
```

**2. Generar .gro del Ligando (Si no lo tienes)**
- **Input:** `inputs/ligand.pdb`
- **Output:** `ligand.gro`

```
gmx editconf -f inputs/ligand.pdb -o ligand.gro
```

**3. Fusionar Proteína + Ligando y Corregir Átomos**
- **Input:** `receptor.gro`, `ligand.gro`
- **Output:** `complex.gro`    

```
# Fusionar archivos limpiando basura
head -n -1 receptor.gro > complex.gro
grep -vE "Generated|box" ligand.gro | tail -n +3 | head -n -1 >> complex.gro
tail -n 1 receptor.gro >> complex.gro

# Actualizar conteo de átomos en la línea 2
count=$(expr $(wc -l < complex.gro) - 3)
sed -i "2s/.*/$count/" complex.gro
```

**4. Incluir Ligando en Topología**
- **Input:** `topol.top`
- **Output:** `topol.top` (modificado)

```
# Agregar include y molecule
sed -i '/forcefield.itp"/a #include "ligand.itp"' topol.top
echo "ligand   1" >> topol.top
```

---

### ETAPA 2: Solvatación e Iones

**5. Definir Caja y Solvatar**
- **Input:** `complex.gro`
- **Output:** `solvated.gro`, `topol.top` (actualizado con agua)    

```
gmx editconf -f complex.gro -o newbox.gro -c -d 1.2 -bt octahedron
gmx solvate -cp newbox.gro -cs spc216.gro -o solvated.gro -p topol.top
```

**6. Generar Iones (Neutralizar + 0.15M)**
- **Input:** `solvated.gro`, `topol.top`    
- **Output:** `solv_ions.gro`

```
touch empty.mdp
# Generar TPR dummy
gmx grompp -f empty.mdp -c solvated.gro -p topol.top -o ions.tpr -maxwarn 2
# Reemplazar agua por iones (Selecciona grupo SOL automáticamente)
echo "SOL" | gmx genion -s ions.tpr -o solv_ions.gro -p topol.top -pname NA -nname CL -neutral -conc 0.15
```


> [!Comentario]
> JaJa Me habia dado un fallo






---

### ETAPA 3: Minimización de Energía (EM)

**7. Crear Script y Ejecutar**
- **Input:** `solv_ions.gro`
- **Output:** `em.gro` (Estructura relajada)    
```
# Crear em.mdp
cat << EOF > em.mdp
integrator  = steep
nsteps      = 10000
emtol       = 1000.0
emstep      = 0.01
pbc         = xyz
coulombtype = PME
rcoulomb    = 1.1
rvdw        = 1.1
constraints = none
define      = -DPOSRES
EOF

# Compilar y Correr
gmx grompp -f em.mdp -c solv_ions.gro -r solv_ions.gro -p topol.top -o em.tpr -maxwarn 1
gmx mdrun -v -deffnm em
```

---

### ETAPA 4: Calentamiento y Equilibrado (NVT / NPT)

**8. Calentamiento (NVT) - 0 a 310 K**
- **Input:** `em.gro`
- **Output:** `nvt.gro`, `nvt.cpt`    

```
# Crear nvt.mdp
cat << EOF > nvt.mdp
title = NVT Heating
define = 
integrator = sd
dt = 0.002
nsteps = 250000
pbc = xyz
annealing = single
annealing_npoints = 2
annealing_time = 0 500
annealing_temp = 0 310
tc-grps = System
tau_t = 1.0
ref_t = 310
pcoupl = no
constraints = h-bonds
cutoff-scheme = Verlet
coulombtype = PME
rcoulomb = 1.1
rvdw = 1.1
gen_vel = yes
gen_temp = 0
gen_seed = -1
EOF

# Compilar y Correr en GPU
gmx grompp -f nvt.mdp -c em.gro -r em.gro -p topol.top -o nvt.tpr -maxwarn 1
gmx mdrun -v -deffnm nvt -nb gpu
```

**9. Equilibrado Presión (NPT) - 1 Bar**

- **Input:** `nvt.gro`, `nvt.cpt`
- **Output:** `npt.gro`, `npt.cpt`

Bash

```
# Crear npt.mdp
cat << EOF > npt.mdp
title = NPT Equilibration
define = 
integrator = sd
dt = 0.002
nsteps = 250000
pbc = xyz
tc-grps = System
tau_t = 1.0
ref_t = 310
pcoupl = C-rescale
pcoupltype = isotropic
tau_p = 5.0
ref_p = 1.0
compressibility = 4.5e-5
constraints = h-bonds
continuation = yes
gen_vel = no
coulombtype = PME
rcoulomb = 1.1
rvdw = 1.1
EOF

# Compilar y Correr en GPU
gmx grompp -f npt.mdp -c nvt.gro -r nvt.gro -t nvt.cpt -p topol.top -o npt.tpr -maxwarn 1
gmx mdrun -v -deffnm npt -nb gpu
```

---

### ETAPA 5: Producción (MD)

**10. Dinámica Molecular Final**

- **Input:** `npt.gro`, `npt.cpt`
- **Output:** `md_0_1.xtc` (Trayectoria), `md_0_1.gro` (Final)

Bash

```
# Crear md.mdp
cat << EOF > md.mdp
title = Production MD
integrator = sd
dt = 0.002
nsteps = 500000000 
nstxout = 0
nstvout = 0
nstfout = 0
nstlog = 50000
nstenergy = 50000
nstxout-compressed = 50000
tc-grps = System
tau_t = 1.0
ref_t = 310
pcoupl = Parrinello-Rahman
pcoupltype = isotropic
tau_p = 5.0
ref_p = 1.0
compressibility = 4.5e-5
constraints = h-bonds
continuation = yes
gen_vel = no
coulombtype = PME
rcoulomb = 1.1
rvdw = 1.1
EOF

# Compilar y Correr en GPU
gmx grompp -f md.mdp -c npt.gro -t npt.cpt -p topol.top -o md_0_1.tpr -maxwarn 1
gmx mdrun -v -deffnm md_0_1 -nb gpu
```

---

### ETAPA 6: Corrección y Análisis

**11. Centrar Trayectoria**

Bash

```
# Select: 1 (Protein) -> 0 (System)
echo "1 0" | gmx trjconv -s md_0_1.tpr -f md_0_1.xtc -o md_clean.xtc -pbc mol -center
```

**12. Generar Datos Numéricos (.xvg)**

Bash

```
# RMSD (C-alpha)
echo "3 3" | gmx rms -s md_0_1.tpr -f md_clean.xtc -o rmsd_calpha.xvg -tu ns

# Radio de Giro (Protein)
echo "1" | gmx gyrate -s md_0_1.tpr -f md_clean.xtc -o gyrate.xvg

# RMSF por residuo (C-alpha)
echo "3" | gmx rmsf -s md_0_1.tpr -f md_clean.xtc -o rmsf_residue.xvg -res
```

---

### ETAPA 7: Generación de Gráficas (Python)

13. Script de Visualización

Copia y pega este bloque para generar las imágenes .png automáticamente.

Bash

```
cat << 'EOF' > plot_graphs.py
import matplotlib.pyplot as plt
import os

def read_xvg(filename):
    x, y = [], []
    if not os.path.exists(filename): return x, y
    with open(filename, 'r') as f:
        for line in f:
            if not line.startswith(('@', '#')):
                parts = line.split()
                if len(parts) >= 2:
                    x.append(float(parts[0]))
                    y.append(float(parts[1]))
    return x, y

# RMSD
t, rmsd = read_xvg('rmsd_calpha.xvg')
if t:
    plt.figure()
    plt.plot(t, rmsd, 'b-', label='C-alpha RMSD')
    plt.xlabel('Tiempo (ns)'); plt.ylabel('RMSD (nm)')
    plt.title('Estabilidad (RMSD)'); plt.grid(True, alpha=0.3)
    plt.savefig('GRAFICA_RMSD.png', dpi=300)

# RoG
t, rog = read_xvg('gyrate.xvg')
if t:
    plt.figure()
    plt.plot(t, rog, 'g-', label='Radio de Giro')
    plt.xlabel('Tiempo (ps)'); plt.ylabel('Rg (nm)')
    plt.title('Compacidad (RoG)'); plt.grid(True, alpha=0.3)
    plt.savefig('GRAFICA_RoG.png', dpi=300)

# RMSF
res, flu = read_xvg('rmsf_residue.xvg')
if res:
    plt.figure()
    plt.plot(res, flu, 'r-', label='RMSF')
    plt.xlabel('Residuo'); plt.ylabel('Fluctuación (nm)')
    plt.title('Flexibilidad (RMSF)'); plt.grid(True, alpha=0.3)
    plt.savefig('GRAFICA_RMSF.png', dpi=300)
EOF

python3 plot_graphs.py
```




