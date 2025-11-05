# Instalación y Configuración de AMBER en UBUNTU

Este documento describe el proceso para configurar el entorno de AMBER en ubuntu.

## Requisitos Previos
- Conexión a internet para descargar las dependencias.
- Permisos para instalar software dentro de la maquina.

## Pasos de Instalación


### 1. Configuración Inicial

1. *Actualizar la lista de paquetes y actualizar el sistema*  
```bash
sudo apt update && apt upgrade -y || echo "Error al actualizar el sistema"
```

2. *Instalar dependencias necesarias para AMBER*  
```bash
sudo apt -y update
sudo apt -y install tcsh make gcc gfortran flex bison patch bc wget xorg-dev libz-dev libbz2-dev ||echo "Error al instalar dependencias"
```

3. *Instalar Python 3*  
```bash
sudo apt install -y python3 || echo "Error al instalar Python3"
sudo apt update && sudo apt install -y cmake g++ gcc make build-essential || echo "Instalar dependecias del instalador de c++"
```

4. *Verificación*
```bash
dpkg -l | grep -E 'tcsh|make|gcc|gfortran|flex|bison|patch|bc|wget|xorg-dev|libz-dev|libbz2-dev' && python3 --version
```


### 2. Instalación de Miniconda

1. *Descargar el instalador de Miniconda*  
```bash
wget -q https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O /tmp/Miniconda3-latest-Linux-x86_64.sh || echo "Error al descargar Miniconda"
```

2. *Instalar Miniconda de forma silenciosa*  
```bash
bash /tmp/Miniconda3-latest-Linux-x86_64.sh -b -p "$HOME/miniconda" || echo "Error al instalar Miniconda"
```

3. *Configurar el PATH para incluir Miniconda*  
```bash
export PATH="$HOME/miniconda/bin:$PATH"
```

3. *Actualizar PATH*  
```bash
source ~/.bashrc
```

4. *Verificación0*
```bash
$HOME/miniconda/bin/conda --version && echo $PATH | grep "$HOME/miniconda"
```

5. *Activar Conda*
```bash
. /home/fait-arch/miniconda/etc/profile.d/conda.sh
conda init bash
conda activate
```
6. *Crear Entorno de Conda para Amber*
```bash
conda create --name AmberTools23
conda activate AmberTools23
```

7. *Instalar dependencias en el Entorno Conda para amber23* 
```bash
conda install -c conda-forge ambertools=23
```

8. *Actualizar dependencias en el Entorno Conda para amber23*
```bash
conda update -c conda-forge ambertools 
```


### 3. Descarga e Instalación de AmberTools y Amber
1. *Cambiar al directorio temporal*  
```bash
cd /tmp || echo "Error al acceder al directorio /tmp"
```

2. *Descargar AmberTools y Amber*  
```bash
curl -X POST -d "Name=Juan Moromenacho" -d "Institution=UIDE" -o ambertools24.tar.bz2 https://ambermd.org/cgi-bin/AmberTools24-get.pl && curl -X POST -d "Name=Juan Moromenacho" -d "Institution=UIDE" -o amber24.tar.bz2 https://ambermd.org/cgi-bin/Amber24free-get.pl 
```

3. *Extraer los archivos descargados*  
```bash
mkdir ~/Amber24 
tar xvfj ambertools24.tar.bz2 -C ~/Amber24 
tar xvfj amber24.tar.bz2 -C ~/Amber24
```

4. *Verificación*
```bash
ls -l ambertools24.tar.bz2 amber24.tar.bz2
ls ~/Amber24/
```


### 4. Compilación de Amber

1. *Acceder al directorio de construcción*  
```bash
cd ~/Amber24/amber24_src/build || echo "Error al acceder al directorio de construcción"
```   

2. *Ejecutar el script de configuración*  
```bash
sudo ./run_cmake || echo "Error al ejecutar run_cmake"
```
En esta etapa hay que revisar los mensajes de error que pueden surgir devido a problemas de instalación de paquetes en el ubuntu.  
3. *Compilar e instalar*  
```bash
sudo make install || echo "Error al ejecutar make install"
```

### 5. Configuración Final

1. *Configurar el entorno de AMBER*  
```bash
source "$HOME/fait-arch/Amber24/amber24/amber.sh" || echo "Error al configurar el entorno AMBER"
```   
Cambia el usurio fait-arch por el nombre de tu usurio

## Notas

- Asegúrate de reemplazar <User> con tu nombre de usuario en el sistema.


## Referencias

- [Amber Installation Guide](https://ambermd.org/Installation.php)
- [Amber 24 Documentation](https://ambermd.org/doc12/Amber24.pdf)
