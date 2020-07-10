## Cloud Shell

export  PROJECT_ID=qwiklabs-gcp-04-f87328ad8c54
export  BUCKET_ID=$PROJECT_ID-storage

--------------------------

##### Configuracion de proyecto en la CLI 
```
$ gcloud config set project $PROJECT_ID
```
--------------------------

####  Creacion de carpeta y archivo 
```sh
$ mkdir gcf_hello_world

$ cd gcf_hello_world 

$ nano index.js
```
#### Lo que contiene el archivo index.js
```
/**
* Background Cloud Function to be triggered by Pub/Sub.
* This function is exported by index.js, and executed when
* the trigger topic receives a message.
*
* @param {object} data The event payload.
* @param {object} context The event metadata.
*/
exports.helloWorld = (data, context) => {
const pubSubMessage = data;
const name = pubSubMessage.data
    ? Buffer.from(pubSubMessage.data, 'base64').toString() : "Hello World";

console.log(`My Cloud Function: ${name}`);
};
```
#### Salga de nano (Ctrl+X) y guarde (Y) el archivo.
--------------------------

#### Creacion de bucket con la region por defualt

```
$ gsutil mb -p $PROJECT_ID gs://$BUCKET_ID
```
--------------------------
### Creacion de Funcion
```
$ gcloud functions deploy helloWorld \
  --stage-bucket $BUCKET_ID \
  --trigger-topic hello_world \
  --runtime nodejs8
```
--------------------------
### Descripcion de la Funcion
```
$ gcloud functions describe helloWorld
```
--------------------------
### Test de la Funcion
```
$ DATA=$(printf 'Hello World!'|base64) && gcloud functions call helloWorld --data '{"data":"Hello World!"}'
```
--------------------------
### Ver los logs de la Funcion
```
$ gcloud functions logs read helloWorld
```