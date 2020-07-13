##### export PROJECT_ID=qwiklabs-gcp-04-7969ae3e1631
##### export REGION=us-central1
##### export ZONE=us-central1-a 

--------------------------
### Configurar una zona por default
```
$ gcloud config set compute/region $REGION
$ gcloud config set compute/zone $ZONE
```
--------------------------
### Script en Bash para iniciar el web server nginx cuando se inicie una GCE
```
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
```
--------------------------
### Crear un template en el cual estan la meta data que contiene el grupo de GCE
```
$ gcloud compute instance-templates create nginx-template --metadata-from-file startup-script=startup.sh 
```
--------------------------
### Crear un grupo de destino o targe pool para enviar todo el trafico del balanceador al el grupo de GCE
```
$ gcloud compute target-pools create nginx-pool --region $REGION
```
--------------------------
### Crear un grupo de GCE y asociarla a un targe poll y a un template con la metadata de la VMs
```
$ gcloud compute instance-groups managed create nginx-group \
         --base-instance-name nginx \
         --size 2 \
         --template nginx-template \
         --target-pool nginx-pool \
         --zone $ZONE
```
##### Nota: El `size` represente el numero de instancia que se crearan para este ejemplo son 2.
--------------------------
### Listar todo las GCE creadas
```
$ gcloud compute instances list
```
--------------------------
### Creadon una regla de firewall para permitir todo el trafico al protocolo `tcp` en el puerto 80
```
$ gcloud compute firewall-rules create www-firewall --allow tcp:80
```
--------------------------
### Crear un balanceador de carga L3 el cual redirecciona todo el trafico que llege al puerto `80`. Lo redirecciona al grupo de reenvio o targe pool, el cual asociado al grupo de instancias `nginx-group`.
```
$ gcloud compute forwarding-rules create nginx-lb \
         --region $REGION \
         --ports=80 \
         --target-pool nginx-pool
```
--------------------------
### Listar todo los targe pool o regla de reenvio creadas
```
$ gcloud compute forwarding-rules list
```
--------------------------
### Crear un health checks para conocer si las instancia responde al trafico http o https
```
$ gcloud compute http-health-checks create http-basic-check
```
--------------------------
### Crear un servico HTTP y asociarlo al grupo de instancia
```
$ gcloud compute instance-groups managed \
       set-named-ports nginx-group \
       --named-ports http:80 \
       --zone $ZONE
```
--------------------------
### Crear los backend  y asociarle el helt checks
```
$ gcloud compute backend-services create nginx-backend \
      --protocol HTTP --http-health-checks http-basic-check --global
```
--------------------------
### Agregar a los backend el grupo de instancia
```
$ gcloud compute backend-services add-backend nginx-backend \
    --instance-group nginx-group \
    --instance-group-zone $ZONE \
    --global
```
--------------------------
### Crear un URL map por default para redirigir todo la solicitudes a las instancias 
```
$ gcloud compute url-maps create web-map \
    --default-service nginx-backend
```
--------------------------
### Crear un proxy reverso para enrutar las solicitud al URL map creado
```
$ gcloud compute target-http-proxies create http-lb-proxy \
    --url-map web-map
```
--------------------------
### Crear una regla de reenvio global para administrar y enrutar las solicitudes entrantes.
```
$ gcloud compute forwarding-rules create http-content-rule \
        --global \
        --target-http-proxy http-lb-proxy \
        --ports 80
```
--------------------------
### Lista las regla de renvio 
```
$ gcloud compute forwarding-rules list
```





