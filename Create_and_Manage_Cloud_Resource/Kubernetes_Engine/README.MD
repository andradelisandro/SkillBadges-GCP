##### export PROJECT_ID=qwiklabs-gcp-00-f5e667dfc919
##### export REGION=us-central1 
##### export ZONE=us-central1-a 

--------------------------
### Configurar una zona por default
```
$ gcloud config set compute/zone us-central1-a
```
--------------------------
### Creacion de un clusters
```
$ gcloud container clusters create my-cluster
```
--------------------------
### Autenticar contra el clusters
```
$ gcloud container clusters get-credentials my-cluster
$ gcloud container clusters get-credentials my-cluster --zone $ZONE --project $PROJECT_ID
```
--------------------------
### Deploy de un aplicacion en kubernetes por medio de una imagen
```
$ kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
```
--------------------------
### Exponer el pod para recibir peticiones publicamente en el puerto 8080
```
$ kubectl expose deployment hello-server --type=LoadBalancer --port 8080
```
--------------------------
### Verificar los servicios creados en kubernestes
```
$ kubectl get service
```
--------------------------
### Borrar un cluster de kubernetes 
```
$ gcloud container clusters delete my-cluster
```