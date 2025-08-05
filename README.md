# IBM COS OTEL Demo Environment

# Why?

Version 3.19 of the ClevOS introduced support for Open Telemetry collectors.   This feature allows for the deployment of Open Telemetry collectors to all of the IBM COS appliances and supports the collection of metrics and sent the metric collection stack of your choice.

# What?

This demo environment spins up a containerized metric stack based on the following:

* [Grafana Mimir](https://grafana.com/oss/mimir/)
  * Three instances of monolithic-mode Mimir to provide high availability
  * Multi-tenancy enabled (tenant ID for IBM COS is lab)
  * Stores metrics from Prometheus and OpenTelemetry over time.
* [Minio](https://min.io/)
  * S3-compatible persistent storage for blocks, rules, and alerts
* [Prometheus](https://prometheus.io/)
  * Scrapes Grafana Mimir metrics, then writes them back to Grafana Mimir to ensure availability of ingested metrics
* [Grafana](https://grafana.com/)
  * Includes a preinstalled data source to query Grafana Mimir (from the Grafana Mimir demo)
  * Includes preinstalled dashboards for monitoring Grafana Mimir (from the Grafana Mimir demo)
  * Does **NOT** include any preinstalled dashboards for quering or monitoring IBM COS
  * Allows you to visualize, monitor, and analyze data from mutiple sources.  In this case, prometheus.
* Load balancer
  * A simple [NGINX](https://nginx.org/)-based load balancer that exposes Grafana Mimir endpoints on the host

The diagram below illustrates the relationship between these components:
![Component Diagram](documentation/tutorial-architecture.png "Demo Component Diagram")

The following ports will be exposed on the host:

    Grafana on http://< hostname/ip address >:9000
    Grafana Mimir on http://< hostname/ip address >:9009

There are mulitple ways to collect Open Telemetry metrics from IBM COS.  This is but one way.

# How To Setup This OTEL Demo Environment

## IBM COS Demo Environment

The OTEL stack that this repository builds will support the following IBM COS demo environment:

* 1x IBM COS Manager
* 4x IBM COS Accesser(s)
* 15x IBM COS Slicestors(s)
  * 1x 12 Wide Storage Pool (Standard Mode)
  * 1x 3 Wide Storage Pool (Concentrated Dispersal Mode)

Basically, an IBM COS solution consisting of up to twenty (20) IBM COS appliances.

## OTEL Stack Prerequisites

The following are perquisites for the demo stack:

* 4x CPU
* 16GB memory
* 32GB disk space
* A running/configured docker environment

This environment was tested in a Debian 12 Bookworm virtual image using the above prerequisites.

## How to Run / Start the Demo

The instructions below assume that the user you are running can manage docker as a non-root user.   Follow the instructions [Manage Docker as a non-root user](https://docs.docker.com/engine/install/linux-postinstall/) to allow non-root users to manage your local docker instance.

On you virtual machine, clone the github repository, [bcleonard/ibm_otel_demo_environment.git](https://github.com/bcleonard/ibm_otel_demo_environment).  There are several ways to clone a github repository, choose the method that is easiest for you.

then change into the repository via the following command via the CLI:

```bash
cd ibm_otel_demo_environment
```

Next, start the stack by executing the docker compose file via the CLI:

```bash
docker compose up -d
```

This command will download all the docker container images, create the docker network, create all the necessary docker volumes, start all the docker container instances and stitch the whole application stack together

## How to configure and deploy IBM COS OTEL Collectors

First you need to make sure that the installed version of ClevOS is 3.19.x.x or higher.

Second, you need to configure and deploy an OTEL collector.  Use the following step to configure and deploy an OTEL collector.

1. Log into the IBM COS Manager GUI
2. Select the "Settings" tab, the "Monitoring" section on the left and then the "Telemetry" link on the bottom of the page.
3. Select the "Create Collector" button in the upper right-hand side of the web page
4. Select the "General Tab"
5. Make sure the "Collector Status" is enabled.
6. Type in a Name for the collector, such as "IBM COS OTEL OBSERVE"
7. In the "Exporter Endpoint" section type in the URL, "http://<ip address / hostname>:9009/otlp/v1/metrics".  Use the hostname or ip address of the server that has the OTEL stack you just created.
8. For the "Exporter Type", select "otlphttp"
9. In the Export, Additional HTTP Headers, type in the text "X-Scope-OrgID=lab"
10. In the Export, Additional Secret HTTP Headers, type in the text "X-Scope-OrgID=lab"
11. Next, select the Deployments tab.
12.  Select the Access Pools, Storage Pool Sets, or individual devices to deploy the collector.
13.  Select the "Save" button and the collector configuration will be saved and deployed to the appliances selected.

# Acessing the OTEL Stack

The OTEL stack will expose two ports that you can access.  The first is Grafana Mimir which can be accessed via the URL:  http://< hostname / ip address >:9009.  This is the Grafana Mimir Admin page.   This is not the main inferface but just gives you a view of the Grafana Mimir configuration and status.

The 2nd is the Grafana interface which can be accessed via the URL:  http://< hostname / ip address >:9000.   This is the main Grafana webpage.

To see all the metrics, select "Explore", "Metrics" and then select the "Let's Start! ->" blue button.   This will bring up a metrics dashboard that shows you **EVERY** IBM COS metric that is collected and sent to the stack.  In the upper right-hand side of the page, there should be a selector titled "OTEL experience".  Enable it.  This provides a basic OTEL experience.

There are no IBM COS dashboards included in this demo.   This demo is designed to provide you with a test stack to start collecting metrics with.

# Gotcha / Limitations

There are some gotcha and limitations you need to be aware of.  Most of the issues you **MIGHT** run into are listed here, along with some way to address them (if known).

* Sizing constraints - The size requirements the OTEL stack might be too big / too much for your environment.   The number of appliances you have deployed connectors to has a direct correlation on the amount of memory, CPU and disk space allocated.   If you need to, reduce the CPU, followed by memory.  I don't recommend you reduce the disk space.   Reducing the number of appliances sending in data will reduce the CPU, memory and disk requirements.
* Disk Space - Metric data can grow fast.  The collectors send a large amount of data to the stack which can overload the environment.  The demo environment limits the disk space to 10GB and/or 1 day.   The is controlled via the Prometheus cli options in the docker-compose.yml file via the "*--storage.tsdb.retention.time=1d*" and "*--storage.tsdb.retention.size=10GB*".
* Mimir overload - when the stack starts up, all metric data that was **NOT** successfully send to the stack is sent.   If it’s been a while since metric data was send, this can be alot of data.   If mimir is overloaded, you'll see several different types of messages:
  * The first message you might see is "*the request has been rejected because the tenant exceeded the ingestion rate limit, set to 30000 items/s with a maximum allowed burst of 200000. This limit is applied on the total number of samples, exemplars and metadata received across all distributors*".  If you get these for a while, its fine.  Sustained entries that never go away are a problem.  I've tried to choose values that will allow the system to catchup, but if the system never does, they need to be adjusted.   In the mimir.yml configuration file, change the value of "*ingestion_rate*" to something higher than the current value.   The value of 30000 was chosen to reflect 10000 per mimir instance.   I never needed to adjust the "*ingestion_burst_size*", but it’s there in case you need to.
  * The second message you might see is "*detected an error while ingesting OTLP metrics request (the request may have been partially ingested)" httpCode=400 err="send data to ingesters: failed pushing to ingester mimir-2: user=lab: per-user series limit of 150000 exceeded (err-mimir-max-series-per-user). To adjust the related per-tenant limit, configure -ingester.max-global-series-per-user, or contact your service administrator. (sampled 1/10)*".  If you get these for a while, its fine.  Sustained entries that never go away are a problem.  I've tried to choose values that will allow the system to catchup, but if the system never does, they need to be adjusted.   In the mimir.yml configuration file, change the value of "*max_global_series_per_user*" to something higher than the current value.  The default is *1500000*.
* Memory and/or CPU overload - If the COS appliance are generating too many metrics, the server running the stack will run really low on memory and the CPU will be overloaded.  There are a couple of options you can try:
  * First, you can remove the collectors for the appliances they are deployed on.   Start with just a few and then deploy collectors to appliances one by one to keep the Memory / CPU to acceptable levels.
  * Increase the number of CPUs.   I'm not sure how effective this will be.  The demo is sized so there is one (1) CPU per mimir instance and one (1) for the OS.   If CPU are less than mimir instances + 1, increasing the number of CPU's is an easy thing to try.
  * Increasing the amount of memory - I'm not sure how effective this or if it is even necessary.   I've had limited success in fixing overload issues by changing the amount of memory.

# Sizing & Resource Requirements

IBM is following the industry standard of not supplying the exact number of metrics nor providing documentation for every single metric.  As the product evolves and changes, the number of metrics will change as well.

Grafana gives guidance on how to calculate the amount resources needed in [Grafana Mimir Planning Capacity](https://grafana.com/docs/mimir/latest/manage/run-production-environment/planning-capacity/) and its all based on the number of active series.

| Appliance            | Max # of Active Series/24 hours | Disk Space Used (MB) |
| -------------------- | :-----------------------------: | :------------------: |
| Manager (Non-HA) | ~19,000 | ~325 |
| Manager (HA)     | **PENDING** | **PENDING** |
| **Accesser** - Standard / Vault Mode |
| No Load   | ~15,600 | ~267 |
| With Load | ~19.375 | ~300 |
| **Accesser** - Standard / Container Mode |
| No Load   | **PENDING** | **PENDING** |
| With Load | **PENDING** | **PENDING** |
| **Slicestor**  - Vault Mode |
| Standard | ~25,600 | ~437 |
| CDM | **PENDING** | **PENDING** |
| **Slicestor** - Container Mode |
| Standard | **PENDING** | **PENDING** |
| CDM | **PENDING** | **PENDING** |

The methodogy used to determine this is documented in [Issue #5](https://github.com/bcleonard/ibm_otel_demo_environment/issues/5).

# How To Get Help

If you have questions, comments or concerns, the only way to get help is through this repository. We recommend searching through [issues first](https://github.com/bcleonard/ibm_otel_demo_environment/issues). If you don't find what you're looking for, feel free to open a new issue. Please do NOT open a ticket with IBM Support. They can't and won't help you.

# IBM COS OTEL Dashboards

This repository does not contain any IBM COS OTEL dasbboards.  That is by design.  The [IBM COS OTEL Dashboards](https://github.com/bcleonard/ibm_cos_otel_dashboards) repository contains dashboards for you to review and use.

# Notice

This is repository is designed for demonstration, non-production environments.  Don't even think about running this environment in a production environment.   It probably won't work, and you won't be happy with it.  If for some stupid reason you do run this repository as a production instance for heaven's sake, don't call IBM support for help.

# Acknowledgements

This repository would not be possible without the [Play With Grafana Mimir Demo](https://github.com/grafana/mimir/blob/main/docs/sources/mimir/get-started/play-with-grafana-mimir/index.md) developed by the Grafana team.   This repo started from the demo environment.   It was then stripped of all but the configuration necessary to run an environment capable of supporting IBM COS OTEL.  I highly recommend you go through their demo if you can.
