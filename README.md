
# WordCount-Using-MapReduce-Hadoop

This repository is designed to test MapReduce jobs using a simple word count dataset. In this project we provide a input file and then we create a maaper and reducer logic to count the occurence of each word in the given input. There are sample input and Expected output for the sample input.

## Approach and implementation
1. Mapper Logic: We use StringTokenizer to create tokens from the input file and loop it using while loop to map all the words in the input file with key value pairs. In this mapper, it will not count characters that are smaller than 3.

2. Reducer Logic: Using the output of Mapper logic we increase a variable sum value as we encounter same words and retun them. this way we will get a list of words and the number of times it occured in the input file as output.

## Setup and Execution

### 1. **Start the Hadoop Cluster**

Run the following command to start the Hadoop cluster:

```bash
docker compose up -d
```

### 2. **Build the Code**

Build the code using Maven:

```bash
mvn clean package
```

### 4. **Copy JAR to Docker Container**

Copy the JAR file to the Hadoop ResourceManager container:

```bash
docker cp target/WordCountUsingHadoop-0.0.1-SNAPSHOT.jar resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
```

### 5. **Move Dataset to Docker Container**

Copy the dataset to the Hadoop ResourceManager container:

```bash
docker cp shared-folder/input/data/input.txt resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
```

### 6. **Connect to Docker Container**

Access the Hadoop ResourceManager container:

```bash
docker exec -it resourcemanager /bin/bash
```

Navigate to the Hadoop directory:

```bash
cd /opt/hadoop-3.2.1/share/hadoop/mapreduce/
```

### 7. **Set Up HDFS**

Create a folder in HDFS for the input dataset:

```bash
hadoop fs -mkdir -p /input/data
```

Copy the input dataset to the HDFS folder:

```bash
hadoop fs -put ./input.txt /input/data
```

### 8. **Execute the MapReduce Job**

Run your MapReduce job using the following command: Here I got an error saying output already exists so I changed it to output1 instead as destination folder

```bash
hadoop jar /opt/hadoop-3.2.1/share/hadoop/mapreduce/WordCountUsingHadoop-0.0.1-SNAPSHOT.jar com.example.controller.Controller /input/data/input.txt /output1
```

### 9. **View the Output**

To view the output of your MapReduce job, use:

```bash
hadoop fs -cat /output1/*
```

### 10. **Copy Output from HDFS to Local OS**

To copy the output from HDFS to your local machine:

1. Use the following command to copy from HDFS:
    ```bash
    hdfs dfs -get /output1 /opt/hadoop-3.2.1/share/hadoop/mapreduce/
    ```

2. use Docker to copy from the container to your local machine:
   ```bash
   exit 
   ```
    ```bash
    docker cp resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/output1/ shared-folder/output/
    ```
3. Commit and push to your repo so that we can able to see your output

## Challenges Faced & Solutions

**1. Maven build errors due to `tools.jar`**  
- *Challenge:* The initial Maven build failed with an error about `jdk.tools:jdk.tools:jar:1.8` because it was trying to resolve `tools.jar` from a JRE instead of a full JDK.  
- *Solution:* Installed a proper JDK, set the `JAVA_HOME` environment variable, and ensured `%JAVA_HOME%\bin` was at the top of the system `Path`. After that, Maven recognized the JDK and built successfully.

**2. Docker command not recognized**  
- *Challenge:* Even though Docker Desktop was running, the `docker` command was not available in PowerShell.  
- *Solution:* Added Docker’s `resources\bin` folder to the system `Path` so that `docker.exe` could be run from the terminal. Once this was fixed, I was able to bring up the Hadoop cluster with `docker compose up`.

**3. ClassNotFoundException when running Hadoop job**  
- *Challenge:* Running the Hadoop jar initially failed with `ClassNotFoundException: com.example.Controller`.  
- *Solution:* The main class was actually in the `com.example.controller` package (`Controller.java`). By using the fully-qualified class name `com.example.controller.Controller` when invoking the Hadoop job, the program executed correctly.

**4. Retrieving output from HDFS**  
- *Challenge:* After the job ran, I had difficulty accessing the output because the reducer results were inside HDFS and the copied folder structure didn’t match expectations.  
- *Solution:* Used `hdfs dfs -cat` inside the container to dump the reducer output into a single file (`/tmp/part-r-00000`) and then copied it back to the local machine with `docker cp`. This gave me the final `results/part-r-00000` file required for submission.


## Sample Input: 
 ```bash
  Cloud computing makes data analysis easier and faster.
  Hadoop provides distributed storage and processing for big data.
  Data analysis with MapReduce helps extract useful information.
  Students learn cloud computing and big data in this course.
  MapReduce jobs count words and compute their frequency.
   ```

## Obtained output: 
 ```bash
and     4
data    2
big     2
analysis        2
MapReduce       2
computing       2
data.   1
with    1
makes   1
useful  1
Students        1
faster. 1
storage 1
cloud   1
Hadoop  1
frequency.      1
processing      1
provides        1
this    1
compute 1
Cloud   1
Data    1
course. 1
jobs    1
for     1
easier  1
helps   1
their   1
count   1
extract 1
learn   1
information.    1
words   1
distributed     1
   ```


