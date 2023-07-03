![Alt text](md_assets/header.png)
<!-- # llama-cpp-ci-bench -->

### *... Continual integration benchmarking for LLM inference engine **[llama.cpp](https://github.com/ggerganov/llama.cpp)***

<br/>


> ### ⚠️ **WARNING**
> Spinning up GPU virtual machines can get expensive 💰 
> <i>please proceed carefully...</i> 




<br/>

## **About**
ToDo...


<br/>

## **Tech Stack**
- **[Azure cloud](https://azure.microsoft.com/en-us/products/virtual-machines/) by Microsoft** - we are targeting MS Azure exclusively (at this point)
- **[Azure CLI](https://learn.microsoft.com/en-us/cli/azure/what-is-azure-cli)** - `az` is command line tool to managed Azure infrastructure
- **[node.js](https://nodejs.org/en/about)** application logic, **[llama.cpp](https://github.com/ggerganov/llama.cpp)** process control and workflow
- **[InfluxDB](https://www.influxdata.com/products/influxdb-overview/)** - time series database (TSDB) for logs and metrics
- **[CouchDB](https://couchdb.apache.org/)** - simple, robust, noSQL database (distributable)
- **[Telegraf](https://www.influxdata.com/time-series-platform/telegraf/)** - for gathering machine and process telemetry
- \[Later] [Grafana](https://grafana.com/) for telemetry visualisation
- \[Later] Python & Jupyter for benchmark data analysis



<br/>

## **Architecture**
<!-- ![Alt text](md_assets/image.png) -->
<img src="md_assets/architeture-v4.png" width="75%" height="75%" /><br/><br/>



### **⚙️conductor VM**

#### **⚙️conductor Overview**
- always on small cloud VM
- Ubuntu 22.04
- node.js, nvm & [PM2](https://pm2.keymetrics.io/) to manage cron & processes
- InfluxDB database for logs & telemetry
- CouchDB database for config & bench result summaries
- Telegraf to gather machine telemetry

#### **⚙️conductor Functionality**
1. **polls** Gituhb API periodically for latest llama.cpp release
1. instructs Azure to create a GPU VM (⚙️bench-runner) and configures infrastructure etc `az` CLI commands


#### **Config database - CouchDB**
- bench test data
- Azure machine image data

#### **Logging Database - InfluxDB**
- llama.cpp stdout stream with timestamps
- nvidia-smi metrics
- cpu, gpu, ram metrics & other machine metrics

<br/>

### **⚙️bench-runner VM**

#### **⚙️bench-runner overview**
- emphemeral VM
- Ubuntu, Debian or Windows VM
- node.js, nvm & PM2
- nvidia drivers etc
- make, gcc etc
- Telegraf to gather machine telemetry & nvidia telemetry

#### **⚙️bench- runner functionality**
node.js runs a managed node.js sub-process;
1. node.js pulls git code - eg `git pull master-d7d2e6a`
1. node.s builds the code - eg `make clean && make -j`
1. node.js runs **[llama.cpp](https://github.com/ggerganov/llama.cpp)** - eg `time ./perplexity -m ./models/3B/open-llama-3b-q4_0.bin -f build/wiki.test.raw.406 -t 8` 
1. node.js sends `stdout` to 📂 InfuxDB
1. node.js sends llama.cpp process results to 📂 CouchDB
1. *[optionally] node.js switches to a different branch - GOTO #1*
1. node.js sends "bench session end" signal to **⚙️conductor** / 📂 CouchDB / 📂 InfluxDB
1. node.js shutsdown the VM 




---

<br/>

## **Setup**

ToDO