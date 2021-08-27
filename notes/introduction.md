# Introduction to Big Data 

## What is Big Data? 

* Data is considered as big when the **volume**, **velocity** or **variety** exceeds the abilities of what modern IT systems is capable of ingesting, store, analyze and process to derive **actionable** insights in a **timely** manner 

* Big data has 5 key main charactistics are they are:  
    1. **Volume** 
     * Large amount of data, and it could reach zettabytes
     * E.g. Posts to social media, number of pictures uploaded on Flickr  
    2. **Velocity** 
     * Streaming instead of batch data 
     * Requires real time analysis such that the decisions are valid, if not there will be missed opportunities
     * E.g. Prediction of traffic jam
    3. **Variety**
     * Semistructured & unstructured data
     * Normally does not fit into neat tables of columns and rows and it is better to place them in Hadoop Distributed File System (HDFS) or non-relational NoSQL databases 
     * E.g. Video, images, spatial data 
    4. **Veracity** 
     * Concerns on the reliability of the data source 
     * If there are bias, inconsistencies, its accuracy and trustworthiness
    5. **Value**
     * The data sahould be able to derive actionble insights

## Why Big Data? 

* Advancement in hardware technology
  * Parallel processing made possible 
* Automation such that 
  * Data could be collected more cheaply  
* Falling media prices
  * Data could be stored cheaply 
* Data is able to create value and knowledge
  * Deriving product sentiment analysis
  * Meter readings better able to predict power consumption

## How to Derive Value from Big Data? 

* One has to process the raw data to retrieve actionable information 
* Its the process of examining large and varied datasets to uncover: 
    * Hidden patterns, market trends, customer preferences 
    * Which helps organizations make better informed business decisions
* Data has to be stored, managed and analysed
* Big Data Process: 
    * **Discover**: 
      * Understand what data is available
      * What are the implicit assumptions in the data 
    * **Explore**: 
      * Make hypothesis on how the data relates to each other 
    * **Iteration**: 
      * Hypothesis may not be accurate, thus one has to test and
      * Find out the actual relationship 
* Discover patterns and models that are: 
     * **Valid**: 
       * Applies to new data 
     * **Useful**: 
       * Actionable 
     * **Unexpected**: 
       * Non-obvious to the system 
     * **Understandable**: 
       * Patterns should be interpretable
       * Explainable

## What are Examples of Data Analytics Task? 

* **Descriptive Methods**
     * Find human interpretable patterns that describe the data 
     * E.g. Clustering 
* **Predictive Methods**
     * Use variables to predict unknown or future values of other variables 
     * E.g. Recommender System 
* Data Analytics Pipleine Involves: 
     * **Data Collection**
         * Acquiring data from variety of sources
     * **Data Curation**
         * Clean, format, integrate with other datasets, store in the database
         * Data cleaning is important because of duplicate recordes, entity resolution, conflict resolution, missing values, outliers
     * **Data Processing** 
         * Runs queries, plot graphs 
     * **Data Analysis**
         * Examine trends and anomalies, understand the results 
     * At any point in this pipeline, you may have to return to previous steps: 
       * E.g. During data analysis, you suspect that you are missing some data, you will have to get back to data collection
       * E.g. During data processing, you found out that the data could have been formatted in another way, you then go back to data curation. 






