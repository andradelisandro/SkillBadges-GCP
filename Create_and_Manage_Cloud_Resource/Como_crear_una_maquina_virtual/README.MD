## Cómo crear una máquina virtual

##### export PROJECT_ID=qwiklabs-gcp-04-37b6cf00f613
##### export REGION=us-central1 
##### export ZONE=us-central1-c 
##### export TYPE_INSTANCES=n1-standard-2
--------------------------
### Lista de disco y imagenes
```
$ gcloud compute disk-types list|grep $ZONE
$ gcloud compute images list |grep debian
```
--------------------------
### Verificar si existe la regla para permitir http, si no exite hay que crearla
```
gcloud compute --project=$PROJECT_ID firewall-rules create default-allow-http \
--direction=INGRESS --priority=1000 --network=default --action=ALLOW \
--rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server
```
--------------------------
### Creando Instances
```
gcloud compute instances create gcelab --machine-type $TYPE_INSTANCES --zone $ZONE \
--boot-disk-type pd-standard \
--image-project debian-cloud \
--image debian-9-stretch-v20200618 \
--boot-disk-device-name gcelab \
--boot-disk-size 10GB \
--tags http-server \
--metadata startup-script='#! /bin/bash
# Installs apache and a custom homepage
sudo su -
apt-get update
apt-get install nginx -y'
```
--------------------------
### Crear otra VM
```
gcloud compute instances create gcelab2 --machine-type $TYPE_INSTANCES --zone $ZONE
```
--------------------------
### Establecer una conexion ssh con una Instances
```
gcloud compute ssh gcelab2 --zone $ZONE
```