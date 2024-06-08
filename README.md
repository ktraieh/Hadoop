# Hadoop

1. Making a working directory

```
mkdir running-hadoop
cd running-hadoop
```

2. Clone the big-data-europe fork for commodity hardware

```
git clone https://github.com/wxw-matt/docker-hadoop.git
cd docker-hadoop
```

3. Test Docker 

```
docker run hello-world
```

4. Change version to use docker-compose-v2

```
mv docker-compose.yml docker-compose-v1.yml
mv docker-compose-v3.yml docker-compose.yml
```

This gave me "no such file or directory"


5. Bring the Hadoop containers up - 88.6 sec

```
docker-compose up -d
```

6. Use "docker ps" to list the nodes, this is what came up for me: 

```
docker ps

CONTAINER ID   IMAGE                                                    COMMAND                  CREATED          STATUS                    PORTS                                            NAMES
aad45d5df159   bde2020/hadoop-historyserver:2.0.0-hadoop3.2.1-java8     "/entrypoint.sh /run…"   16 minutes ago   Up 16 minutes (healthy)   8188/tcp                                         historyserver
4376c1148b61   bde2020/hadoop-nodemanager:2.0.0-hadoop3.2.1-java8       "/entrypoint.sh /run…"   16 minutes ago   Up 16 minutes (healthy)   8042/tcp                                         nodemanager
5a2dd978558e   bde2020/hadoop-resourcemanager:2.0.0-hadoop3.2.1-java8   "/entrypoint.sh /run…"   16 minutes ago   Up 16 minutes (healthy)   8088/tcp                                         resourcemanager
82cb997da75c   bde2020/hadoop-datanode:2.0.0-hadoop3.2.1-java8          "/entrypoint.sh /run…"   16 minutes ago   Up 16 minutes (healthy)   9864/tcp                                         datanode
4e108842af27   bde2020/hadoop-namenode:2.0.0-hadoop3.2.1-java8          "/entrypoint.sh /run…"   16 minutes ago   Up 16 minutes (healthy)   0.0.0.0:9000->9000/tcp, 0.0.0.0:9870->9870/tcp   namenode
b11e43865f46   docker/welcome-to-docker:latest                          "/docker-entrypoint.…"   37 minutes ago   Up 37 minutes             0.0.0.0:8088->80/tcp                             welcome-to-docker
![image](https://github.com/ktraieh/Hadoop/assets/158105841/8caf2b3b-3d4c-4e6e-9061-5d5ddc427912)

```

Used the one with "name" in it:

```
docker exec -it docker-hadoop-namenode-1 /bin/bash
```

Testing hello world:

```
echo hello world
```

7. Set up the node with directories - data, results, jars 

```
mkdir app
mkdir app/data
mkdir app/res
mkdir app/jars
```

8. Fetch data to app/data
While in root user, I then inputed:

```
cd /app/data
curl https://www.gutenberg.org/cache/epub/1342/pg1342.txt -o austen.txt

curl https://raw.githubusercontent.com/cd-public/books/main/pg84.txt -o shelley.txt

curl https://raw.githubusercontent.com/cd-public/books/main/pg768.txt -o bronte.txt
```

Checking file sizes 

```
ls -al
```

I got: 

```
total 1888
drwxr-xr-x 2 root root   4096 May 31 03:32 .
drwxr-xr-x 5 root root   4096 May 31 03:31 ..
-rw-r--r-- 1 root root 772420 May 31 03:31 austen.txt
-rw-r--r-- 1 root root 693877 May 31 03:32 bronte.txt
-rw-r--r-- 1 root root      3 May 31 03:31 hi.txt
-rw-r--r-- 1 root root 448937 May 31 03:31 shelley.txt

```

9. Fetch compute to app/jars

```
cd /app/jars
docker cp .\jobs\jars\WordCount.jar namenode:/app/jars/WordCount.jar
lstat /Users/kassandratraieh/running-hadoop/eu/docker-hadoop/.jobsjarsWordCount.jar
```
This gave me "no such file or directory" and then I realized I needed to flip the direction of the slash marks: 

```
docker cp ./jobs/jars/WordCount.jar namenode:/app/jars/WordCount.jar

```

10. Load data in HDFS

Move data from the Linux file system into the Hadoop file system:

```
cd /
hdfs dfs -mkdir /test-1-input
hdfs dfs -copyFromLocal -f /app/data/*.txt /test-1-input/
```
This gave me "command not found: hdfs", I went back and forth for a while before getting where I needed to be:

```
(base) kassandratraieh@Kassandras-MacBook-Pro ~ % cd running-hadoop
(base) kassandratraieh@Kassandras-MacBook-Pro running-hadoop % ls
eu	wxw
(base) kassandratraieh@Kassandras-MacBook-Pro running-hadoop % cd wxw
(base) kassandratraieh@Kassandras-MacBook-Pro wxw % ls
docker-hadoop
(base) kassandratraieh@Kassandras-MacBook-Pro wxw % cd docker-hadoop
(base) kassandratraieh@Kassandras-MacBook-Pro docker-hadoop % ls
Makefile		datanode		hadoop-release		historyserver		nodemanager
Makefile-x		docker-compose-v3.yml	hadoop-standalone	jobs			resourcemanager
README.md		docker-compose.yml	hadoop.env		namenode		submit
base			hadoop			hdfs			nginx
(base) kassandratraieh@Kassandras-MacBook-Pro docker-hadoop % cd jobs
(base) kassandratraieh@Kassandras-MacBook-Pro jobs % ls
data	jars	res
(base) kassandratraieh@Kassandras-MacBook-Pro jobs % cd jars
(base) kassandratraieh@Kassandras-MacBook-Pro jars % ls
WordCount.jar
(base) kassandratraieh@Kassandras-MacBook-Pro jars % docker cp ./jobs/jars/WordCount.jar namenode:/app/jars/WordCount.jar
lstat /Users/kassandratraieh/running-hadoop/wxw/docker-hadoop/jobs/jars/jobs: no such file or directory
(base) kassandratraieh@Kassandras-MacBook-Pro jars % docker cp WordCount.jar namenode:/app/jars/WordCount.jar
```


11. Run Hadoop/MapReduce

```
hadoop jar jars/WordCount.jar WordCount /test-1-input /test-1-output
```

12. Copy results out of hdfs

```
hdfs dfs -copyFromLocal -f /app/data/*.txt /test-1-input/

```

13. See the results!

```
head /app/res/test-1-output/part-r-00000

```
This is what was given:
```
#1342]	1
#768]	1
#84]	1
$5,000)	3
&	1
($1	3
(801)	3
(By	1
(Godwin)	1
(He	1
```

