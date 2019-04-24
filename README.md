PerfKitBenchmarker Code Lab
--------------
--------------

## Installation
These instructions are for Google Cloud Console
They detail a simple PKB setup to run basic tests on GCP. Additional setup may be required to run benchmarks on other providers and more complex benchmarks may also require additional setup that is specific to that benchmark
More complete instructions for all tasks can be found here

Go to your cloud console (console.cloud.google.com) and click the ‘Activate Cloud Shell’ button on the top right of the screen.

Clone or download the PerfkitBenchmarker Repo

    git clone https://github.com/SMU-ATT-Center-for-Virtualization/PerfKitBenchmarker.git

Install PKB dependencies

    cd /path/to/PerfKitBenchmarker
    
    sudo pip install -r requirements.txt

Just running `./pkb.py` will attempt to run a standard set of benchmarks on default machine types in the default region.

To run a specific benchmark, we need to use flags. Lets say we want to run a throughput test (iperf) for n1-standard-2 machines. We would need to use the `--benchmarks` flag and the `--machine_type` flag.

    ./pkb.py --benchmarks=iperf --machine_type=n1-standard-2
    
To run multiple benchmarks, we can give the benchmarks flag a comma separated list.
There are also a variety of flags are specific to each benchmark. To find these flags and further information about a benchmark, we can use the `--helpmatchmd` flag. Run the following command to find flags for the iperf benchmark

    ./pkb.py --helpmatchmd=iperf
    
We can see that there are multiple flags that we can use to customize our iperf benchmark to match our specific needs. For example, in iperf, we can use `--iperf_runtime_in_seconds` to set the amount of time the throughput test runs for or we can use `--iperf_sending_thread_count` to set the number of threads that iperf uses.

## Network Tests

For this codelab, we are primarily interested in network benchmarks. PerfKitBenchmarker includes 3 main widely used network benchmarks: ping, iperf, netperf.

### ping
Ping is the standard tool to test network latency. It simply measures the round trip time (rtt) of ICMP packets.

### iperf
Iperf is a tool that is used to measure network throughput using TCP or UDP streams. It supports multiple threads streaming data simultaneously. It has a variety of parameters that can be set to test and optimize throughput. 

### netperf
Netperf contains several different test types. We can use TCP_RR to test network latency or TCP_STREAM to test network throughput. Though it doesn't support multiple streaming threads, we can get around this by running multiple instances of netperf in parallel. It also supported UDP latency and throughput tests. With netperf, we can get data at a very fine granularity with its histograms.

We tend to use a combination of all three benchmark tools when doing our network benchmarks to make sure they agree with each other.

## Config Files

If we want to run benchmarks between two specific zones with specific flags, the easiest way to do so is with benchmark configuration files. The following file runs iperf between a VM in zone us-central1-b and a VM in zone europe-west1-d.

    iperf:
      vm_groups:
        vm_1:
          cloud: GCP
          vm_spec:
            GCP: 
              zone: us-central1-b
        vm_2:
          cloud: GCP
          vm_spec:
            GCP: 
              zone: europe-west1-d

To run this benchmark, we need to specify the config file. If we place the config file in the PerfKitBenchmarker/perfkitbenchmarker/configs/ directory, we can simply specify the name of the config file

    ./pkb.py --benchmark_config_file=<config_file.yml> --benchmarks=iperf
    
If the config file is in another location, we need to specify the full path of the config file
 
    ./pkb.py --benchmark_config_file=/path/to/config/file.yml --benchmarks=iperf


By default PKB will output results to the terminal and save them in the directory /tmp/perfkitbenchmarker/runs/

We can also push the results of our benchmarks to BigQuery. To do this we must add the following flags to our pkb command

    ./pkb.py --benhmarks=iperf --bigquery_table=<dataset.table> --bq_project=<project_id>
