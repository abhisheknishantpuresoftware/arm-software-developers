---
processors: ["Neoverse-N1"]
software: ["linux"]
title: "Benchmark MongoDB on Arm with Yahoo Cloud Serving Benchmark (YCSB)"
linkTitle: "Run YCSB to benchmark MongoDB performance on Arm"
type: docs
weight: 3
hide_summary: true
description: >
    This article shows you how to run the Yahoo Cloud Serving Benchmark on MongoDB to measure performance, such as latency and throughput.
---

## Pre-requisites

* MongoDB installed and running on your Arm-based cloud service. Follow the steps outlined [here](/cloud/mongodb/install).
* Java 1.8 and Python 2.7 (or later) installed on your instance.
{{< tabpane code=true >}}
  {{< tab header="Ubuntu" >}}
sudo apt-get update
sudo apt install default-jre python
{{< /tab >}}
{{< tab header="RedHat" >}}
sudo yum check-update
sudo yum install java-1.8.0-openjdk python
{{< /tab >}}
{{< /tabpane >}}
* [Maven 3.x](https://maven.apache.org/index.html) installed on your instance. Run the following commands to install Maven:
```console
   wget http://ftp.heanet.ie/mirrors/www.apache.org/dist/maven/maven-3/3.1.1/binaries/apache-maven-3.1.1-bin.tar.gz
   sudo tar xzf apache-maven-*-bin.tar.gz -C /usr/local
   cd /usr/local
   sudo ln -s apache-maven-* maven
   cd ~/
   export M2_HOME=/usr/local/maven
   export PATH=${M2_HOME}/bin:${PATH}
```
## Detailed Steps

To measure the performance of MongoDB running on Arm, we will run the [Yahoo Cloud Serving Benchmark](http://github.com/brianfrankcooper/YCSB).

[YCSB](https://research.yahoo.com/news/yahoo-cloud-serving-benchmark/) is an open sourced project which provides the framework and common set of workloads to evaluate the performance of different "key-value" and "cloud" serving stores. Here we will list out the steps to run YCSB to evalute the performance of MongoDB running on a 64-bit Arm machine.

### Setup YCSB 

On your instance that is running MongoDB (you may need to start a new terminal), download the latest released YCSB zip file and uncompress

```console
mkdir ~/ycsb && cd ~/ycsb
curl -O --location https://github.com/brianfrankcooper/YCSB/releases/download/0.17.0/ycsb-0.17.0.tar.gz
tar xfvz ycsb-0.17.0.tar.gz
```
Now `cd` into project folder and run the executable to print a description of how to use the benchmark
```console
cd ycsb-0.17.0
bin/ycsb --help
```

### Load/Insert Test on MongoDB

To test the performance of loading data(`INSERT`) into default database `ycsb` at `localhost:27017` where MongoDB is running using the `synchronous` driver run the following command:
```console
./bin/ycsb load mongodb -s -P workloads/workloada -p mongodb.url=mongodb://localhost:27017/ycsb?w=0 -threads 10
```
The `-P` (upper case) parameter is used to load property files. In this example, we used it load the `workloada` parameter file, which sets the recordcount to 1000 in addition to other parameters. Specific properties can be overridden with `-p` (lower case).\
The `-threads` parameter indicates the number of threads and is set to 1 by default.

To load/insert data using the `asynchronous` driver instead run the following command (changing `mongodb` to `mongodb-async`):
```console
./bin/ycsb load mongodb-async -s -P workloads/workloada -p mongodb.url=mongodb://localhost:27017/ycsb?w=0 -threads 10
```

### Update/Read/Read Modify Write Test on MongoDB

To test the performance of executing a workload which includes running `UPDATE`, Read Modify Write(`RMW`) and/or `READ` operations on the data using 10 threads, use the following example command:

```console
./bin/ycsb load mongodb -s -P workloads/workloada -p mongodb.url=mongodb://localhost:27017/ycsb?w=0
```
The `workloads/workloada` file in this example sets the following values `readproportion=0.5` and  `updateproportion=0.5` which means there is an even split between the number of `READ` and `UPDATE` operations performed. You can change the type of operations and the splits by providing your own workload parameter file.

To run the same command using the asynchronous driver change `mongodb` in the above to `mongodb-async`. 

For more detailed information on all the parameters for running a workload refer to [this section](https://github.com/brianfrankcooper/YCSB/wiki/Running-a-Workload) of the original repository.

### View the results

At the end of each test, statistics are printed to the console. Shown below is an example output from the end of `Load/Insert` test:
```
2022-07-06 15:50:18:917 1 sec: 1000 operations; 542.01 current ops/sec; [CLEANUP: Count=10, Max=12951, Min=0, Avg=1295.2, 90=4, 99=12951, 99.9=12951, 99.99=12951] [INSERT: Count=1000, Max=134655, Min=561, Avg=8506.37, 90=10287, 99=39903, 99.9=134015, 99.99=134655]
[OVERALL], RunTime(ms), 1849
[OVERALL], Throughput(ops/sec), 540.8328826392644
[TOTAL_GCS_Copy], Count, 5
[TOTAL_GC_TIME_Copy], Time(ms), 23
[TOTAL_GC_TIME_%_Copy], Time(%), 1.2439156300703083
[TOTAL_GCS_MarkSweepCompact], Count, 0
[TOTAL_GC_TIME_MarkSweepCompact], Time(ms), 0
[TOTAL_GC_TIME_%_MarkSweepCompact], Time(%), 0.0
[TOTAL_GCs], Count, 5
[TOTAL_GC_TIME], Time(ms), 23
[TOTAL_GC_TIME_%], Time(%), 1.2439156300703083
[CLEANUP], Operations, 10
[CLEANUP], AverageLatency(us), 1295.2
[CLEANUP], MinLatency(us), 0
[CLEANUP], MaxLatency(us), 12951
[CLEANUP], 95thPercentileLatency(us), 12951
[CLEANUP], 99thPercentileLatency(us), 12951
[INSERT], Operations, 1000
[INSERT], AverageLatency(us), 8506.367
[INSERT], MinLatency(us), 561
[INSERT], MaxLatency(us), 134655
[INSERT], 95thPercentileLatency(us), 11871
[INSERT], 99thPercentileLatency(us), 39903
[INSERT], Return=OK, 1000
```

### Further Reading

For instructions on running any other tests or more details on the metrics reported, refer to the [github project for the YCSB](https://github.com/brianfrankcooper/YCSB/wiki/).

To view performance results of MongoDB on AWS Graviton2-based Amazon EC2 R6g instance and how it compares to Intel Xeon-based R5 instances read the [blog here](https://community.arm.com/arm-community-blogs/b/operating-systems-blog/posts/mongodb-performance-on-aws-with-the-arm-graviton2).

[<-- Return to Learning Path](/cloud/mongodb/#sections)
