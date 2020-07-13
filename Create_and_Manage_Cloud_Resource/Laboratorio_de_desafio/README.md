##### export PROJECT_ID=qwiklabs-gcp-04-07405616b3a7
##### export REGION=us-east1
##### export ZONE=us-east1-b

--------------------------
### Configurar una zona por default
```
$ gcloud config set compute/region $REGION
$ gcloud config set compute/zone $ZONE
```
--------------------------
### Cree una instancia jumphost para el proyecto
```
$ gcloud compute instances create nucleus-jumphost --machine-type f1-micro \
--boot-disk-type pd-standard \
--image debian-9-stretch-v20200618 \
--image-project debian-cloud \
--boot-disk-device-name nucleus-jumphost \
--boot-disk-size 10GB \
--network nucleus-vpc \
--zone $ZONE
```
--------------------------
### Cree un Clúster (en la Region us-east1) para alojar el servicio
```
$  gcloud container clusters create nucleus-cluster --region us-east1  --network nucleus-vpc --num-nodes 1
```
--------------------------
### Autenticar contra el clusters
```
$ gcloud container clusters get-credentials nucleus-cluster --region $REGION --project $PROJECT_ID
```
--------------------------
### Utilice el contenedor hello-app de Docker ('gcr.io/google-samples/hello-app:2.0').
```
$ kubectl create deployment hello-app --image=gcr.io/google-samples/hello-app:2.0
```
--------------------------
### Exponga la aplicación en el puerto 8080.
```
$ kubectl expose deployment hello-app --type=LoadBalancer --port 8080
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
### Cree una plantilla de instancias
```
$ gcloud compute instance-templates create nucleus-nginx-template --metadata-from-file startup-script=startup.sh \
--tags default-allow-http \
--machine-type f1-micro  \
--network nucleus-vpc
```
### Cree un grupo de destino o targe pool
```
$ gcloud compute target-pools create nucleus-nginx-pool --region $REGION
```
--------------------------
### Cree un grupo de instancias administradas.
```
$ gcloud compute instance-groups managed create nucleus-nginx-group \
         --base-instance-name nginx \
         --size 2 \
         --template nucleus-nginx-template \
         --target-pool nucleus-nginx-pool \
         --region $REGION
```
--------------------------
### Cree una regla de firewall para permitir el tráfico (80/TCP).
```
gcloud compute firewall-rules create web-server-firewall \
 --allow tcp:80  \
 --network nucleus-vpc
```
--------------------------
### Cree una verificación de estado.
```
$ gcloud compute http-health-checks create http-basic-check
```
--------------------------
### Cree un servicio de backend y conecte el grupo de instancias administradas.
```
# Crear un servico HTTP y asociarlo al grupo de instancia

$ gcloud compute instance-groups managed \
       set-named-ports nucleus-nginx-group \
       --named-ports http:80 \
       --region $REGION

# Crear los backend  y asociarle el helt checks

$ gcloud compute backend-services create nginx-backend \
      --protocol HTTP  \
      --http-health-checks http-basic-check --global

# Agregar a los backend el grupo de instancia

$ gcloud compute backend-services add-backend nginx-backend \
    --instance-group nucleus-nginx-group \
    --instance-group-region $REGION \
    --global
```
--------------------------
### Cree un mapa de URL y un Proxy HTTP de destino para enrutar las solicitudes a su mapa de URL.
```
# Crear un URL map por default para redirigir todo la solicitudes a las instancias 

$ gcloud compute url-maps create web-map --default-service nginx-backend

# Crear un proxy reverso para enrutar las solicitud al URL map creado

$ gcloud compute target-http-proxies create http-lb-proxy --url-map web-map
```
### Cree una regla de reenvío.
```
$ gcloud compute forwarding-rules create http-content-rule \
        --global \
        --target-http-proxy http-lb-proxy \
        --ports 80
```
