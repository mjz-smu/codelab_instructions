# PerfKitBenchmarker Code Lab

## Overview

### Performance benchmarking

### Performance benchmarking on public cloud

### Introducing PerfKit Benchmarker

PerfKit Benchmarker (PKB) is an open source framework that provides a wide
selection of benchmarking tools that you can use to measure and compare
cloud providers. PKB automates setup and teardown of resources, including
Virtual Machines (VMs), on whichever cloud provider you choose, along with
installing and running the actual benchmark tests.

### Performance benchmarking on Google Cloud Platform with PKB

Performance benchmarking, with PKB, on Google Cloud Platform (GCP).

### PKB Architecture

[**TODO(truty): insert diagram about stages of performance benchmarking.**]

## What you'll do

*   Install PerfKit Benchmarker
*   Run benchmark tests using PerfKit Benchmarker
*   Inspect and using custom configuration files for tests
*   Persist test result data
*   Query and visualize result data

## Prerequisites

*   Basic familiarity with Linux command line

## Setup

### What you'll need

To complete this lab, you'll need:

*   Access to a standard internet browser (Chrome browser recommended),
    where you can access the GCP Console and the GCP Cloud Shell
*   A GCP project

### Sign in to GCP Console

In your browser, open the [GCP Console](https://console.cloud.google.com).

### Activate the Cloud Shell

From the GCP Console click the Cloud Shell icon on the top right toolbar:

![alt text](images/cloud_shell_icon.png "Cloud Shell Icon")

Then click "Start Cloud Shell".

It should only take a few moments to provision and connect to the environment.

This virtual machine is loaded with all the development tools you'll need.
It offers a persistent 5GB home directory, and runs on the Google Cloud,
greatly enhancing network performance and authentication. Much, if not all,
of your work in this lab can be done with simply a browser or your
Google Chromebook.

Once connected to the Cloud Shell, you should see that you are already
authenticated and that the project is already set to your PROJECT_ID.

Run the following command in the Cloud Shell to confirm that you are
authenticated:

```
gcloud auth list
```

**Command output**

```
Credentialed accounts:
 - <myaccount>@<mydomain>.com (active)
```

**Note:** `gcloud` is the powerful and unified command-line tool for
Google Cloud Platform. Full documentation is available
from https://cloud.google.com/sdk/gcloud. It comes pre-installed on
Cloud Shell. You will notice its support for tab-completion.

In Cloud Shell, Verify your project is set up as expected.

```
gcloud config list project
```

**Command output**

```
[core]
project = <PROJECT_ID>
```

If it is not, you can set it with this command:

```
gcloud config set project <PROJECT_ID>
```

**Command output**

```
Updated property [core/project].
```

## Installing PKB using git

**Note**: in this lab, you use the
[SMU AT&T Center for Virtualization](https://www.smu.edu/Provost/virtualization)
fork of the base
[PKB repo](https://github.com/GoogleCloudPlatform/PerfKitBenchmarker). This
[fork](https://github.com/SMU-ATT-Center-for-Virtualization/PerfKitBenchmarker.git)
has a few extra features which will be merged back to the main repo soon.

#### Step 1

In Cloud Shell, `clone` the PerfKitBenchmarker repository.

```
git clone https://github.com/SMU-ATT-Center-for-Virtualization/PerfKitBenchmarker.git
```

#### Step 2

Change to the PerfKitBenchmarker directory.

```
cd PerfKitBenchmarker
```

#### Step 3

Install dependencies of PKB

```
sudo pip install -r requirements.txt
```

**Note**: As part of this exercise, you will will run a few basic tests on GCP
within a simple PKB environment. Additional setup may be required to run
benchmarks on other providers, or to run more complex benchmarks. Comprehensive
instructions for running other benchmarks can be located by reviewing the
[README in the PKB repo](https://github.com/GoogleCloudPlatform/PerfKitBenchmarker).

## Running PKB using command-line flags

You can run benchmarks right now. If you execute `./pkb.py`, with no
command-line flags, PKB will attempt to run a standard set of benchmarks
on default machine types in the default region. You can read more about the
**standard_set** later in this lab.

Instead, it is more common to choose specific benchmarks and options, such
as project, cloud provider, zone, and machine_type, using
**command-line flags**.

### The project, cloud provider, zones, and machine_type flags

You should understand how the `--cloud` provider, `--project`,  `--zones`,
and `--machine_type` flags work.

*   `--cloud`: As **GCP** is the default cloud provider for PKB, the
    `--cloud` flag has a default value of **GCP**.
*   `--project`: PKB needs to have a GCP **project-id** to run. In Cloud
    Shell, PKB inherits the `--project` **project-id** from the environment.
*   `--zone`: Every cloud provider has a default zone. For GCP, the `--zone`
    flag defaults to
    [**us-central1-a**](https://github.com/GoogleCloudPlatform/PerfKitBenchmarker/blob/master/perfkitbenchmarker/configs/default_config_constants.yaml).
*   `--machine_type`: Benchmarks are frequently tightly coupled to
    specific machine capabilities, especially CPU and memory. You can pick
    your specific machines with the `--machine_type` flag. Most benchmark
    tests, including the common networking benchmarks **ping**, **iperf**,
    and **netperf**, default to the provider-specific
    `default_single_core` machine. On GCP, the default machine is the [**n1-standard-1**](https://github.com/GoogleCloudPlatform/PerfKitBenchmarker/blob/master/perfkitbenchmarker/configs/default_config_constants.yaml.)

You can learn more about alternative flag values in the
[Useful Global Flags](https://github.com/GoogleCloudPlatform/PerfKitBenchmarker#useful-global-flags)
section of the PKB readme.

### Run one benchmark test with the benchmark flag

The `--benchmark` flag is used to select the benchmark(s) run. Not supplying
`--benchmark` is the same as using `--benchmarks="standard_set"`.
The **standard_set** is a collection of commonly used benchmarks. You can read
more about benchmark sets later in this lab.

Cloud benchmark tests commonly need at least 10minutes to complete because of
the many resources, including networks, firewall rules, and VMs, that must
be both provisioned and de-provisioned.

Start a benchmark test now, and then continue working through the lab while
the test executes.

Run the commonly used network throughput test, **iperf**, with a small
machine, **n1-standard-1**.

**Expected duration**: ~13-14min.

```
./pkb.py --benchmarks=iperf
```

**Expected output**:

```
-------------------------PerfKitBenchmarker Results Summary-------------------------
IPERF:
  receiving_machine_type="n1-standard-1" receiving_zone="us-central1-a" run_number="0" runtime_in_seconds="60" sending_machine_type="n1-standard-1" sending_thread_count="1" sending_zone="us-central1-a"
  Throughput                         1966.000000 Mbits/sec                      (ip_type="external")
  Throughput                         1968.000000 Mbits/sec                      (ip_type="internal")
  Throughput                         1962.000000 Mbits/sec                      (ip_type="external")
  Throughput                         1968.000000 Mbits/sec                      (ip_type="internal")
  End to End Runtime                  787.150529 seconds
...
------------------------------------------
Name   UID     Status     Failed Substatus
------------------------------------------
iperf  iperf0  SUCCEEDED
------------------------------------------
Success rate: 100.00% (1/1)
```

## Discover helfpul flags and notes about benchmark tests

While **iperf** is running, explore PKB benchmarks and flags.

#### Step 1

Open a second Cloud Shell in GCP Console by clicking the **Add Cloud Shell
Session** button on top of the existing Cloud Shell.

#### Step 2

Change to the PerfKitBenchmarker directory.

```
cd PerfKitBenchmarker
```

#### Step 3

Base PKB includes the `--helpmatch` flag which can be used to discover details
about benchmarks and related configuration flags. You can pass
`--helpmatch` a regex and it will print related help text.

Review all the global flags for PKB.

```
./pkb.py --helpmatch=pkb
```

#### Step 4

As used earlier, the `--benchmarks` selects a specific benchmark or
benchmark set.

Review the full list of benchmarks available.

```
./pkb.py --helpmatch=benchmarks | grep perfkitbenchmarker
```

You should see around 70 different benchmarks available to run, within
linux_benchmarks collection.

PKB has a naming convention for benchmarks of
**[NAME]_benchmark**. For example, notice: **ping_benchmark**,
**iperf_benchmark**, and **netperf_benchmark**.0

#### Step 3

When you want to review the details and flags of a benchmark in depth, it can be
easier to read formatted MarkDown.

One advantage of our **fork** of PKB is a new `--helpmatchmd` flag. The
`--helpmatchmd` flag emits more easily readable MarkDown text than
`--helpmatch`.

The MarkDown from using `--helpmatchmd` has also been checked back into our
fork in the [testsuite_docs folder](https://github.com/SMU-ATT-Center-for-Virtualization/PerfKitBenchmarker/tree/master/testsuite_docs).

Review the available Linux benchmarks from the
[linux_benchmarks file](https://github.com/SMU-ATT-Center-for-Virtualization/PerfKitBenchmarker/blob/master/testsuite_docs/linux_benchmarks.md).

Try searching on **ping_benchmark**, **iperf_benchmark**, and
**netperf_benchmark**.

#### Step 4

Each benchmark can have custom flags too. Review the flags for the
**netperf** and **iperf** benchmarks.

```
./pkb.py --helpmatchmd=netperf

./pkb.py --helpmatchmd=iperf
```

You can see that there are multiple flags to customize benchmark runs.

## Considering Different Network Test Tools

PerfKitBenchmarker includes 3 widely used **networking** benchmarks:
**ping**, **iperf**, and **netperf**. Each of these network tests can be useful
in different situations.

### ping

The **ping** command is the most widely distributed and is commonly used to
verify connectivity and measure simple network latency. It measures the
round trip time (rtt) of ICMP packets.

### iperf

The **iperf** tool is easy to use and is used to measure network throughput
using TCP or UDP streams. It supports multiple threads streaming data
simultaneously. It has a variety of parameters that can be set to test and
maximize throughput.

**Note**: as of 2019, **iperf** is the name of the benchmark, but
**iperf v2** is used.

### netperf

The **netperf** tool contains several different test types. You can use
**TCP_RR**, TCP request-response, to test network latency. You can run
**TCP_STREAM** to test network throughput.

You can run multiple instances of netperf in parallel to heavily stress links
via multiple processors. The netperf tool also supports running UDP latency
and throughput tests.

With netperf, you can also see alternative reporting flavors with its data
histograms.

In many cases, it is recommended to run combinations of all three networking
benchmark tools and use added tests for result confirmation.

## Explore the results of a benchmark

Return to the first Cloud Shell to review the test results from **iperf**.

Detailed output from the benchmark is printed to the screen, and saved to
log files under `/tmp/perfkitbenchmarker/runs/`.

Whether you scroll back in the Cloud Shell, or look through the pkb.log file,
you can review many details of the benchmark pass:

*   PKB details: version# and flags used.
*   Resources being provisioned: an auto-mode VPC network, two firewall rules,
    one for internal IPs and another for external IPs, two VM instances, and
    attached persistent-disks.
*   Software setup: Setup directories on both VMs and install python, iperf,
    and other packages.
*   System configuration: adjust kernel settings,
    including **tcp_congestion_control**.
*   Test execution:
    *   VM1->VM2 throughput test over external IPs
    *   VM1->VM2 throughput test over internal IPs
    *   VM2->VM1 throughput test over external IPs
    *   VM2->VM1 throughput test over internal IPs
*   Resources being cleaned up.
*   Detailed result data: includes details about the resources along with
    metrics that include timestamp, units, and values.
*   Results Summary: an easy-to-read table with the key metrics and values.
*   Overall test status, especially useful when multiple benchmarks have run.

## Running Network Benchmarks

### Measure latency with ping

Run a test to determine the latency between two machines in a single
zone: `us-east1-b`.

**Expected duration**: ~11-12min.

```
./pkb.py --benchmarks=ping --machine_type=f1-micro
./pkb.py --benchmarks=ping --zones=us-east1-b
```

**Expected output**:

```
-------------------------PerfKitBenchmarker Results Summary-------------------------
PING:
  ip_type="internal" receiving_zone="us-central1-a" run_number="0" sending_zone="us-central1-a"
  Min Latency                           0.240000 ms
  Average Latency                       0.307000 ms
  Max Latency                           0.605000 ms
  Latency Std Dev                       0.059000 ms
  Min Latency                           0.171000 ms
  Average Latency                       0.703000 ms
  Max Latency                          35.503000 ms
  Latency Std Dev                       3.531000 ms
  End to End Runtime                  691.010288 seconds
...
----------------------------------------
Name  UID    Status     Failed Substatus
----------------------------------------
ping  ping0  SUCCEEDED
----------------------------------------
Success rate: 100.00% (1/1)
...
```

### Create Config Files for More Complex Tests

To run networking benchmarks between two specific zones with specific flags, the
easiest way is with **benchmark configuration files**. For example, the
following file runs **iperf** between a VM in zone `us-central1-b` and a VM
in zone `us-east1-b`, with 5 sending threads for 30 seconds.

You can set the cloud, zone, machine type, as well as many other options for
each VM in the config file.

**sample_config.yml**

```
iperf:
  vm_groups:
    vm_1:
      cloud: GCP
      vm_spec:
        GCP:
          machine_type: n1-standard-2
          zone: us-central1-b
    vm_2:
      cloud: GCP
      vm_spec:
        GCP:
          machine_type: n1-standard-2
          zone: us-east1-b
flags:
  iperf_sending_thread_count: 5
  iperf_runtime_in_seconds: 30
```

Run this benchmark using the config file.

**Expected duration**: xxmin.

```
./pkb.py --benchmark_config_file=sample_config.yml --benchmarks=iperf
```

**Expected output**: xxx.

By default, config files must reside under the
`PerfKitBenchmarker/perfkitbenchmarker/configs/` directory. Our fork has a
`sample_config.yml` file in the configs directory.

You can also specify the full path of the config file:

```
./pkb.py --benchmark_config_file=/path/to/config/file.yml --benchmarks=iperf
```

## Benchmark Sets

Other sets
are curated and available to run as well. You can run multiple benchmarks
by using a comma separated list.



## Saving and Using Data with BigQuery

By default PKB will output results to the terminal and save logs to the
directory `/tmp/perfkitbenchmarker/runs/`.

If you plan to run many tests, a recommended practice is to push your result
data to [BigQuery](https://cloud.google.com/bigquery/), a serverless,
highly-scalable, cost-effective data warehouse. You can then use BigQuery to
review your test results over time, and create data visualizations.

### Create a dataset

To do this, create a **dataset** where result tables and views can be created,
secured and shared. You can create datasets either using the GCP Console or
using the BigQuery command-line tool `bq` in Cloud Shell.

```
bq mk example_dataset
```

### Run PKB with BigQuery arguments

Then, when you run PKB, you supply BigQuery-specific arguments to send your
result data directly to BigQuery tables.

*   `--bq_project`: your GCP Project-ID that owns the dataset and tables.
*   `bigquery_table`: a fully qualified table name, including the dataset.

The first time you run experiments, PKB will create the table if it does not
yet exist.

Run an iperf benchmark test and push the test log data to a BigQuery table
named `network_tests`.

**Expected duration**: xxmin.

```
    ./pkb.py --benhmarks=iperf \
        --bq_project=<project_id> \
        --bigquery_table=example_dataset.network_tests>
```

**Expected output**: xxx.

You can then see your data using the
[BigQuery UI](https://console.cloud.google.com/bigquery).

Use the **Query View** to run a simple query that shows your results.

```
SELECT * FROM <dataset.table> LIMIT 200;
```

You can also use the command-line `bq` tool, again, in Cloud Shell.

```
bq query 'SELECT * FROM <dataset.table> LIMIT 200'
```

## Visualizing Data with PerfKit Explorer

So far we have learned how to run tests and store that data. To make that
data more useful to use, we should visualize it in some manner.
PerfKit Benchmarker has a sister application called PerfKit Explorer that we
can use for this. First we just need to set it up.

1. In your cloud shell, first let's install the prerequisite packages:

```
sudo apt-get update
sudo apt-get install python2.7 openjdk-8-jdk git nodejs nodejs-legacy npm
```

1. Make sure that the Google App Engine SDK for Python is installed.

```
gcloud components install app-engine-python
gcloud components install app-engine-python-extras
```

1. Clone the repository

```
cd ~
git clone https://github.com/SMU-ATT-Center-for-Virtualization/PerfKitExplorer.git
```

1. Move into the repository and download the required submodules

```
cd PerfKitExplorer
git submodule update --init
```

1. Install required packages

```
npm install
sudo npm install -g bower
bower install
```

### Compile and Deploy

Next we need to compile and deploy this as an App Engine Application. Make
sure you are doing this on a Google CLoud Project where you do not already
have an App Engine app running, or this will run instead

1. Modify the config/data_source_config.json so that the production tags
are appropriate for the repository you created previously. If not using an
analytics key, leave that field blank. For example:

```
project_id: <project-id>
project_name: <project-name>
samples-mart: <project_id>.example_dataset
analytics-key:
```

1. Compile the program

```
bash compile.sh
```

1. Move into the newly created `deploy` folder and deploy the application to
App Engine

```
cd deploy
gcloud app deploy
```

The app will deploy to http://PROJECT_ID.appspot.com

### Set up the dashboard

1. Open the project URL http://PROJECT_ID.appspot.com in your browser.

1. Click "Edit Config" in the gear icon at the top right and set
"default project" to the project id.

1. In Perfkit Dashboard Administration, click "Upload", and select the
sample dashboard file: PerfKitExplorer/data/samples_mart/sample_dashboard.json

## Congratulations!

You learned xxx

### Next Steps / Learn More

*   Watch the Perfkit Benchmarker video from Google Cloud Next '19.
