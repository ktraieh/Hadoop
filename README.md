# Hadoop

1. Make a working directory, somewhere

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

I didn't have my Docker running and had to start it.

```
docker run hello-world
```

4. Change version - we need to use docker-compose-v2, but the docker-compose command will look for an unversioned file.

```
mv docker-compose.yml docker-compose-v1.yml
mv docker-compose-v3.yml docker-compose.yml
```

This will draw two warnings - you can fix the files or ignore both.


5. Bring the Hadoop containers up (took me 81.6s). I had previously pulled other hadoop images though so your time may vary.

```
docker-compose up -d
```

6. In a Hadoop cluster, we mostly work within the namenode. 

I used "docker ps" to list the nodes, then picked the one with name in it. For me:

```
docker ps
CONTAINER ID   IMAGE                                                    COMMAND                  CREATED         STATUS                            PORTS      NAMES
de4a8afd3940   wxwmatt/hadoop-historyserver:2.1.1-hadoop3.3.1-java8     "/entrypoint.sh /run…"   6 seconds ago   Up 4 seconds (health: starting)   8188/tcp   docker-hadoop-historyserver-1
a0e551086c0c   wxwmatt/hadoop-namenode:2.1.1-hadoop3.3.1-java8          "/entrypoint.sh /run…"   6 seconds ago   Up 4 seconds (health: starting)   9870/tcp   docker-hadoop-namenode-1
7b70e2a0bbb3   wxwmatt/hadoop-nodemanager:2.1.1-hadoop3.3.1-java8       "/entrypoint.sh /run…"   6 seconds ago   Up 5 seconds (health: starting)   8042/tcp   docker-hadoop-nodemanager-1
46e45d7b9f49   wxwmatt/hadoop-resourcemanager:2.1.1-hadoop3.3.1-java8   "/entrypoint.sh /run…"   6 seconds ago   Up 4 seconds                      8088/tcp   docker-hadoop-resourcemanager-1
a31b6cbaa2d8   wxwmatt/hadoop-datanode:2.1.1-hadoop3.3.1-java8          "/entrypoint.sh /run…"   6 seconds ago   Up 5 seconds (health: starting)   9864/tcp   docker-hadoop-datanode-1
```

So I used:

```
docker exec -it docker-hadoop-namenode-1 /bin/bash
```

I also tested `hello world` here:

```
echo hello world
```

7. Set up the node

While within the node, we will need directories for data, for compute, and for results. I created those under a new 'app' directory.

```
mkdir app
mkdir app/data
mkdir app/res
mkdir app/jars
```

8. Fetch data to app/data

'hello world' for 'map reduce' is 'word count' so we get some words to count. I got two of my favorite books and also Wuthering Heights from Project Gutenberg in plaintext format like so:

```
cd /app/data
curl https://raw.githubusercontent.com/cd-public/books/main/pg1342.txt -o austen.txt
curl https://raw.githubusercontent.com/cd-public/books/main/pg84.txt -o shelley.txt
curl https://raw.githubusercontent.com/cd-public/books/main/pg768.txt -o bronte.txt
```

It should be easy enough to find other text files on the Internet, but I used these three. Before going further, I verified that I had files of some size:

```
ls -al
```

I got: 

```
total 1884
drwxr-xr-x 2 root root   4096 May 28 19:42 .
drwxr-xr-x 5 root root   4096 May 28 19:38 ..
-rw-r--r-- 1 root root 772420 May 28 19:41 austen.txt
-rw-r--r-- 1 root root 693877 May 28 19:42 bronte.txt
-rw-r--r-- 1 root root 448937 May 28 19:42 shelley.txt
```

The byte size is between the second "root" and the date - and all three are nonzero and similarish in size (as each is a novel).

9. Fetch compute to app/jars

We are using WordCount.jar, which is helpfully provided by somebody, but most critically is in the repo. We can grab from the local file system (boring) or we just curl again:

```
cd /app/jars
curl https://github.com/wxw-matt/docker-hadoop/blob/master/jobs/jars/WordCount.jar -o WordCounter.jar
```

In general, you should not just pull executable files off of the internet and run them, but we are running this in a container, which provides a modicum of safety.

If you don't/can't/won't do a curl, you just take the local jar, though you'll have to drop out of the container to grab it (unless there's some docker commands I don't know).

```
docker cp .\jobs\jars\WordCount.jar namenode:/app/jars/WordCount.jar
```

10. Load data in HDFS

Hadoop can only read and write to something called the Hadoop Distributed File System, which is a very fancy hash table that works well in data centers.

We need to move data from the Linux file system into the Hadoop file system. We use the "hdfs" commands.

```
cd /
hdfs dfs -mkdir /test-1-input
hdfs dfs -copyFromLocal -f /app/data/*.txt /test-1-input/
```

11. Run Hadoop/MapReduce

```
hadoop jar jars/WordCount.jar WordCount /test-1-input /test-1-output
```

12. Copy results out of hdfs

```
hdfs dfs -copyToLocal /test-1-output /app/res/
```

13. See the results!

```
cat /app/res/test-1-output/part-r-00000
```

14. I did this probably around 20 times and it worked about 2 (so 10%) and I couldn't figure out why it worked sometimes and not others. Here's some tricks:

* Run in Unix rather than on Windows
* Use --remove-orphans with docker compose
* Make sure names are all the same everywhere
* Use ls, ls -al, and cat liberally to check files, hdfs and otherwise
* Use docker cp and curl and verifying the similarity with diff
* Check the following:
	- https://hub.docker.com/r/apache/hadoop
	- https://github.com/big-data-europe/docker-hadoop
	- https://github.com/wxw-matt/docker-hadoop
