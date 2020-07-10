## Cloud Shell

##### export  PROJECT_ID=qwiklabs-gcp-04-8351fba8efb7
##### export  BUCKET_ID=$PROJECT_ID-storage-photo
--------------------------

#### Set default region y zona para todo el proyecto por default
```
$ gcloud compute project-info add-metadata \
    --metadata google-compute-default-region=us-east1,google-compute-default-zone=us-east1-b

$ gcloud init
```
--------------------------
#### Ver que region y zona estar por default en el proyecto  todo el proyecto por default
```
$ gcloud compute project-info describe --project $PROJECT_ID | grep google-compute-default
```
--------------------------
#### Configurar una zona y una region para el actual projecto
```
$ gcloud config set compute/zone asia-east1-b
$ gcloud config set compute/region asia-east1-b
```
--------------------------
#### Configuracion de proyecto en la CLI 
```
$ gcloud config set project $PROJECT_ID
```
--------------------------
####  Crear un Cloud Storage
```
$ gsutil mb -p $PROJECT_ID gs://$BUCKET_ID
```
--------------------------
####  Crear un Topics 
```sh
$ gcloud pubsub topics create recuerdos-photos-topics
```
####  Creacion de carpeta y archivo 
```sh
$ mkdir thumbnail_fucntion

$ cd thumbnail_fucntion 

$ nano index.js
```
#### Lo que contiene el archivo index.js
```
/* globals exports, require */
//jshint strict: false
//jshint esversion: 6
"use strict";
const crc32 = require("fast-crc32c");
const gcs = require("@google-cloud/storage")();
const PubSub = require("@google-cloud/pubsub");
const imagemagick = require("imagemagick-stream");

exports.thumbnail = (event, context) => {
  const fileName = event.name;
  const bucketName = event.bucket;
  const size = "64x64"
  const bucket = gcs.bucket(bucketName);
  const topicName = "recuerdos-photos-topics";
  const pubsub = new PubSub();
  if ( fileName.search("64x64_thumbnail") == -1 ){
    // doesn't have a thumbnail, get the filename extension
    var filename_split = fileName.split('.');
    var filename_ext = filename_split[filename_split.length - 1];
    var filename_without_ext = fileName.substring(0, fileName.length - filename_ext.length );
    if (filename_ext.toLowerCase() == 'png' || filename_ext.toLowerCase() == 'jpg'){
      // only support png and jpg at this point
      console.log(`Processing Original: gs://${bucketName}/${fileName}`);
      const gcsObject = bucket.file(fileName);
      let newFilename = filename_without_ext + size + '_thumbnail.' + filename_ext;
      let gcsNewObject = bucket.file(newFilename);
      let srcStream = gcsObject.createReadStream();
      let dstStream = gcsNewObject.createWriteStream();
      let resize = imagemagick().resize(size).quality(90);
      srcStream.pipe(resize).pipe(dstStream);
      return new Promise((resolve, reject) => {
        dstStream
          .on("error", (err) => {
            console.log(`Error: ${err}`);
            reject(err);
          })
          .on("finish", () => {
            console.log(`Success: ${fileName} → ${newFilename}`);
              // set the content-type
              gcsNewObject.setMetadata(
              {
                contentType: 'image/'+ filename_ext.toLowerCase()
              }, function(err, apiResponse) {});
              pubsub
                .topic(topicName)
                .publisher()
                .publish(Buffer.from(newFilename))
                .then(messageId => {
                  console.log(`Message ${messageId} published.`);
                })
                .catch(err => {
                  console.error('ERROR:', err);
                });

          });
      });
    }
    else {
      console.log(`gs://${bucketName}/${fileName} is not an image I can handle`);
    }
  }
  else {
    console.log(`gs://${bucketName}/${fileName} already has a thumbnail`);
  }
};
```
```sh
$ nano package.json
```
#### Lo que contiene el archivo package.json
```
{
  "name": "thumbnails",
  "version": "1.0.0",
  "description": "Create Thumbnail of uploaded image",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "@google-cloud/storage": "1.5.1",
    "@google-cloud/pubsub": "^0.18.0",
    "fast-crc32c": "1.0.4",
    "imagemagick-stream": "4.1.1"
  },
  "devDependencies": {},
  "engines": {
    "node": ">=4.3.2"
  }
}
```
#### Salga de nano (Ctrl+X) y guarde (Y) el archivo.
--------------------------
### Permiso de Allusers al bucket
```
$ gsutil iam ch allUsers:objectadmin gs://$BUCKET_ID
```
--------------------------
### Creacion de Funcion
```
$ gcloud functions deploy thumbnail \
  --trigger-resource $BUCKET_ID \
  --trigger-event google.storage.object.finalize \
  --runtime nodejs8
```
--------------------------
### Eliminacion de roles a un usuario
```
$ gcloud projects remove-iam-policy-binding $PROJECT_ID --member='user:student-04-b6ec39a41703@qwiklabs.net' --role='roles/viewer'
```