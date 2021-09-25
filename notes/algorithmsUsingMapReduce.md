# Algorithms Using MapReduce

* In the previous section [**Map Reduce And Hadoop**](/notes/mapReduceAndHadoop.md)  
   * The MapReduce programming model which is programming with distributed system was introduced 
   * We need MapReduce because of the requirement of achieving: 
      * Parallelism and to build fault tolerance into the system 
* In this section, we will explore using MapReduce for matrix multiplications as well as for processing relation operators

## 1. Matrix-Vector Multiplication 
* Given a matrix M and a vector V with the following sizes as shown in the diagram below: 
   * Matrix M: 
      * Dimensions n x n 
      * m<sub>ij</sub> denotes element at row i and column j  
   * Vector v: 
      * length n 
      * v<sub>j</sub> denotes the jth element 
   * Matrix vector multiplication is defined as: 
      *  > $$ \sum m_{ij}v{j} $$ 
   * For each ith row, it represents url i, and m<sub>ij</sub> will be 1 if i and j urls are linked  
   * v<sub>j</sub> represents the importance of the url j 
   * ![MatrixMultiplication](/images/matrixMultiplication.PNG)
   * Example application: 
      * Rank web pages in search engines, where there are tens of billions of them 
      * Matrix represents the links in the webpage, and
         * m<sub>ij</sub> is non-zero if there is a link between i to j 
* Matrix-Vector Multiplication with MapReduce 
   * Matrix M and vector v are stored in seperate files in Distributed File System DFS
   * Compute node executing map task read vector v 
   * Each map task operates on a chunk of matrix M 
   * **Map** Function:  
      * Applies to one element of (i, j, m<sub>ij</sub>) of m 
      * Produces key-value pair (i, m<sub>ij</sub>v<sub>j</sub>)
      * All terms of the sum that makes up the component x<sub>i</sub> of the matrix-vector product will get the same key i 
   * **Reduce** Function: 
      * Sum up all the values associated with a given key i 
      * Result is a pair (i, x<sub>i</sub>) 
* **What if the Vector Could Not Fit Into the Memory?**  
   * Divide M into **vertical** strips of equal width 
   * Divide v into equal number of **horizontal** stripes of the same height 
   * Vector in one strip can fit into the memory 
   * Multiply the ith stripe of M with components from the ith stripe of v 
   * One file for each stipe of matrix and vector 
   * Each map task is assigned a chunk from one matrix strip, and corresponding vector strip 
   * ![MatrixVectorStrip](/images/matrixVectorStrip.PNG)

## 2. Relational Algebra Operations  
* A relation represents a database table
* Major difference between SQL and relational algebra is that: 
   * In relational algebra duplicate rows are implicitly eliminated 
   * This is not the same with SQL implementations
* Many operations on data can be described in terms od database query primitives 
* **Relation Links** describe Web Structure 
* Two attributes **From** and **To**
* Row or tuple is a pair of URLs such that there is at least a link from the first URL to the second as shown below: 
* ![LinksTuples](images/linksTuples.PNG)
* Relation algebra operations with MapReduce: 
   * Relations are stored as a file in the DFS  
   * Elements of the files are tuples of the relation  
   * We can perform standard operations on the relations
      * Selection 
      * Projection 
      * Natural Join 
      * Grouping and Aggregation 
   * Each of these operations will be discussed in detail in the subsequent section 
* I. **Selection $\sigma_{C}(R)$**  
   * In SQL: 
      * Selection(WHERE clause in SQL) lets you apply a condition over the data you have and
      * Only get the rows that satisfy the condition
      * ![SelectionSQL](/images/selectionSQL.PNG)
    * In MapReduce: 
       * Apply condition C to each tuple in relation R and output only those tuples that satisfies C
       * **Map** Function: 
          * For each tuple t in R, test if it satisfies C
          * If yes, output key-value pair (t, t)
          * Key and value are the same
       * **Reduce** Function [optional]:
          * Has nothing to do 
          * Simply writes the value for each key it receives to the output 
       * E.g. If the condition was to select **To** attributes to have url 3, then  
          * Only rows two and three are selected 
          * Map has already done the selection, and it outputs the tuples, thus
          * Reduce function is optional
       * Data we have initially distributed as files in Map workers: 
          * ![DataDistribution](/images/dataDistributionInFiles.PNG)
       * Application of Map function, outputs are shown below:  
          * Tuples are constructed with 0th index containing values from A column and 1st index containing values from B 
          * ![MapStageSelection](/images/selectionMap.PNG)
       * Assuming 2 Reduce workers
          * A hash function is applied, and below is the output 
          * ![ReduceStageSelection](/images/selectionReduce.PNG)
       * Final output after applying the Reduce function and ignoring the Keys shown below: 
          * ![FinalOutputSelection](/images/selectionOutput.PNG)
* II. **Projection $\pi_{S}(R)$**  
   * In SQL: 
      * To select some columns only, one uses the projection operator, analogous to SELECT
      * ![ProjectionSQL](/images/projectionSQL.PNG)
   * In MapReduce: 
      * Given a subset S of attributes of a relation R, 
         * Produce from each tuple only the values for the attributes in S 
      * **Map** Function: 
         * For each tuple in R, construct t' by eliminating values of attributes that are not in S
         * Outputs key-value pair (t', t')
      * **Reduce** Function: 
         * Eliminates duplications, just obtains the value at the 0th index
         * (t', [t', t', ..., t']) -> (t', t')
         * Operation is associative and commutative  
            * One could use a combiner with each Map task to eliminate duplicates produced locally 
            * Reduce tasks are still needed to eliminate duplicates from different Map tasks
      * Below is an example on how it is being done: 
         * Computing projection (A, B) for the following table: 
            * ![Projection Data Distribution](/images/projectionData.PNG)
         * After application of Map function, and grouping the keys the data will look like the following: 
            * ![Projection Map](/images/projectionMap.PNG) 
            * At this stage, one could have done the combiner at the map task to eliminate duplications produced locally, but in this example it was not done 
         * After partitioning using a hash function, the data looks like the following:  
            * ![Projection Partition](/images/projectionPartition.PNG)
         * At each of the Reduce workers the data will be as follows: 
            * ![Projection Reduce Workers](/images/projectionReduceWorkers.PNG)
         * At the Reduce nodes, the keys will be aggregated again, as 
            * The same keys might have occurred at multiple map workers
            * Reduce function is applied which will consider only the first value of the values  list and ignores the rest of the information
            * ![Projection Output](/images/projectionOutput.PNG)
* III. **Natural Join** 
   * In SQL: 
      * Merge two tables based on some common column
      * INNER JOIN in SQL 
      * The condition is implicit on the column that is in both tables 
      * Output will only contain rows for which the values in the common column matches 
      * It will generate one row for every time the column value across two table matches 
      * ![Natural Join](/images/naturalJoin.PNG)
      * Join can explode the number of rows we have, since 
          * We have to form each and every possible combinations of the values from both tables 
   * In MapReduce: 
      * Join R(A, B) and R'(B,C)
         * Find tuples that agree on the B attribute
         * Find the B value of tuples as Key, and 
         * Value will be the relation (table) name and other attributes
         * We will need the relation name so that the Reduce function knows where the tuple was from 
         * Look out for what you want to join on, and that will be the key 
      * **Map** Function 
         * Each tuple (a, b) of R --> (b, (R, a))
         * Each tuple (b, c) of R' --> (b, (R', c))
      * **Reduce** Function 
         * Each key value b will be associated with a list of pairs which are either: 
            * (b, (R, a)) or (b, (R', c))
         * Match all pairs (b, (a, R)) with all (b, (R', c)) and 
            * Outputs (a, b, c)
    * Example Application: 
       * Use Relation links to find paths of length 2 in Web 
       * Triples of URL (u, v, w) such that there is a link from u to v and from v to w 
       * ![Natural Join of Links](/images/naturalJoinLinks.PNG)
       * Natural Join link output is shown below:
       * ![Output of Natural Join](/images/naturalLinkOutput.PNG)
    * Below is an example of how it is done: 
       * Let's consider joining Table 1 and Table 2, and B is the common column 
          * ![Natural Join Data](/images/naturalJoinData.PNG)
       * After application of the Map function and grouping of keys: 
          * ![Natural Join Map](/images/naturalJoinMapAndGroup.PNG)
       * Files constructed for Reduce workers: 
          * ![Natural Join Prepared for Reduce](/images/naturalJoinPartition.PNG)
       * Data at each of the Reduce workers: 
          * ![Data at Each Reduce Worker](/images/dataAtEachReduceWorker.PNG)
       * Aggregation of keys at Reduce workers: 
          * ![Data Aggregation](/images/dataAggregation.PNG)
       * After applying the Reduce function which will create a row by taking one row from T1 and the other from T2  
          * If there are **only** values from T1 **or** T2 in the value list, that won't constitute a row 
          * ![Reduce Output](/images/naturalJoinOutput.PNG)
* IV. **Grouping and Aggregation $\gamma_{A, \theta(b)}(R)$** 
   * In SQL: 
      * Group rows based on set of columns and 
      * Apply aggregation (sum, count, max, min, etc.) on some column of the small groups that are formed 
      * Correspond to GROUP BY in SQL
    
   * In MapReduce:
      * Given relation R(A,B,C)
         * Partition the tuples according to the values of attribute A (grouping attribute)
         * For each group, aggregate values in attribute B 
      * **Map** Function: 
         * Each tuple (a,b,c) -> key-value pair (a,b)
      * **Reduce** Function performs aggregation: 
         * Each key represents a group 
         * Apply the aggregate operator $\theta$ to list [b<sub>1</sub>, b<sub>2</sub>, ..., b<sub>n</sub>] of B values associated with key a 
         * Output is a key-value pair (a,x) where x is the result of applying the aggregator function to the list 
            * If operator is SUM: x = b<sub>1</sub> + b<sub>2</sub> + ... + b<sub>n</sub>
            * If operator is MAX: x = MAX(b<sub>1</sub>, b<sub>2</sub>, ... , b<sub>n</sub>)
    * Example Application:
       * Social network site has relations Friends (User, Friend)
       * Tuples are pairs (a, b) such that b is a friend of a
       * Statistics on number of friends each member has: 
          * > $$\gamma_{User, COUNT(Friend)}(Friends)$$
       * Group all the tuples by User -> one group for each User 
       * For each group, count the number of friends, e.g. (Sally, 300)
    * Below is an example on how this is done: 
       * Take the attribute which grouping is to be done as the key, and 
       * Values will be the ones which aggregation is to be performed 
       * If the relation has columns A, B, C, D, and
         * If we want to group by A, B and do aggregation on C
         * (A, B) will the key and C the value 
       * Below, we group by (A, B) and apply sum as the aggregation 
         * ![Initial Data](/images/groupAggData.PNG)
       * Application of amp function and group by (A, B) as key and C as value and discards D
         * ![Data at Map](/images/groupAggMap.PNG)
       * After partitioning using hash functions: 
         * ![Partition at Map](/images/groupAggPartition.PNG)
       * Data is grouped together based on the keys before applying the aggregation function 
          * ![Group Values by Keys](/images/groupAggKey.PNG)
       * Applies the aggregation function over the value list, obtains the final output as follows: 
          * ![Aggregates over Value List](/images/groupAggAggregates.PNG)
* **Matrix Multiplication Using Relational Algebra Operations** 
   * Maps each matrix into one relation table
   * Given two matrix M with element m<sub>ij</sub> and matrix N with element n<sub>jk</sub>
   * Product P = MN is the matrix multiplication of M and N with element p<sub>ik</sub> and is given by: 
      * > $$p_{ik} = \sum_{j} m_{ij}n{jk} $$
   * Matrix as a relation with 3 attributes: 
      * Row number
      * Column number 
      * Value 
    * E.g. Relation M(I, J, V) with tuples (i, j, m<sub>ij</sub>)
    * E.g. Relation N(J, K, W) with tuples (j, k, n<sub>jk</sub>)
   * Product MN is a naturall join with common attribute j followed by grouping and aggregation 
      * Implemented as a cascade of two MapReduce operations
   * Join M(I, J, V) and N(J, K, W)
      * Find tuples that agree on attribute J 
      * Produces tuples (i, j, k, v, w)
      * Five component matrix (i, j, k, v, w)
         * Represents the pair of matrix (m<sub>ij</sub>,n<sub>jk</sub>)
      * 4 component matrix (i, j, j, v X w)
         * Represents the product of m<sub>ij</sub>n<sub>jk</sub>
   * Performs grouping and aggregation 
      * Use I, K as grouping attribute and $\theta$ as sum of V x W as aggregation function 
   * **Map** Function: 
      * Each matrix element m<sub>ij</sub> produces (j, (M, i, m<sub>ij</sub>))
      * Each matrix element n<sub>jk</sub> produces (j, (N, k, n<sub>jk</sub>))
   * **Reduce** Function: 
      * Each key j will be associated with a list of values that are either: 
         * (M, i, m<sub>ij</sub>)) or 
         * (N, k, n<sub>jk</sub>))
      * Match all pairs (M, i, m<sub>ij</sub>) and (N, k, n<sub>jk</sub>) to produce key value pair: 
        * Key (i,k) and 
        * Value m<sub>ij</sub>n<sub>jk</sub>
   * Perform grouping and aggregate function with another MapReduce function
   * **Map** Function: 
      * Map function is an identity 
   * **Reduce** Function: 
      * Reduce function sums the list of values associated with the key (i,k)
      * Result is a pair ((i,k), v) where v is the value of the element of the product  

## References 

* **NUS Big Data Analytics and Technologies**
* [**Relational Operations Using MapReduce**](https://medium.com/swlh/relational-operations-using-mapreduce-f49e8bd14e31)


         



    
          
        


