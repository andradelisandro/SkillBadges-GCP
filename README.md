## Todo los pasos realizados en cada unos de Lab

### Configuracion Inicial para todo los projecto o lab 

 - **Lista Proyecto**
 ```
    $ gcloud config list project
 ```
 - **Establecer una region y zona para el projecto**
  ```
   $ gcloud config set compute/zone asia-east1-b
   $ gcloud config set compute/region asia-east1-b
 ```
 - **Ver Region y Zona Predeterminada de un Projecto**
 ```
    $ gcloud compute project-info describe --project project-id
 ```
 - **Listar Regiones disponibles**
  ```
    $ gcloud compute regions list
 ```
 - **Mostrar Informacion sobre la region**
  ```
    $ gcloud compute regions describe [REGION]
 ```
  - **Iniciar una configuracion de Cloud Shell o SDK**
 ```
    $ gcloud init
 ```
 - **Ver configuracion activa de la SDK o cloud shell**
 ```
   $ gcloud config list
 ```