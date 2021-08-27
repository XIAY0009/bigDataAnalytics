# Map Reduce And Hadoop 

## What are Required for Big Data 
1. **Parallelism** 
   * Big data involves increasing data size
   * A single node architecture results in long processing time  
     * Due to the physical aspects of a harddisk, it will take 12 days to retrive 100TB of data for 1 single node
     * ![CPU_RAM_HD](/images/cpuRAMHD.PNG)
     * Although one could opt for SSD, however it is more expensive than traditional hard disk in terms of cost per Megabyte
   * Due to the limitationos of single node architecture, there are emerging affordable **cluster architecture** which consists of clusters of commodity Linux nodes connected with gigabit Ethernet connection
   * Cluster architectures 
     * Each rack contains 16 to 64 nodes and are connected via a rack switch where there is 1 Gbps between any pair of nodes
     * Between racks they are connected connected via an aggregator switch which has a 2 to 10 Gbps
     * One should try to do the computation close to the data to avoid transfering of the data from one node to the other 
     * Below shows an image of a cluster architecture 
     * ![Cluster_Architecture](/images/clusterArchitecture.PNG)
2. **Fault Tolerance** in Storage Infrastructure 
   * We need to build fault-tolerance into the storage infrastructure because cheap nodes fail, especially when there are many 
   * Mean time to failure for 1 node is 3 years, thereby  
     * Thereby for 1000 nodes each node will fail every year 
     * If there are 1 million nodes, then 1,000 nodes will fail every day 
   * It is thus important to build fault-tolerance into the storage infrastructure 
     * We achieve this with:  
        * Distributed File System 
        * Replicating the files multiple times 
3. **Computation Close to Data Source**
* Since the data is sitting at a certain part of the cluster and commodity network has low bandwidth, thereby 
   * Copying data over a network takes significant time 
   * One will have to avoid transferring the data to somewhere outside the rack
   * One will thereby bring the computation close to the data 
4. **Large Scale Computing**
* Because of parallelism and the requirement to build fault tolerance into the system, we will need programming with distributed system
* This is difficult, which ressults in: 
   * Adopting a data-parallel programmimg model, **MapReduce**
   * Users will write "Map" and "Reduce" functions
   * Systems handles the work distribution and fault tolerance 
   * A Hadoop Cluster is shown: 
   * ![Hadoop_Cluster](/images/hadoopCluster.PNG)
   * There could be 40 nodes within a rack, and each cluster could contain 1000 to 4000 nodes 
   * Disk crash will result in one node being unavailable 
   * If the rack switch is spoiled, then it results in a network failure, whereby
      * The entire rack will become inaccessible 
      * However the failure of the router is much less probable than a disk crash this is because: 
      * There are no mechanical parts within a router
      * For a node, if one uses the traditional hard disk, there are mechanical parts which results in a higher failure rate 
* Let's take a closer look into Haddop in the next immediate section following this **Overview of Hadoop**

## Overview of Hadoop 

### Hadoop has 2 Main Components which are: 
1. **Distributed File System**  
   * Files are stored redundantly thereby ensuring 
   * --> Data availability 
2. **MapReduce Programmimg System** 
   * Computations divided into tasks 
   * --> Restart failed task without affecting other tasks
Let's go into detail for each of these 2 components: 
3. **Distributed File System**: 
   * The files are big (100s of GB to TB) 
   * Data are kept in **chunks** spread across machines
   * Each chunk is replicated on different machines
      * Seamless recovery from disk or machine failure 
   * We bring computation directly to the data 
      * Chunk servers also serve as compute servers 
      * If the process needs to access chunk 2, then one will try to put the process in the server that contains chunk 2, instead of 
         * Putting it in a chunk server that does not contain the chunk 2 data 
   * Typical usage pattern: 
      * Data is rarely updated in place 
      * Reads and append only 
   * Filesa are split into contiguous chunks (64 to 128MB)
      * We choose the size of each chunk to be 64GB to 128MB and not smaller because when the data is being read, there will be some communication overhead, with a bigger chunk, it reduces the overhead, and thus more efficient 
      * Each chunk is replicated (usually 3 times)
      * Replicas are kept in different racks  
        * Assuming file 1 in the diagram below has been divided into 4 chunks: Chunk 1, 2, 3 and 4 
        * As an example, chunk 1, 2, and 4 have been stored in rack 1 
        * Chunk 1 has been stored in rack 1, 2 and 3 
        * If both datanodes 1, and 2 failed because of disk failure, the entire file's data is still available, this is becasue: 
           * Chunk 1, 2, 3 and 4 can still be accessed from rack 3 and 4 
           * But do note that when you have a lot of chunks, it will not be as stable, this is because:  
               * If you loose one server, the many chunks would have been lost, since in this example we have used 63- 128MB size and there are only 4 chunks, and each chunk is replicated 3 times 
      * ![Name_Nodes_Data_Nodes](/images/nameNodeDataNodes.PNG)
      * In the image above, there is the Master node a.k.a **Namenode**
        * It stores the metadata about where the chunks are located 
        * If the master node is lost, much will be affected, as we will loose the locations of the chunks, thus people do replicate it 
      * Client Library for file access  
        * Talks to the master to find chunk servers
        * Connects directly to the chunk servers to access the data
4. **MapReduce Overview**  
    * MapReduce is another component of Hadoop
    * It is a data-parallel programmimg model for clusters of commodity machines 
       * Due to the retrieval of data from harddisk is slow, thereby 
          * We want to access the data in parallel
    * MapReduce could be summarized as such: 
       * Sequentially read a lot of data 
       * Map: Extract out something that one cares about 
       * Group by Key: Sort and shuffle 
       * Reduce: Aggregate, summarize, filter, or transform 
       * Write the result 
    * MapReduce Programming Model
       * Word Count Problem
          * One has a huge text document  
          * File is too large to fit into the memory 
          * We will like to count the number of times each distinct word appears in the file 
          * For this, it is difficult to do it in a single node: 
             * Since the number of distinct words are unknown, and it is allocated on the fly, it may not be able to store in the main memory 
             * If there happens to be insufficient memory, there may be swapping of memory which is occupied by the huge text document with 
             * The distinct words and its count
             * Hence it is difficult to do it in a single machine, even if we are able to break the document up
          * Example application will be analyze web server logs to find popular URLs  
          * Diagram shows the MapReduce Word Count Programming Model  
          * ![Word_Count_MapReduce](/images/mapReduceWordCount.PNG)
             * On the left of the diagram you could see that the document is splitted up into 3 chunks
             * The 3 chunks are being passed to 3 **Map** tasks, which will each output the unique word as the key and the value 1, we call this the **(key, value)** pairs 
                * E.g.  (the, 1), (brown, 1), (ate, 1)
            * **Sort & Shuffle** will place all the same keys into the same **Reduce** 
            * **Reduce** will then sum up all the values for the unique key 
            * Data type are key-value records
            * Note that the types of keys and values are arbitrary 
            * **Map function**: (kin, vin) --> <k, v>*  
               * The input to the Map function: 
                  * Key - Offset of the text file 
                  * Value -  Content in the text file
                  * Text file takes the file offset as the key and file content as the value 
                  * It takes in a key-value pair and outputs a set of key-value pairs
                  * One Map call for each (kin, vin) pair 
                  * Example of <k, v>* is (brown, 1), (fox, 1)
            * **Reduce function**: (k, <v>*) --> <kout, vout>* 
               * All values v with the same key k are reduced together
               * The is one Reduce function call per unique key k  
            * Another representation of MapReduce word count problem is shown below
               * ![Word_Count_MapReduce](/images/mapReduceWordCount1.PNG)
            * Pseudocode for Map Function
               * For each word, emit w which is the key and 1  
               ```
               map (key, value): 
               //key: document name; value: document text 
                  for each word w in value: 
                     emit (w, 1)
               ```
            * Pseuedocode for Reduce Function 
               * 
               ```
               reduce(key, values): 
               //key: a word; count: an iterator over values
                  result = 0
                  for each count v in values: 
                    result += v
                  emit(key, result)
               ```
            * **Grouping by Key**: 
               * User defines number of Reduce tasks: R 
               * System hash function to apply to keys, producing a bucket number from 0 to R-1
               * Hash each key output by the Map task, 
                 * Put the key-value pair into one of the R local files
               * Master controller merges files from Map tasks destined for the same Reduce task as a sequence of (key, list of values) pairs
        * **MapReduce Environment**
           * Takes care of the input data partitioning
           * Schedules of the program's execution across a set of machines
           * Performs Group by Key step (Sort & Shuffle)
           * Handles machines failures
           * Manages inter-machine communication 
       * Below shows a diagram of Map, Group by Key and Reduce 
           * ![Map_GroupByKey_Reduce](/images/mapReduceDiagram.PNG)
        * Below shows a diagram of MapReduce in parallel
           * ![MapReduce_in_Parallel](/images/mapReduceInParallel.PNG)
           * Map task is seperated to different nodes
           * Partition function will send all the keys 1 and 3 to Reduce task 2 
        * **MapReduce Data Flow** 
           * The input and the output are stored on a distributed file system (FS)
           * Scheduler tries to schedule the task close to the physical storage of data 
           * Will want to push the computation to the data, so as to 
              * Minimise network usage 
           * A chunk which is replicated to 3 different places, 
              * There is a load balancing mechanism to know where the map tasks should be 
           * Intermediate results are stored in local FS of Map workers rather than pushing it directly to Reducers 
              * This is to aid recovery if a Reducer crashes
           * The output could be an input to another MapReduce task
        * **MapReduce Coordination** 
           * Master nodes takes care of the coordination: 
              * Tasks status: idle, in-progress, completed 
              * Idle tasks gets scheduled as workers become available 
           * When a map task gets completed, 
              * It sends the master node the location, and 
              * Size of its R intermediate files, one for each Reducer 
           * Master then pushes these information to the Reducers
           * Master pings the workers periodically to detect failures 
           * Below shows the MapReduce flow 
              * ![MapReduce_Flow](/images/mapReduceFlow.PNG)
           * The Master will look at the program and look at how many Map tasks there are and how many Reduce tasks there are 
           * It will assign the tasks to the nodes 
           * Each of the map workers will take the data from the chunks and process it and 
              * Workers will write it out to two intermediate files, since 
              * There are only 2 Reducers
           * If there are only 2 Reducers, the intermediate files will be either 0 or 1
           * Each Reduce worker will make the remote call and 
              * Pull the 0 file to the Reduce worker 0 
              * Pull the 1 file to the Reduce worker 1
            * Output files after Reduce function will be stored in HDFS for reliability  
        * **Dealing with Failures**
           1. Master Node Fails:  
              * Restart the entire MapReduce job 
              * This is because all the information is stored in the Master node
            2. Compute Node of a Map Woker Fails
              * Reset the completed or in-progress Map tasks at worker to idle
              * Restart all the Map tasks assigned to the node, this is because: 
                 * Map tasks stores all the intermediate outputs on their local FS  
              * Inform the Reduce workers when the tasks are rescheduled on another worker
                 * Location of the nput from the Map task has changed 
            3. Compute Node of a Reduce Worker Fails:  
              * Only reset the in-progress tasks to idle
              * Restart these Reduce tasks at another node
        * **Number of MapReduce Jobs**
           * M Map tasks, and R Reduce tasks
           * Make M much larger than the number of nodes in the cluster: 
              * One DFS chunk per map is common, this is because: 
                 * It improves dynamic load balancing, and
                 * Speeds up recovery from worker failures 
           * Usually R is smaller thnan M
              * So that there will be not so much sorting
        * **Map Output Stores in Local Disk**
           * Map output is intermediate 
           * It is to be processed by the Reduce tasks to produce the final output 
           * Once the job is completed it is to be discarded
           * Therefore storing in DFS with replication is an overkill 
           * Automatically reruns the map tasks on another node to recreate the map output if the node running the map tasks fails before the map output is consumed by the Reduce tasks
    * **Task Granularity & Pipelining** 
        * Normally map tasks are much larger than the number of machines, the reason is because:  
           * This minimizes time for fault recovery   
              * Since if one map task takes in a huge chunk of data and it fails, then time to recover will be long
              * However the number of map tasks cannot be smaller than the number of chunks, if not many map tasks will  be competiting for the same chunk 
           * One can do pipeline shuffling with map execution 
              * Please see image below, once the Map task 1 is completed, the worker could start reading
           * Better dynamic load balancing 
        * Note that the Map and Reduce function could be running on the same node, but
           * It has to wait for the Map function to complete first 
        * Should a worker completes first, it will be assigna  new map task, however, 
           * The assignment of the task should be to the worker that has the data, if not there will be a long transfer time 
        * ![pipelining](/images/pipelining.PNG)
    * **Refinement: Backup Tasks**  
        * Slow workers significantly lengthen job completion time, and this could be due to: 
           * Other jobs on the machine 
           * Bad disks 
        * Solution: 
           * Near the end of phrase, spawn backup copies of tasks and takes the results of whichever that completes first 
           * This helps to shorten job completion time 
    * **Refinement: Combiners**
        * A Map task often produce many pairs of the form (k, v1), (k, v2), etc. for the **same** key
           * E.g. Popular words in the word count example
        * A combiner is a  **local** aggregation function for repeated keys produced by the same map: 
           * For associative operations such as sum, count, max
           * It helps to decrease the size of the intermediate data, thereby saving the network usgae
           * In addition, less data needs to be shuffled
           * Since the intermediate result is stored in the local file system, by decreasing the intermediate data, one could save the network usage since one do not need to send all these repeated keys over to the Reducer 
        * E.g. def combiner(key, values): output(key, sum(values))
    * **Refinement: Partition Function**
       * We use the hash function to decide which Reducer should be handling the particular Map output 
       * By modifying the hash function, we will like to control how keys are partitioned 
       * Input to map tasks are created by contiguous splits of input file 
       * Reduce needs to ensure that the records with the same intermediate key end up at the same worker  
          * Susyem uses a default partition funcution: *hash(key) mod R* 
          * It may be useful to override the hash function: 
             * E.g. *hash(hostsname(URL)) mod R* ensures URLs from a host end up in the same output file 


## References 

* **NUS Big Data Analytics and Technologies**