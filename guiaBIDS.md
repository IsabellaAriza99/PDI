# Guía de uso DCM2BIDS

## 1. Instalar dcm2niix
   a. `py -m pip install dcm2niix`

## 2. Instalar dcm2bids
   a. `Py -m pip install dcm2bids`

## 3. Añadir la ruta donde se almacenan los ejecutables a el path – la añadimos a las variables de entorno.
   a. `C:\Users\DRA01\AppData\Local\Programs\Python\Python311\Scripts`
      En esta ruta se guardan los ejecutables de dcm2bids y dcm2niix.

## 4. Creamos una carpeta para almacenar la información y crear la estructura bids
   a. `mkdir dcm2bids-tutorial`
   b. `cd dcm2bids-tutorial`

## 5. Creamos la estructura BIDS
   a. `dcm2bids_scaffold –help` vemos información de ayuda
   b. `dcm2bids_scaffold -o bids_project` – crea una carpeta llamada bids_project con subcarpeta y archivos con estructura bids
   c. `cd bids_project` – Nos movemos a la carpeta del proyecto

## 6. Descargamos los datos
   Opción larga
   a. [Descarga el archivo ZIP](https://github.com/neurolabusc/dcm_qa_nih/archive/refs/heads/master.zip)
   b. `tar -xzf dcm_qa_nih-master.zip -C sourcedata/` - extraemos los datos a la carpeta sourcedata
   c. `ren sourcedata\dcm_qa_nih_master dcm_qa_nih` - cambio el nombre de la carpeta
   Opción corta
   a. `git clone https://github.com/neurolabusc/dcm_qa_nih/ sourcedata/dcm_qa_nih` - Clono el repositorio con los datos

   Nota: En este tutorial, hay dos directorios con datos, uno con datos provenientes de un escáner Siemens (20180918Si) y otro con datos provenientes de GE (20180918GE). El tutorial utilizará los datos adquiridos en ambos escáneres y en el escáner Siemens ubicados en sourcedata/dcm_qa_nih/In/y pretenderá ser un solo participante.

## 7. Construcción de archivo de configuración
   Usamos el comando dcm2bids_helper que nos va a ayudar a convertir los datos a nifti y guardarlos en una nueva carpeta de archivos temporales para inspeccionar la información que contienen y nutrir el archivo de configuración.
   a. `dcm2bids_helper -d sourcedata/dcm_qa_nih/In/`
   
   Luego miramos las diferencias entre dos archivos, nos paramos en la ruta donde están los archivos.
   
   Nos vamos a enfocar solo en estas 3 adquisiciones:
   1. 004_In_DCM2NIIX_regression_test_20180918114023
   2. 003_In_EPI
