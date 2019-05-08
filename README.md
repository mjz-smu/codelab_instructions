# PerfKitBenchmarker Code Lab

## Overview

PerfKit Benchmarker (PKB) is an open source framework that provides a wide
selection of benchmarking tools that you can use to measure and compare
cloud providers. PKB automates setup and teardown of resources, including
Virtual Machines (VMs), on whichever cloud provider you choose, along with
installing and running the actual benchmark tests.

### What you'll do

*   How to install and use PerfKit Benchmarker to run benchmarks
*   How to inspect and use configuration files for different tests
*   How to store test result data and then query it

### Prerequisites

* Basic familiarity with Linux command line

## Setup

### What you'll need

To complete this lab, you'll need:

*   Access to a standard internet browser (Chrome browser recommended).
*   A Google Cloud Platform (GCP) project.

### Sign in to Google Cloud Console

In your browser, open the [GCP Console](https://console.cloud.google.com).

### Activate Google Cloud Shell

From the GCP Console click the Cloud Shell icon on the top right toolbar:

![alt text](images/cloud_shell_icon.png "Cloud Shell Icon")

Then click "Start Cloud Shell".

It should only take a few moments to provision and connect to the environment.

This virtual machine is loaded with all the development tools you'll need.
It offers a persistent 5GB home directory, and runs on the Google Cloud,
greatly enhancing network performance and authentication. Much, if not all,
of your work in this lab can be done with simply a browser or your
Google Chromebook.

Once connected to the cloud shell, you should see that you are already
authenticated and that the project is already set to your PROJECT_ID.

Run the following command in the cloud shell to confirm that you are
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

## Install PKB using git

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
instructions for running other benchmarks can be located within the
[PKB repo](https://github.com/GoogleCloudPlatform/PerfKitBenchmarker).

## Running PKB and using command-line flags

You can run benchmarks right now. Executing `./pkb.py` would attempt to run a
standard set of benchmarks on default machine types in the default region.

To run a specific benchmark, you need to use **flags**.

### Setting cloud provider and zones

You should understand how `--cloud` provider and `--zones` flags work.
GCP is the default provider for PKB, and the `--cloud` flag has a default of
**GCP**. Each cloud has a default zone, and, for GCP, the `--zone` flag defaults
to **us-central1-a**. You can learn more about alternative cloud providers and
zones in the [Useful Global
Flags](https://github.com/GoogleCloudPlatform/PerfKitBenchmarker#useful-global-flags)
section of the PKB readme.

### Discovering helfpul details about benchmark tests

Base PKB includes the `--helpmatch` flag which can be used to discover details
about benchmarks and related configuration flags. You can pass
`--helpmatch` a regex and it will print related help text.

#### Step 1

Review all the global flags for PKB.

```
./pkb.py --helpmatch=pkb
```

#### Step 2

The `--benchmarks` flag is used to select a specific benchmark or
benchmark set.

Review the full list of benchmarks available.

```
./pkb.py --helpmatch=benchmarks | grep perfkitbenchmarker
```

You should see around 70 different linux_benchmarks available to run.

#### Step 3

If you want to review the details of the different benchmarks, it can be
easier to read formatted MarkDown.

One advantage of our **fork** of PKB is a new `--helpmatchmd` flag. The
`--helpmatchmd` flag emits easily readable MarkDown text than `--helpmatch`.
The MarkDown from using `--helpmatchmd` has also been checked back into our
fork [here](https://github.com/SMU-ATT-Center-for-Virtualization/PerfKitBenchmarker/tree/master/testsuite_docs).

Review the available Linux benchmarks from the MarkDown
[here](https://github.com/SMU-ATT-Center-for-Virtualization/PerfKitBenchmarker/blob/master/testsuite_docs/linux_benchmarks.md).

### Run one benchmark

The `--benchmark` flag is used to select the benchmark(s) run. Not supplying
`--benchmark` is the same as using `--benchmarks="standard_set"`.
The **standard_set** is a collection of commonly used benchmarks. Other sets
are curated and available to run as well. You can run multiple benchmarks
by using a comma separated list.

The `--machine_type` flag is used to select the cloud-specific machine.
Alternative ways to configure machine type can be researched in the
Useful Global Flags section of the
[readme](https://github.com/GoogleCloudPlatform/PerfKitBenchmarker#useful-global-flags).

#### Step 1

Run a common network throughput test, **iperf**, with a small
machine, **n1-standard-2**.

```
./pkb.py --benchmarks=iperf --machine_type=n1-standard-2
```

#### Step 2

Each benchmark can have custom flags too. Review the flags for the
**netperf** and **iperf** benchmarks.

```
./pkb.py --helpmatchmd=netperf
./pkb.py --helpmatchmd=iperf
```

You can see that there are multiple flags to customize **iperf** benchmark runs.
For example, in iperf, `--iperf_runtime_in_seconds` is used to set the amount
of time the test runs. Also, you can use `--iperf_sending_thread_count` to set
the number of threads that iperf uses.

Again, flags and descriptions for all Linux benchmarks can be found
[here](https://github.com/SMU-ATT-Center-for-Virtualization/PerfKitBenchmarker/blob/master/testsuite_docs/linux_benchmarks.md).

## Considering Different Network Test Tools

PerfKitBenchmarker includes 3 widely used **networking** benchmarks:
**ping**, **iperf**, and **netperf**. Each of these network tests can be useful
in different situations.

### ping

The **ping** command is the most widely distributed and is commonly used to
verify connectivity and measure network latency. It measures the round trip
time (rtt) of ICMP packets.

### iperf

The **iperf** tool is easy to use and is used to measure network throughput using
TCP or UDP streams. It supports multiple threads streaming data simultaneously.
It has a variety of parameters that can be set to test and maximize throughput.

**Note**: iperf 2 is preferred and used in PKB.

### netperf

The **netperf** tool contains several different test types. You can use
**TCP_RR** to test network latency (using TCP requests and responses), or
**TCP_STREAM** to test network throughput. You can run multiple instances of
netperf in parallel to heavily utilize multiple processors. It also
supports running UDP latency and throughput tests. With netperf, you can also
get some advanced reporting with its data histograms.

In many cases, it is recommended to run combinations of all three networking
benchmark tools. The results can then be used to confirm results.

## Running Network Benchmarks

### Measuring latency

Run a test to determine the latency between two machines in a single
zone: `us-east1-b`.

```
./pkb.py --benchmarks=ping --zones=us-east1-b
```

### Using Config Files for More Complex Tests

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

To run this benchmark, supply the config file.

```
./pkb.py --benchmark_config_file=sample_config.yml --benchmarks=iperf
```

By default, config files must reside under the
`PerfKitBenchmarker/perfkitbenchmarker/configs/` directory. Our fork has a
`sample_config.yml` file in the configs directory.

You can also specify the full path of the config file:

```
./pkb.py --benchmark_config_file=/path/to/config/file.yml --benchmarks=iperf
```

## Saving and Using Data

By default PKB will output results to the terminal and save them in the
directory `/tmp/perfkitbenchmarker/runs/`.

You can also push the results of your benchmark runs to
[BigQuery](https://cloud.google.com/bigquery/), a serverless, highly-scalable,
cost-effective data warehouse.

To to this you must first create a dataset named `example_dataset` to drop
the results into. In your cloud console, enter:

```
bq mk example_dataset
```

When you run PKB, we need to specify the parameters for where we want our data
to go in BigQuery. `--bq_project` is the ID of your GCP project and
`bigquery_table` takes the dataset name followed by a table name. You don't
need to create a table beforehand; it will be created for you by pkb if one of
that name does not already exist.

Now let's run a benchmark and push the data to our BigQuery table named
`network_tests`

```
    ./pkb.py --benhmarks=iperf --bigquery_table=example_dataset.network_tests --bq_project=<project_id>
```

After this has finished, we can now see our data in BigQuery either by going
to [BigQuery](https://console.cloud.google.com/bigquery).

Perform a simple query to select all results:

```
SELECT * from <dataset.table>
```

This can also be done using `bq` in cloud shell:

```
bq query 'select * from <dataset.table>'
```

## PerfKit Explorer (Data Visualization)

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

### Loading Samples and Setting up the dashboard

A dashboard isn't much use without any data to show, so for this codelab, we have provided sample data and a sample dashboard. 

1. Make sure you are in the PerfKitExplorer Directory

1. To load the sample data into BigQuery, you'll need to specify the dataset (using the one created above is fine) and table name. The `--autodetect` flag is used to figure out the schema, so the table doesn't need to exist before running the command:
```
bq --location=US load --autodetect --source_format=NEWLINE_DELIMITED_JSON <dataset>.<table> data/codelab/bq_pkb_sample.json
```

1. The sample dashboard is stored in a json file. Before uploading the sample dashboard, replace all the BigQuery references with the dataset and table name you used to load the samples (you can edit these references in the dashboard too but this is way quicker):

```
sed -i 's/my_project.my_dataset.my_table/<actual_project>.<actual_dataset>.<actual_table>/g' data/codelab/codelab_perfkit_dashboard.json
```

1. Open the project URL http://PROJECT_ID.appspot.com in your browser.

1. Click "Edit Config" in the gear icon at the top right and set
"default project" to the project id.

1. In Perfkit Dashboard Administration, click "Upload", and select the
sample dashboard file: PerfKitExplorer/data/codelab/codelab_perfkit_dashboard.json


## Congratulations!

You learned xxx

### Next Steps / Learn More

*   Watch the Perfkit Benchmarker video from Google Cloud Next '19.
