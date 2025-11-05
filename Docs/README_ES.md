<div align="center"><b><a href="//README.md">English</a> | <a href="./README_ES.md">Español</a> </b></div>

![Bandera DM](/Docs/Img/BanderaDM.png)
--
<h2 align="center" style="border-bottom: none">OpenOpenMM and Amber's force field</h2>

---


# Dinámica Molecular complejo ligando-receptor amber

Este repositorio ofrece un flujo de trabajo completo y automatizado para realizar simulaciones de dinámica molecular de un complejo ligando-receptor utilizando el conjunto de herramientas Amber24. El proyecto está diseñado para ser reproducible en diversas plataformas, proporcionando guías de instalación detalladas para Ubuntu, así como para entornos de computación de alto rendimiento (HPC) que utilizan contenedores Docker y EnRoot.



##  Arbol de Archivos

-   [HpCedia](../Installation/HpCedia)

El objetivo es documentar el proceso de instalación, ejecución y simulación del conjunto de herramientas AmberTools24 dentro de la hipercomputadora de HpCedia *Sin GPU*. 
    -   Instalacion 
    -   Comprobación
    -   Simulación

-   [Ubuntu](../Installation/Ubuntu)

El objetivo es documentar el proceso de instalación, ejecución y simulación del conjunto de herramientas AmberTools24 dentro de una máquina virtual con Ubuntu *Sin GPU*.

-   [Cluster de docker](../GoogleColab/Cluster)

El repositorio [Making-it-rain](https://github.com/pablo-arantes/Making-it-rain?tab=readme-ov-file) es una excelente opción para ejecutarlo en Google Colab. Sin embargo, los tiempos de vida de las sesiones pueden ser un problema, ya que suelen durar un máximo de dos días, e incluso menos si usas GPU, que solo está disponible por unas horas.

Lo que yo hice fue utilizar la opción de [**“Conectar a un entorno de ejecución local”**](https://research.google.com/colaboratory/local-runtimes.html), que permite enlazar Google Colab con los recursos de tu propio equipo. De esta forma, no tienes límite de tiempo y puedes aprovechar la GPU de tu sistema. Eso sí, necesitas tener **Docker** instalado en tu computadora para poder configurarlo correctamente.



## 📝 Referencias

- [Amber Installation Guide](https://ambermd.org/Installation.php)
- [Amber 24 Documentation](https://ambermd.org/doc12/Amber24.pdf)
- [Installing Amber on Ubuntu](https://ambermd.org/InstUbuntu.php)
- [Installing dependencies Amber on Ubuntu](https://ambermd.org/InstUbuntu.php)

- [Making it rain](https://github.com/pablo-arantes/Making-it-rain?tab=readme-ov-file)
