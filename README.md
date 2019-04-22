# codelab_instructions

PKB Quickstart
--------------
--------------

These instructions are for Google Cloud Console
They detail a simple PKB setup to run basic tests on GCP. Additional setup may be required to run benchmarks on other providers and more complex benchmarks may also require additional setup that is specific to that benchmark
More complete instructions for all tasks can be found here

Go to your cloud console (console.cloud.google.com) and click the ‘Activate Cloud Shell’ button on the top right of the screen.

Clone or download the PerfkitBenchmarker Repo

    git clone https://github.com/GoogleCloudPlatform/PerfKitBenchmarker.git

Install PKB dependencies

    cd /path/to/PerfKitBenchmarker
    
    sudo pip install -r requirements.txt

Now you can run a simple throughput test on GCP using

    ./pkb.py --benchmarks=iperf



By default PKB will output results to the terminal and save them in the directory /tmp/perfkitbenchmarker/runs/

By adding the following flags, we can also push results to BigQuery

    --bigquery_table=<dataset.table> --bq_project=<project_id>
