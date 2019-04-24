PerfKitBenchmarker Code Lab
--------------
--------------

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

#TODO add stuff about config files

##Network Tests

For this codelab, we are primarily interested in network benchmarks. PerfKitBenchmarker includes 3 main widely used network benchmarks: ping, iperf, netperf.

###ping

###iperf

###netperf



By default PKB will output results to the terminal and save them in the directory /tmp/perfkitbenchmarker/runs/

We can also push the results of our benchmarks to BigQuery. To do this we must add the following flags to our pkb command

    ./pkb.py --benhmarks=iperf --bigquery_table=<dataset.table> --bq_project=<project_id>
