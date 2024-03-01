# Guía de uso DCM2BIDS

## 1. Instalar dcm2niix
   `py -m pip install dcm2niix`

## 2. Instalar dcm2bids
   `py -m pip install dcm2bids`

## 3. Añadir la ruta donde se almacenan los ejecutables a el path – la añadimos a las variables de entorno.
   `C:\Users\DRA01\AppData\Local\Programs\Python\Python311\Scripts`
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
   2. 003_In_EPI_PE=AP_20180918121230
   3. 004_In_EPI_PE=PA_20180918121230
   
   La primera es una adquisición de resonancia magnética funcional en estado de reposo, mientras que la segunda y la tercera son EPI de mapa de campo.
   
   - Puedo revisar las diferencias entre los archivos 2 y 3
   
   `fc 003_In_EPI_PE=AP_20180918121230.json 004_In_EPI_PE=PA_20180918121230.json`
   
   Idealmente nos debemos fijar en un campo especifico, el comando anterior nos entrega demasiadas información, entonces en este caso nos enfocamos en un campo “seriesDescription”.
   
   4. Por ejemplo vamos a fijarnos en el campo “seriesDescription” de la imagen 004_In_DCM2NIIX_regression_test_20180918114023
   
   Para esto, revisamos la información que hay dentro del archivo de metadatos asociado a esa imagen por medio del comando:
   
   `type 004_In_DCM2NIIX_regression_test_20180918114023.json`
   
   Encontramos que en “seriesDescription” está el valor “Axial EPI-FMRI (Interleaved I to S)", entonces para filtrar este tipo de imagen en el archivo de configuración podríamos usar el valor “Axial EPI-FMRI*”
   
   b. `findstr "Axial EPI-FMRI" tmp_dcm2bids/helper/*.json`
   
   La idea es solo encontrar una coincidencia, pero este caso tenemos 4, entonces tenemos que mejorar nuestro filtro, podríamos probar con la expresión completa, es decir: “Axial EPI-FMRI (Interleaved I to S)"
   
   Si volvemos a buscar:
   
   c. `findstr /C:"Axial EPI-FMRI (Interleaved I to S)" tmp_dcm2bids/helper/*.json`
   
   Ahora que tenemos una coincidencia única sabemos como filtrar en nuestro archivo de configuración.
   
   Pasando a los mapas de campo, si inspeccionamos los archivos secundarios (los mismos que se compararon en la sección dcm2bids_helper), puede ver un patrón de "EPI PE=AP", "EPI PE=PA", "EPI PE=RL"y "EPI PE=LR en el campo "SeriesDescription"
   
   En nuestra carpeta code vamos a guardar nuestro archivo de configuración, usualmente se nombra “dcm2bids_config.json”
   
   Creamos el archivo de configuración por consola:
   
   d. `echo { "descriptions": [] } > code\dcm2bids_config.json`
   
   Ahora, modificamos el archivo .json así:

```json
{
  "descriptions": [
    {
      "id": "id_task-rest",
      "datatype": "func",
      "suffix": "bold",
      "custom_entities": "task-rest",
      "criteria": {
        "SeriesDescription": "Axial EPI-FMRI (Interleaved I to S)*"
      },
      "sidecar_changes": {
        "TaskName": "rest"
      }
    },
    {
      "datatype": "fmap",
      "suffix": "epi",
      "criteria": {
        "SeriesDescription": "EPI PE=*"
      },
      "sidecar_changes": {
        "intendedFor": ["id_task-rest"]
      }
    }
  ]
}
```

Para los mapas de campo, debemos agregar un campo "intendedFor” para mostrar que estos mapas de campo deben usarse con su adquisición de resonancia magnética funcional.

Ahora ya podemos correr el programa

a. `dcm2bids –help` nos da información de la sintaxis

b. `dcm2bids -d sourcedata/dcm_qa_nih/In/ -p ID01 -c code/dcm2bids_config.json --auto_extract_entities`
