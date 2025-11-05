<div align="center"><b><a href="./README.md">English</a> | <a href="./Docs/README_ES.md">Español</a> </b></div>

## ![DM Flag](Docs/Img/BanderaDM.png)

<h2 align="center" style="border-bottom: none">OpenOpenMM and Amber’s Force Field</h2>

---

# Molecular Dynamics of a Ligand–Receptor Complex using Amber

This repository provides a complete and automated workflow for performing molecular dynamics simulations of a ligand–receptor complex using the Amber24 toolkit.
The project is designed to be reproducible across multiple platforms, providing detailed installation guides for Ubuntu as well as for high-performance computing (HPC) environments using Docker and EnRoot containers.

---

## File Tree

* [HpCedia](./Installation/HpCedia/installAMBER_Docker.md)

The goal is to document the installation, execution, and simulation process of the AmberTools24 toolkit on the HpCedia supercomputer *without GPU support*.
-   Installation
-   Verification
-   Simulation

* [Ubuntu](./Installation/Ubuntu/installAmberUbuntu.md)

The goal is to document the installation, execution, and simulation process of the AmberTools24 toolkit on a virtual machine running Ubuntu *without GPU support*.

* [Docker Cluster](./GoogleColab/Cluster/Explicacion.md)

The repository [Making-it-rain](https://github.com/pablo-arantes/Making-it-rain?tab=readme-ov-file) is an excellent option for running on Google Colab. However, Colab’s session lifetimes can be problematic, as they usually last a maximum of two days—or even less if using a GPU, which is only available for a few hours.

What I did instead was use the [**“Connect to a local runtime”**](https://research.google.com/colaboratory/local-runtimes.html) option, which allows you to link Google Colab to your own computer’s resources.
This way, there’s no time limit, and you can take advantage of your system’s GPU.
However, you’ll need to have **Docker** installed on your computer to configure it properly.

* [Docker Cluster](./GoogleColab/Docker/Explicacion.md)
This section explains how to set up a Docker container to run Amber simulations on a local machine or one server. [Oficial Documentation](https://research.google.com/colaboratory/local-runtimes.html)
---

## 📝 References

* [Amber Installation Guide](https://ambermd.org/Installation.php)
* [Amber 24 Documentation](https://ambermd.org/doc12/Amber24.pdf)
* [Installing Amber on Ubuntu](https://ambermd.org/InstUbuntu.php)
* [Installing Amber dependencies on Ubuntu](https://ambermd.org/InstUbuntu.php)
* [Making it rain](https://github.com/pablo-arantes/Making-it-rain?tab=readme-ov-file)