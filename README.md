# Spotify Artist Popularity MapReduce Analysis

## Overview

This project uses Hadoop MapReduce (via Hadoop Streaming and Python scripts) to analyze the Spotify dataset.  
**Goal:** Compute the average number of followers for each artist popularity score.

## Project Contents

- **Document (PDF/Word):** Full report including implementation, execution, results, and conclusion.
- **README file:** (this file) Step-by-step instructions for setup and execution.
- **Source code:** `mapper.py` and `reducer.py` (in this repo or attached as zip).
- **Dataset file:** `spotify_dataset.csv` (from Kaggle, or as provided).
- **Results summary and evidence:** Screenshots/logs/video (see report).

## Prerequisites

- Ubuntu/Linux system
- Java (JDK)
- Hadoop 3.x (single-node setup)
- Python 3

## 1. Hadoop & Environment Setup

### **Install Java**

sudo apt-get update
sudo apt-get install -y default-jdk
java -version

### **Create a Dedicated Hadoop User**

sudo adduser hadoop
sudo usermod -aG sudo hadoop
sudo su - hadoop

### **Configure SSH for Hadoop User**

sudo apt-get install openssh-server openssh-client -y
ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
ssh localhost

### **Download and Install Hadoop**

wget https://downloads.apache.org/hadoop/common/hadoop-3.4.0/hadoop-3.4.0.tar.gz
tar -xzf hadoop-3.4.0.tar.gz
sudo mv hadoop-3.4.0 /usr/local/hadoop
sudo mkdir /usr/local/hadoop/logs
sudo chown -R hadoop:hadoop /usr/local/hadoop

### **Set Hadoop Environment Variables**

Edit `~/.bashrc` and add:

export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME

Apply changes:
source ~/.bashrc

### **Configure Hadoop**

Edit the following files in `$HADOOP_HOME/etc/hadoop/`:

- `core-site.xml`
- `hdfs-site.xml`
- `mapred-site.xml`
- `yarn-site.xml`
- `hadoop-env.sh` (set JAVA_HOME)

### **Format the NameNode**

hdfs namenode -format

### **Start Hadoop Services**

start-dfs.sh
start-yarn.sh
jps

### **(If clusterID error occurs)**

stop-all.sh
rm -rf /home/hadoop/hadoopdata/hdfs/datanode/\*
start-all.sh
jps

## 2. HDFS Setup and Data Upload

### **Verify Hadoop Installation**

- NameNode UI: [http://localhost:9870](http://localhost:9870)
- ResourceManager UI: [http://localhost:8088](http://localhost:8088)

### **Test HDFS Commands**

hdfs dfs -mkdir /test
hdfs dfs -ls /

### **Upload the Dataset**

hdfs dfs -mkdir -p /user/hadoop/spotify
hdfs dfs -put /home/malith/Desktop/assignment_1/spotify_dataset.csv /user/hadoop/spotify/
hdfs dfs -ls /user/hadoop/spotify

## 3. Prepare and Run MapReduce Job

### **Restart Hadoop Services if Needed**

stop-all.sh
start-all.sh

### **Copy Python Files and Make Executable**

cp /home/malith/Desktop/assignment_1/mapper.py /tmp/
cp /home/malith/Desktop/assignment_1/reducer.py /tmp/
chmod +x /tmp/mapper.py /tmp/reducer.py

### **Remove Previous Output Directory**

hdfs dfs -rm -r /user/hadoop/spotify/output

### **Run the MapReduce Job**

hadoop jar $HADOOP_HOME/share/hadoop/tools/lib/hadoop-streaming-\*.jar \
 -input /user/hadoop/spotify/spotify_dataset.csv \
 -output /user/hadoop/spotify/output \
 -mapper /tmp/mapper.py \
 -reducer /tmp/reducer.py

## 4. View and Collect Results

### **View Output in Terminal**

hdfs dfs -cat /user/hadoop/spotify/output/part-\*

### **View First 20 Lines**

hdfs dfs -cat /user/hadoop/spotify/output/part-\* | head -n 20

### **Copy Output Files to Local Directory**

mkdir -p /home/hadoop/spotify_output
hdfs dfs -get /user/hadoop/spotify/output/part-\* /home/hadoop/spotify_output/

### **Merge All Output Files into a Single Local File**

hdfs dfs -getmerge /user/hadoop/spotify/output/ /home/hadoop/spotify_output.txt

## 5. Results

- The output file(s) will contain lines of the form:
- Example:
  0 52.68
  10 642.03
  100 44606973.00

## 6. Troubleshooting

- **Permission Denied:**  
  Ensure `mapper.py` and `reducer.py` are executable (`chmod +x ...`).
- **Output Directory Exists:**  
  Remove it with `hdfs dfs -rm -r `.
- **No Output:**  
  Test scripts locally with `head -n 10  | ./mapper.py`.
- **clusterID Error:**  
  Reset DataNode as shown above.

## 7. References

- [Kaggle Spotify Dataset](https://www.kaggle.com/datasets/lehaknarnauli/spotify-datasets?select=artists.csv)

## 8. Contact

EG/2020/3797 - Aberuwan R. M. M. P.
EG/2020/4190 - Sandamal I.M.K.
EG/2020/4219 - Siriwardhana T.D.R.D.
