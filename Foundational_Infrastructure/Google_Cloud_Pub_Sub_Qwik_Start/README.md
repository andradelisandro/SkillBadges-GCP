## Cloud Shell

export  PROJECT_ID=qwiklabs-gcp-04-f87328ad8c54

--------------------------

##### Configuracion de proyecto en la CLI 
```
$ gcloud config set project $PROJECT_ID
```
--------------------------

####  Crear uno o varios Topics 
```sh
$ gcloud pubsub topics create myTopic
$ gcloud pubsub topics create Test1
$ gcloud pubsub topics create Test2

```
--------------------------

#### Listar los Topics

```sh
$ gcloud pubsub topics list
```
--------------------------
### Eliminar varios Topics
```
$ gcloud pubsub topics delete Test1 &&  gcloud pubsub topics delete Test2
```
--------------------------
### Crear una o varias Subscriptions  
```
$ gcloud  pubsub subscriptions create --topic myTopic mySubscription
$ gcloud  pubsub subscriptions create --topic myTopic Test1
$ gcloud  pubsub subscriptions create --topic myTopic Test2
```
--------------------------
### Listar las Subscriptions
```
$ gcloud pubsub topics list-subscriptions myTopic
```
--------------------------
### Eliminar varias Subscription
```
$ gcloud pubsub subscriptions delete Test1 && gcloud pubsub subscriptions delete Test2
```
--------------------------
### Publicar uno o varios mensaje a un Topic
```
$ gcloud pubsub topics publish myTopic --message "Hello"

$ gcloud pubsub topics publish myTopic --message "Publisher's name is Lisandro Andrade"

$ gcloud pubsub topics publish myTopic --message "Publisher likes to eat Pasticho"

$ gcloud pubsub topics publish myTopic --message "Publisher thinks Pub/Sub is awesome"

```
--------------------------
### Extraer el ultimo mensaje de un Topics
```
$ gcloud pubsub subscriptions pull mySubscription --auto-ack
```
--------------------------

### Publicar uno o varios mensaje a un Topic
```
$ gcloud pubsub topics publish myTopic --message "Publisher is starting to get the hang of Pub/Sub"

$ gcloud pubsub topics publish myTopic --message "Publisher wonders if all messages will be pulled"

$ gcloud pubsub topics publish myTopic --message "Publisher will have to test to find out"
```
--------------------------
### Extraer todos los mensaje de un Topics
```
$ gcloud pubsub subscriptions pull mySubscription --auto-ack --limit=3
```
##### Nota: `Limit` representa el numero de mensaje que se quiere mostrar del topics