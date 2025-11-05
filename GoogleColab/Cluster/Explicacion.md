## Método 1: Conectar desde Google Colab (Recomendado)

### Paso 1: Abrir Google Colab
Ve a https://colab.research.google.com/

### Paso 2: Conectar a runtime local
1. En la esquina superior derecha, haz clic en el botón **"Conectar"** (o "Connect")
2. Del menú desplegable, selecciona **"Conectar a un entorno de ejecución local"** (Connect to local runtime)
3. Ingresa la URL: `http://127.0.0.1:9000`
4. Haz clic en **"Conectar"**

### URLs disponibles:
- Runtime 1: `http://127.0.0.1:9000`
- Runtime 2: `http://127.0.0.1:9001`
- Runtime 3: `http://127.0.0.1:9002`

## Método 2: Acceso directo desde el navegador

También puedes acceder directamente a Jupyter desde tu navegador:

### Opción A: JupyterLab
```
http://127.0.0.1:9000/lab
http://127.0.0.1:9001/lab
http://127.0.0.1:9002/lab
```

### Opción B: Jupyter Notebook clásico
```
http://127.0.0.1:9000/tree
http://127.0.0.1:9001/tree
http://127.0.0.1:9002/tree
```

## Troubleshooting si Colab no conecta:## Resumen de URLs y Accesos:

| Runtime | API | JupyterLab | Notebook Clásico | Para Colab |
|---------|-----|------------|------------------|------------|
| Runtime 1 | http://127.0.0.1:9000/api | http://127.0.0.1:9000/lab | http://127.0.0.1:9000/tree | http://127.0.0.1:9000 |
| Runtime 2 | http://127.0.0.1:9001/api | http://127.0.0.1:9001/lab | http://127.0.0.1:9001/tree | http://127.0.0.1:9001 |
| Runtime 3 | http://127.0.0.1:9002/api | http://127.0.0.1:9002/lab | http://127.0.0.1:9002/tree | http://127.0.0.1:9002 |