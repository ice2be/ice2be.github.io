+++
author = "Matthew Zegar"
title = "Querying the entirety of English Wikipedia in seconds using GPUs and Node.js"
date = "2021-12-16"
description = "Overview of the SQL module I wrote for node RAPIDS"
tags = [
    "node RAPIDS",
    "NVIDIA",
    "node RAPIDS SQL",
    "multi GPU SQL"
]
+++

During the summer and fall term I had a chance to intern on the [RAPIDS](https://rapids.ai/) data visualization team working on an assortment of projects. 
One of the larger projects was hooking up a GPU accelerated SQL engine to Node.js. This enabled us to query extremely large datasets on NVIDIA GPUs 
straight from TypeScript and Node.js. For example, a downloaded copy of English Wikipedia is ~16 GBs. We were able to query this entire dataset from a 
Node process in only ~14 seconds with two NVIDIA Titan RTX GPUs.


## Node RAPIDS

[Node RAPIDS](https://github.com/rapidsai/node) is a collection of RAPIDS library bindings in Node.js. [You can check out this blog post to learn more about it](https://medium.com/rapids-ai/accelerating-node-js-javascript-for-visualization-and-beyond-with-rapids-on-gpus-6906f975e86a).
To summarize, the project aims to provide GPU-accelerated data science to the Node.js developer community (long since deprived of first-class data science libraries) 
by interfacing directly with the RAPIDS C++ libraries, such as libcudf.


## SQL Module

On that foundation, we built the [SQL module in Node RAPIDS](https://github.com/rapidsai/node/tree/main/modules/sql). We took the open source BlazingSQL code, stripped away the Python, 
then interfaced with the C++ layer using TypeScript and Node.js. This allowed us to provide a JavaScript centric interface for the module. 
Here’s a snippet using the [SQLContext](https://rapidsai.github.io/node/classes/sql_src.SQLContext.html) class we built…

```javascript
var { Series, DataFrame, Int32 } = require("@rapidsai/cudf");
var { SQLContext } = require("@rapidsai/sql");

var a  = Series.new({type: new Int32(), data: [1, 2, 3]});
var b  = Series.new({type: new Int32(), data: [4, 5, 6]});
var df = new DataFrame({'a': a, 'b': b});

var sqlContext = new SQLContext();

sqlContext.createDataFrameTable('test_table', df);

await sqlContext.sql('SELECT a FROM test_table').result(); // [1, 2, 3]
```

The preceding code snippet creates a table from a cuDF DataFrame and then queries it on a single NVIDIA GPU. 
But, what if we wanted to distribute the work among **multiple** NVIDIA GPUs?


## Multi-GPU Queries

Using [child_processes](https://nodejs.org/api/child_process.html#child-process) from Node.js we were able to spin up multiple instances and allocate an NVIDIA GPU to each instance. 
We facilitated communication between these instances using BlazingSQL's UCX implementation. This enabled us to query and distribute the workload across multiple NVIDIA GPUs. 
The following snippet uses our multi-GPU [SQLCluster](https://rapidsai.github.io/node/classes/sql_src.SQLCluster.html) class...

```javascript
var { Series, DataFrame } = require("@rapidsai/cudf");
var { SQLCluster } = require("@rapidsai/sql");

var a  = Series.new(['foo', 'bar']);
var df = new DataFrame({'a': a});

var sqlCluster = await SQLCluster.init({numWorkers: 2});
await sqlCluster.createDataFrameTable('test_table', df);


await sqlCluster.sql('SELECT a FROM test_table WHERE a LIKE \'%foo%\'');  // ['foo']
```


## Querying English Wikipedia

To showcase this module in action we needed a large dataset to query. We downloaded the entirety of English Wikipedia and loaded it up into our SQL module. 
[The exact instructions on how we generated this dataset can be found here](https://github.com/rapidsai/node/tree/main/modules/demo/sql/sql-cluster-server#dataset).
This dataset ends up being around ~16 GBs in total size.


## Workarounds

One issue we ran into when importing the dataset was the [maximum DataFrame column size limitation](https://github.com/rapidsai/cudf/issues/3958). cuDF has a limit of two billion elements; 
we were exceeding that limit with the size of our dataset and at times the size of our query result. 
To work around this we created multiple DataFrames and split the data among them instead of storing everything in one single DataFrame.

We ended up being able to query the entirety of English Wikipedia in around ~40s (using two GPUs). This was blazing fast… but we can do better.


## GPUDirect Storage (GDS)

One of the bottlenecks when processing large datasets on GPUs is the I/O. To move data into and out of GPU memory we previously used the bounce buffer through the CPU. 
We implemented support for GDS which skips that process entirely and moves data from storage directly to GPU memory. 
This resulted in our query time decreasing from ~40s to ~14s (65% decrease!).


## Building a Frontend and Results

While this SQL module is accessible in a typical notebook like [nteract](https://nteract.io/) or [jupyter](https://jupyter.org/), 
we created a quick demo that allows for building queries easily down below...

{{< youtube -llIzlx7a-U >}}

{{< youtube rH7Wxn5Yr_A >}}


## Conclusion

Hopefully this demo and blog showcases the performance of GPUs as well as ease of use from Node.js. 
If you are interested in the [SQL module](https://github.com/rapidsai/node/tree/main/modules/sql) feel free to check out the [Node RAPIDS](https://github.com/rapidsai/node) project. 
It was a pleasure to intern at NVIDIA and have the opportunity to build out such complicated projects with talented teammates.
