---
title: 'Working With Your Biomedical KG'
date: 2026-03-07
permalink: /posts/2026/03/2026-03-07-Working-With-Your-Biomedical-KG/
tags:
  - Data Science
  - Knowledge Graphs
  - Biomedical
  - Neo4j
  - System Biology
---

As a data scientist, a student needing to perform data science tasks, you may be interested to work on Knowledge Graphs (KG). The past decades, the biomedical research community has been working on the development of Knowledge Graphs to represent complex biomedical data. In this post, we will discover how you could start working with Knowledge Graphs and Biomedical KG. 


Introduction
======

For whom?
------

This post is for anyone interested in data science tasks (data modelization, data analysis, etc.), who wants to work on KGs. To follow this post, you will need to have good programming knowledge (we may have a few command lines to do), and a good understanding of data science and data wrangling techniques.

Understanding the difference between a KG and a dataset
------
When we work in data science, we work on "data". "data" can come from many places: databases, APIs, web scraping, experiments...etc. A lot of data points are unstructured and need to be transformed through data wrangling (curation, cleaning, transformation). Once that done, we have a dataset to work with. We start making hypotheses, testing those hypotheses, fitting models to create "knowledge": association of variables, causality...etc. We therefore derive "knowledge" from our data : links between variables, as concepts, backed up with statistical evidence from our data modeling.

Knowledge is often represented as a graph: nodes and edges, with nodes representing concepts and edges representing associations between concepts. Essentially, the KG is many datasets distilled to what we learnt from them conceptually. When we work at the level of KGs, we aim at studying the topology of this graph: what are the most connected concepts, can we learn new things from what is existing? ...etc.


What is the content?
------
In this post, we will cover the following steps:
- Download Neo4j  (if you do not have it already!)
- Download a biomedical KG (if you do not have one already!)
- The different formats of biomedical knowledge graphs with a focus on .jsonl
- Importing your KG to explore through Neo4j
- Exploring your KG

This post is not meant to be exhaustive, but rather as a starting point. We will not review all places where you can get a KG, nor all existing formats, nor all ways you can explore it or use it to develop. But we will provide you with the tools and knowledge on how to get started.

Let's dive in
======

Getting a biomedical KG to work on
------
Go to: [KG registry on KGhub](https://kghub.org/kg-registry/)
For this tutorial, I will use the BioLink KG called [RTX-KG2](https://kghub.org/kg-registry/resource/rtx-kg2/rtx-kg2.html)
This biomedical KG has been [published](https://link.springer.com/article/10.1186/s12859-022-04932-3) and you should feel free to use it for your science!
There are 2 .jsonl files to download:
- ....nodes.jsonl.gz
- ....edges.jsonl.gz
Unpack them in your working directory. 

The .jsonl format
------

Importing the KG as a database
------
1. Download Neo4j

Neo4j desktop is an open-source graph visualization tool. 
Download Neo4j desktop free version [here](https://neo4j.com/download/)
It will appear, after installation in your root folder under `.Neo4jDesktop2/`

2. Creating a new, empty database in Neo4j
Now, let's create "a local graph database instance": 
On the left side panel, under Data services, click on "Local instances" and then "Create instance". You will need to give a name and a password:
```
Instance name : RTX-KG2
Password : *****
```
Click create. The instance will take a few minutes to get created and then starts RUNNING automatically. On your instance, you will see a ... button with the plugin option. Click on plugin and install the `APOC` plugin. Then restart your instance (either click on restart or stop, wait for it to stop and then play buttons).

Under your instance you can see 2 databases. The work we will do is on the one called `neo4j` (do not bother with the system one).
In your root folder `.Neo4jDesktop2/` under `Data/dbmss` you will see a new foler staring with `dbmss-....` which is your new database.
Navigate to `.Neo4jDesktop2/Data/dbmss-.../conf` and add a file **without extension** called `apoc.conf` and write inside:
```
apoc.import.file.enabled=true
```
Navigate to `.Neo4jDesktop2/Data/dbmss-.../` and create a folder called `import`

3. Chunking your KG
KGs can be very large, so it's a good idea to break your KG into smaller chunks for importing it. The size of the chunk will be dependent on your (local) RAM capacity. See section 3. to evaluate it as you may need to build smaller chunk sizes to accomodate your hardware. 

Generating your chunks
At the command line, navigate to your directory where you have stored your RTX-KG2 nodes and eges file and run:
```
$ split -l 100000 nodes_c.jsonl nodes_c_chunk_
$ split -l 100000 edges_c.jsonl edges_c_chunk_
```
This will generate files like:  edges_chunk_aa,edges_chunk_ab, ...., edges_chunk_az, edges_chunk_ba...
Each chunk will have 100000 edges or nodes (in .jsonl, 1 line is 1 element).

We can see at the end :
- nodes chunks go up to `co` chunk
- edges chunks go up to `kj` chunk
 
Transfer all your chunks to `.Neo4jDesktop2/Data/dbmss-.../import/`

4. Evaluate the memory needed for Neoj DB:
In `.Neo4jDesktop2/Data/dbmss-.../import/`, look at the max chunk size: here it is ~330MB. Per Chunk processing, you will need about **3x this amount** so ~1GB of RAM.

Open `.Neo4jDesktop2/Data/dbmss-.../conf.neo4j.conf` and change the following variables:
dbms.memory.transaction.total.max=2g
server.memory.heap.initial__size=4g
server.memory.heap.max__size=8g

If your DB is running, stop and restart it to apply changes.

5. Importing your KG into the empty database
Connecting to your database will enable you to run queries on your database.
Click on `Connect to instance` select the `RTX-KG2` and click `Connect`.

Indexing your database before importing any KG data is a good practice to speed up the importing process. Nodes will get an indexed property: an ID that is not present in the original data, but unique for each node. This can be done like this:
```
CREATE INDEX node_id IF NOT EXISTS 
FOR (n:Node) ON (n.id);
```
Generate a list of file names for your chunks and generate the Cypher imports:
It is important to start with the importation of nodes and then the relationships, because Neo4j does not allow relationships to be created before nodes exist.
- Nodes
Run this query to generate the imports commands up to `co`for the nodes:
WARNING: change `co` with your max chunk name if using another KG. 
```
WITH range(0,3*26-1) as indices
UNWIND indices as i
WITH 
  ['a','b','c'][i / 26] as first,
  ['a','b','c','d','e','f','g','h','i','j','k','l','m','n','o','p','q','r','s','t','u','v','w','x','y','z'][(i % 26)] as second
WHERE first + second <= 'co'
RETURN "CALL apoc.load.json('file:///nodes_c_chunk_" + first + second + "') YIELD value CREATE (n:Node) SET n = value;" as query
ORDER BY first + second;
```
This query will return something like that:
```
CALL apoc.load.json('file:///nodes_c_chunk_aa') YIELD value CREATE (n:Node) SET n = value;
CALL apoc.load.json('file:///nodes_c_chunk_ab') YIELD value CREATE (n:Node) SET n = value;
CALL apoc.load.json('file:///nodes_c_chunk_ac') YIELD value CREATE (n:Node) SET n = value;
CALL apoc.load.json('file:///nodes_chunk_ad') YIELD value CREATE (n:Node) SET n = value;
CALL apoc.load.json('file:///nodes_c_chunk_ae') YIELD value CREATE (n:Node) SET n = value;
CALL apoc.load.json('file:///nodes_c_chunk_af') YIELD value CREATE (n:Node) SET n = value;
CALL apoc.load.json('file:///nodes_c_chunk_ag') YIELD value CREATE (n:Node) SET n = value;
CALL apoc.load.json('file:///nodes_c_chunk_ah') YIELD value CREATE (n:Node) SET n = value;
CALL apoc.load.json('file:///nodes_c_chunk_ai') YIELD value CREATE (n:Node) SET n = value;
```
Now copy paste it all in a new Cypher query in the Neo4j Browser and run the 67 chunks to import.
WARNING: make sure there is no characters like \" or \' in the copied query as it will cause syntax error.

- Edges
```
Run this query to generate the imports commands up to `co`for the nodes:
WARNING: change `kj` with your max chunk name if using another KG. 
```
WITH range(0,11*26-1) as indices
UNWIND indices as i
WITH 
  ['a','b','c','d','e','f','g','h','i','j','k'][i / 26] as first,
  ['a','b','c','d','e','f','g','h','i','j','k','l','m','n','o','p','q','r','s','t','u','v','w','x','y','z'][(i % 26)] as second
WHERE first + second <= 'kj'
RETURN "CALL apoc.load.json('file:///edges_c_chunk_" + first + second + "') YIELD value 
MATCH (from:Node {id: value.subject}) 
MATCH (to:Node {id: value.object}) 
CALL apoc.create.relationship(from, value.predicate, {json_data: apoc.convert.toJson(value)}, to) YIELD rel 
RETURN count(rel);" as query
ORDER BY first + second;
```

Run the queries and get your list of 337 Cypher commands:
```
CALL apoc.load.json('file:///edges_chunk_aa') YIELD value 
MATCH (from:Node {id: value.subject}) 
MATCH (to:Node {id: value.object}) 
CALL apoc.create.relationship(from, value.predicate, {json_data: apoc.convert.toJson(value)}, to) YIELD rel 
RETURN count(rel);
CALL apoc.load.json('file:///edges_chunk_ab') YIELD value 
MATCH (from:Node {id: value.subject}) 
MATCH (to:Node {id: value.object}) 
CALL apoc.create.relationship(from, value.predicate, {json_data: apoc.convert.toJson(value)}, to) YIELD rel 
RETURN count(rel);
CALL apoc.load.json('file:///edges_chunk_ac') YIELD value 
MATCH (from:Node {id: value.subject}) 
MATCH (to:Node {id: value.object}) 
CALL apoc.create.relationship(from, value.predicate, {json_data: apoc.convert.toJson(value)}, to) YIELD rel 
RETURN count(rel);
CALL apoc.load.json('file:///edges_chunk_ad') YIELD value 
MATCH (from:Node {id: value.subject}) 
MATCH (to:Node {id: value.object}) 
CALL apoc.create.relationship(from, value.predicate, {json_data: apoc.convert.toJson(value)}, to) YIELD rel 
RETURN count(rel);
```


Exploring your KG
------
Run:
```
MATCH (n)-[r]-(m) 
RETURN n,r,m LIMIT 100;
```

Click on graphs to see the visualization of the relationships. You're all set! Start exploring!

Side note: Importing a graph from a cytoscape export
------

Many of you may work with Cytoscape already so I thought of adding a part onto how to load a Cytoscape graph on Neo4j.
After creating an instance and connecting to it, create the import folder at Neo4j root and add your cytoscape .csv files: nodes.csv, edges.csv.

Run the following query (based on Cytoscape export files headers and attributes). Works great on big files:
```
CALL () {
  // Step 1: Load ALL node attributes from nodes.csv
  LOAD CSV WITH HEADERS FROM "file:///nodes.csv" AS nodeRow
  WITH nodeRow
  WHERE NOT nodeRow.name IS NULL
  MERGE (cat:Category {name: nodeRow.name})
  SET cat += nodeRow  // Import ALL columns as properties (except name)
  
  // Step 2: Load relationships from edges.csv  
  LOAD CSV WITH HEADERS FROM "file:///edges.csv" AS edgeRow
  WITH nodeRow, edgeRow  // Keep both row scopes
  WHERE NOT edgeRow.source IS NULL 
    AND NOT edgeRow.target IS NULL 
    AND NOT edgeRow.interaction IS NULL
  MATCH (src:Category {name: edgeRow.source})
  MATCH (tgt:Category {name: edgeRow.target})
  CREATE (src)-[:`edgeRow.interaction`]->(tgt)
} IN TRANSACTIONS OF 1000 ROWS
```