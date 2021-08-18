# Introudction to Spark and Installation 

## Overview of Spark and Why Use It 

At a high level, every Spark application consists of a **driver** program that runs the user's main function and executes various **parallel** operations on a cluster. Advantages of Spark includes: 

1. **Resilient Distributed Dataset (RDD)**
* RDD is a collection of elements partitioned across the nodes of the clusters which could be operated on in parallel
* Users may choose to ask Spark to persist an RDD in memory, thereby allowing it to be reused efficiently across parallel operations
* RDDs automatically recovers from node failures 

2. **Shared Variables**
* Spark provides shared variables that can be used in parallel operations 
* When Spark runs a function in parallel as a set of tasks on different nodes, it ships a copy of each variable used in the function to each task
* Sometimes, a  variable needs to be shared across tasks, or between tasks and the driver program 
* Spark supports 2 types of shared variables: 
    * Brodcast variables: Used to cache a value in memory on all nodes
    * Accumulator variables: Variables that are only added to counters and sums

## Setting up Spark 

0. Optional: Use docker to ensure a clean environment for the installation 
```bash 
docker pull ubuntu:16.04
```
* Start the docker with the following command 
```bash
docker run --rm -it \
--net=host \
-e DISPLAY=$DISPLAY \
-v $HOME/.Xauthority:/root/.Xauthority:rw \
--name minnie \
-v $PWD:/workspace \
ubuntu:16.04                                                       
```
* If this is successful, you should be inside the docker 

1. Verify if you have already installed java by typing 
```bash 
java --version
```
* If java has not already been installed, follow the below steps to get it installed. 
* If you have installed a brand new docker, please follow these steps, if not you could proceed to adding the java respository
```bash 
apt-get update
apt-get install software-properties-common
```
* Installing java 
```bash 
sudo apt-get install openjdk-8-jdk 
```
* After successful installation, set the environment by editing /etc/environment with your favorite editor
```bash 
vi /etc/environment
```
* Insert the following line in: *JAVA_HOME="/usr/lib/jvm//usr/lib/jvm/ java-8-openjdk-amd64*

2. Installs Scala 
```bash 
apt-get install wget
wget www.scala-lang.org/files/archive/scala-2.12.4.deb
dpkg -i scala-2.12.4.deb 
```
3. Installs Maven (This is to Compile Java Files)
* Download [Maven](https://maven.apache.org/download.cgi) and apache-maven-3.5.2-bin.tar.gz was download
* ![Download Maven](images/downloadingMaven)
* untar the downloaded with the following command
```bash 
tar xvzf apache-maven-3.5.2-bin.tar.gz 
```
* Similar as Scala, move the apache-maven-3.5.2 folder to /user/local/maven. If you are not in docker,, please go to the root account with *su -* and execute the move command, after which *exit* the root user mode 
```bash 
mv apache-maven-3.5.2 /usr/local/maven 
```
* As you may have guessed by now, edit the path for Maven 
```bash 
vi /etc/environment 
```
* Append the location of Maven at the end of the *PATH* variable and save the file: */usr/lcoal/maven/bin* 

4. Installs Spark 
* Download [Spark](https://spark.apache.org/downloads.html) and choose to download spark-2.2.1-bin-hadoop2.7.tgz
* ![Download Spark](images/downloadingSpark)
* After which similar to the above, extract the file 
```bash 
tar xvzf spark-2.2.1-bin-hadoop2.7.tgz
```
* Move the Spark files to the directory /usr/local/spark, if  you are not in docker, remember to execute this with root permission 
```bash 
mv spark-2.2.1-bin-hadoop2.7 /usr/local/spark 
```
* Lastly, please set the path for Spark 
```bash 
vi /etc/environment 
```
* Append the following at the end of *PATH* variable: */usr/local/spark/bin*

5. Save the Docker and Re-enter or Restart the System 
* If you are using the docker, please remember to save the docker image with the following command 
```bash 
docker commit -a <your name> -m "java_scala_maven_spark packages installed" <docker id> spark:v1 
```
* Re-enter the docker with the following command 
```bash 
docker run --rm -it \
--net=host \
-e DISPLAY=$DISPLAY \
-v $HOME/.Xauthority:/root/.Xauthority:rw \
--name minnieMouse \
-v $PWD:/workspace \
spark:v1
```
6. Verify the installtion of the above with the following commands 

* Verify if Java has been installed successfully 
```bash 
java -version 
```
* Verify if Scala has been installed successfully 
```bash 
scala -version 
```
* Verify if Maven has been installed successfully 
```bash 
mvn -version 
```
* Verify if spark has been installed successfully 
```bash 
spark-shell 
```
* If all the packages above have been installed successfully, you will see the below output 
    * If any of the package could not be found, you could try to type the following and try again 
    ```bash 
    source /etc/environment 
    ```
    ![Package Verification](images/packageInstallationVerification)

