# Installation de Hadoop avec Docker

Ce guide vous montre comment configurer un cluster Hadoop avec Docker et exécuter un job MapReduce pour compter les mots dans un fichier.

## Étapes d'installation

### 1. Télécharger l'image Docker

```bash
docker pull liliasfaxi/spark-hadoop:hv-2.7.2
```

### 2. Création des conteneurs

#### 2.1. Création du réseau bridge

```bash
docker network create --driver bridge hadoop
```

#### 2.2. Création et lancement des conteneurs

```bash
# Conteneur maître
docker run -itd --net=hadoop --name hadoop-master -p 50070:50070 -p 8088:8088 -p 16010:16010 liliasfaxi/spark-hadoop:hv-2.7.2

# Conteneurs esclaves
docker run -itd --net=hadoop --name hadoop-slave1 -p 8042:8042 liliasfaxi/spark-hadoop:hv-2.7.2
docker run -itd --net=hadoop --name hadoop-slave2 -p 8043:8042 liliasfaxi/spark-hadoop:hv-2.7.2
```

### 3. Accès au conteneur maître

```bash
docker exec -it hadoop-master bash
```

### 4. Démarrage de Hadoop et YARN

Dans le conteneur `hadoop-master` :

```bash
./start-hadoop.sh
```

### 5. Utilisation de HDFS

Créez un répertoire dans HDFS, chargez un fichier, et vérifiez son contenu.

```bash
hadoop fs -mkdir -p /input
hadoop fs -put purchases.txt /input
hadoop fs -ls /input
hadoop fs -tail /input/purchases.txt
```

## Exécution d'un Job MapReduce

### 1. Préparez le fichier JAR de votre application MapReduce et copiez-le dans le conteneur `hadoop-master`.

```bash
docker cp chemin/vers/wordcount-1.0-SNAPSHOT.jar hadoop-master:/root/
```

### 2. Exécutez le job MapReduce

```bash
hadoop jar wordcount-1.0-SNAPSHOT.jar tp1.WordCount /input/purchases.txt /output
```

### 3. Affichez les résultats

```bash
hadoop fs -ls /output
hadoop fs -cat /output/part-r-00000
```

### Pourquoi copier le fichier JAR dans Hadoop ?

- **Déploiement du code** : Permet à Hadoop de distribuer le code MapReduce sur les nœuds du cluster.
- **Exécution des jobs** : Le JAR permet d’exécuter le job MapReduce sur les données en HDFS.
- **Accès aux données HDFS** : Le JAR accède aux fichiers dans HDFS pour traitement parallèle sur le cluster.

Copier le fichier JAR dans Hadoop permet ainsi d'exécuter le job MapReduce sur le cluster pour traiter les données en parallèle.
