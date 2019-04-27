PerfKitBenchmarker Code Lab
--------------
--------------
## Introduction
PerfKit Benchmarker (PKB) is an open source benchmarking tool that provides a wide array of benchmarking tool to help measure and compare cloud offerings. It automates VM setup and teardown on which ever cloud provider you choose along with installing and running the actual benchmarks. 

In this codelab we will go through installation and basic operation of PerfKit Benchmarker. We will discuss how to run benchmarks, use configuration files, and store and use test results.

### What you'll learn
* How to install and use PerfKit Benchmarker
* How to use configuration files for PerfKit Benchmarker
* How to Store and Query data with PerKit Benchmarker

### What you'll need
* A Google Cloud Platform project
* A web browser, such as Google Chrome
* Basic familiarity with Linux command line

## Installation
These instructions are designed to run in Google Cloud Shell. If not running in Cloud Shell, some additional setup may be needed.
They detail a simple PKB setup to run basic tests on GCP. Additional setup may be required to run benchmarks on other providers and more complex benchmarks may also require additional setup that is specific to that benchmark
More complete instructions for all tasks can be found here

Go to your cloud console (console.cloud.google.com) and click the ‘Activate Cloud Shell’ button on the top right of the screen.

![alt text](images/cloud_shell_icon.png "Cloud Shell Icon")

Cloud Shell provides a small virtual machine that is preloaded with Google Cloud development tools. It also has your PROJECT_ID set to your current project.

In cloud shell, clone the PerfkitBenchmarker Repository into your directory

    git clone https://github.com/SMU-ATT-Center-for-Virtualization/PerfKitBenchmarker.git

Move into the PerfKitBenchmarker directory and install PKB's dependencies

    cd PerfKitBenchmarker
    
    sudo pip install -r requirements.txt

Now we are able to start running benchmarks. Just executing `./pkb.py` will attempt to run a standard set of benchmarks on default machine types in the default region.

To run a specific benchmark, we need to use flags. Lets say we want to run a throughput test (iperf) for n1-standard-2 machines. We would need to use the `--benchmarks` flag and the `--machine_type` flag.

    ./pkb.py --benchmarks=iperf --machine_type=n1-standard-2
    
To run multiple benchmarks, we can give the benchmarks flag a comma separated list.
There are also a variety of flags are specific to each benchmark. To find these flags and further information about a benchmark, we can use the `--helpmatchmd` flag. Run the following command to find flags for the iperf benchmark

    ./pkb.py --helpmatchmd=iperf
    
We can see that there are multiple flags that we can use to customize our iperf benchmark to match our specific needs. For example, in iperf, we can use `--iperf_runtime_in_seconds` to set the amount of time the throughput test runs for or we can use `--iperf_sending_thread_count` to set the number of threads that iperf uses.

Additionally, flags and descriptions for all linux benchmarks can be found [here](https://github.com/SMU-ATT-Center-for-Virtualization/PerfKitBenchmarker/blob/master/testsuite_docs/linux_benchmarks.md)

## Network Tests

For this codelab, we are primarily interested in network benchmarks. PerfKitBenchmarker includes 3 main widely used network benchmarks: ping, iperf, netperf. These network tests are each most useful in different situations.

### ping
Ping is the standard tool to test network latency. It simply measures the round trip time (rtt) of ICMP packets.

### iperf
Iperf is a tool that is used to measure network throughput using TCP or UDP streams. It supports multiple threads streaming data simultaneously. It has a variety of parameters that can be set to test and optimize throughput. 

### netperf
Netperf contains several different test types. We can use TCP_RR to test network latency or TCP_STREAM to test network throughput. Though it doesn't support multiple streaming threads, we can get around this by running multiple instances of netperf in parallel. It also supported UDP latency and throughput tests. With netperf, we can get data at a very fine granularity with its data histograms.

We tend to use a combination of all three benchmark tools when doing our network benchmarks to make sure they agree with each other.

### Running Network Benchmarks
Now that we know what benchmarks are available to us, we can determine what tests would fit our needs. Let's say we want to determine the latency between two machines in zone `us-east1-b` 

    ./pkb.py --benchmarks=ping --zones=us-east1-b

## Config Files

If we want to run benchmarks between two specific zones with specific flags, the easiest way to do so is with benchmark configuration files. The following file runs iperf between a VM in zone `us-central1-b` and a VM in zone `us-east1-b` with 5 sending threads for 30 seconds. As you can see, we can set the cloud, zone, machine type, as well as many other options for each Virtual machine.

sample_config.yml

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


To run this benchmark, we need to specify the config file. If the config file in the PerfKitBenchmarker/perfkitbenchmarker/configs/ directory, we can simply specify the name of the config file. In our case, this config file is already present in that directory, so we can run pkb using that config using the following command:

    ./pkb.py --benchmark_config_file=sample_config.yml --benchmarks=iperf
    
If the config file is in another location, we need to specify the full path of the config file
 
    ./pkb.py --benchmark_config_file=/path/to/config/file.yml --benchmarks=iperf

## Saving and Using Data

By default PKB will output results to the terminal and save them in the directory /tmp/perfkitbenchmarker/runs/

We can also push the results of our benchmarks to BigQuery. To to this we must first create a dataset named `example_dataset` to drop the results into. In your cloud console, enter:

    bq mk example_dataset

When we run pkb, we need to specify the parameters for where we want our data to go in BigQuery. `--bq_project` is the ID of your GCP project and `bigquery_table` takes the dataset name followed by a table name. You don't need to create a table beforehand; it will be created for you by pkb if one of that name does not already exist.

Now let's run a benchmark and push the data to our BigQuery table named `network_tests`

    ./pkb.py --benhmarks=iperf --bigquery_table=example_dataset.network_tests> --bq_project=<project_id>
    
After this has finished, we can now see our data in BigQuery either by going to [BigQuery](https://console.cloud.google.com/bigquery).

Perform a simple query to select all results: `SELECT * from <dataset.table>`

This can also be done using `bq` in cloud shell:

    bq query 'select * from <dataset.table>'
    
## PerfKit Explorer (Data Visualization)

So far we have learned how to run tests and store that data. To make that data more useful to use, we should visualize it in some manner. PerfKit Benchmarker has a sister application called PerfKit Explorer that we can use for this. First we just need to set it up. 

1. In your cloud shell, first let's install the prerequisite packages:
  
       sudo apt-get update

       sudo apt-get install python2.7 openjdk-8-jdk git nodejs nodejs-legacy npm

1. Make sure that the Google App Engine SDK for Python is installed.

       gcloud components install app-engine-python

       gcloud components install app-engine-python-extras
       
1. Clone the repository

       cd ~
       git clone https://github.com/SMU-ATT-Center-for-Virtualization/PerfKitExplorer.git

1. Move into the repository and download the required submodules
       
       cd PerfKitExplorer
       git submodule update --init
       
1. Install required packages
      
       npm install
       sudo npm install -g bower
       bower install

### Compile and Deploy

Next we need to compile and deploy this as an App Engine Application. Make sure you are doing this on a Google CLoud Project where you do not already have an App Engine app running, or this will run instead

1. Modify the config/data_source_config.json so that the production tags are appropriate for the repository you created previously. If not using an analytics key, leave that field blank. For example:

       project_id: <project-id>
       project_name: <project-name>
       samples-mart: <project_id>.example_dataset
       analytics-key: 
       
1. Compile the program

       bash compile.sh
       
1. Move into the newly created `deploy` folder and deploy the application to App Engine

       cd deploy
       gcloud app deploy
       
The app will deploy to http://PROJECT_ID.appspot.com

### Setup the dashboard

1. Open the project URL http://PROJECT_ID.appspot.com in your browser.

1. Click "Edit Config" in the gear icon at the top right and set "default project" to the project id.

1. In Perfkit Dashboard Administration, click "Upload", and select the sample dashboard file: PerfKitExplorer/data/samples_mart/sample_dashboard.json
