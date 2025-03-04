# IBM OTEL Demo Environment

# Why?

Version 3.19 of the ClevOS introducted support for Open Telemetry collectors.   This feature allows for the deployment of Open Telemetry collectors to all of the IBM COS appliances and supports the collection of metrics and sent the metric collection stack of your choise.

# What?

This demo environemnt spins up a containerized metric stack based on the following:

* [Grafana Mimir](https://grafana.com/oss/mimir/)
  * Three instances of monolithic-mode Mimir to provide high availability
  * Multi-tenancy enabled (tenant ID is demo)
* [Minio](https://min.io/)
  * S3-compatible persistent storage for blocks, rules, and alerts
* [Prometheus](https://prometheus.io/)
  * Scrapes Grafana Mimir metrics, then writes them back to Grafana Mimir to ensure availability of ingested metrics
* [Grafana](https://grafana.com/)
  * Includes a preinstalled datasource to query Grafana Mimir (from the Grafana Mimir demo)
  * Includes preinstalled dashboards for monitoring Grafana Mimir (from the Grafana Mimir demo)
* Load balancer
  * A simple [NGINX](https://nginx.org/)-based load balancer that exposes Grafana Mimir endpoints on the host

The diagram below illustrates the relationship between these components:
![Component Diagram](documentation/tutorial-architecture.png "Demo Component Diagram")

The following ports will be exposed on the host:

    Grafana on http://< hostname/ip address >:9000
    Grafana Mimir on http://< hostname/ip address >:9009

There are mulitple ways to collect Open Telemetry metrics from IBM COS.  This is but one way.


why

when
# How

## IBM COS Demo Environemnt

The following demo IBM COS environment consisting of:

* 1x IBM COS Manager
* 4x IBM COS Accesser(s)
* 15x IBM COS Slicestors(s)
  * 1x 12 Wide Storage Pool (Standard Mode)
  * 1x 3 Wide Storage Pool (Concintrated Disperal Mode)

## OTEL Stack Prerequestes

The following are prequestuest for the demo stack:

* 4x CPU
* 16GB memory
* 30GB disk space
* running docker environment

This environment was tested in a Debian 12 Bookworm virtual image using the above prerequisites.

## How to Run / Start the Demo

On you virtual machine, clone the repostitory via the following command via the CLI:

```bash
git clone git@github.com:bcleonard/ibm_otel_demo_environment.git
```

then change into the repository via the following command via the CLI:

```bash
cd ibm_otel_demo_environment
```

Next, start the stack by executing the docker compose file via the CLI:

```bash
docker compose up -d
```

This command will download all the docker conainter images, create the docker network, create all the necessary docker volumes, start all the docker container instances and stitch the whole applicaiton stack together

## How to configure and deploy IBM COS Otel Collectors

First you need to make sure that the installed version of ClevOS is 3.19.x.x or higher.

Second, you need to configure and deploy an OTEL collector.  Use the following step to configure and deploy an OTEL collector.

1. Log into the IBM COS Manager GUI
2. Select the "Settings" tab, the "Monitoring" section on the left and then the "Telemetry" link on the bottomr of the page.
3. Select the "Create Collector" button in the upper right hand side of the web page
4. Select the "General Tab"
5. Make sure the "Collector Status" is enabled.
6. Type in a Name for the collector, such as "IBM COS OTEL OBSERVE"
7. In the Export, Additional HTTP Headers, type in the text "X-Scope-OrgID=lab"
8. In the Export, Additional Secretry HTTP Headers, type in the text "X-Scope-OrgID=lab"
9. Next, select the Deployments tab.
10.  Select the Access Pools, Storage Pool Sets, or indivivual devices to deploy the collector.
11.  Select eh "Save" button and the collector configuration will be saved and deployed to the appliances selected.

# How to access the OTEL Stack

The OTEL stack will expose two ports that you can access.  The first is Grafana Mimir which can be accessed via the URL:  http://< hostname / ip address >:9009.  Thiis the Grafana Mimir Admin page.   This is not the main inferfact, but just gives you a view of the Grafana Mimir configuraiton and status.

The 2nd is the Grafana interface which can be accessed via the URL:  http://< hostname / ip address >:9000.   This is the main Grafana webpage.

To see all the metrics, select "Explore", "Metrics" and then select the "Let's Start! ->" blue button.   The will bring up a metrics dashboard that shows you **EVERY** IBM COS metric that is collected and sent to the stack.  In the upper right hand side of the page, there should be a selector titled "Otel experience".  Enable it.  This provides a basic Otel experience.

There are no IBM COS dashboards included in this demo.   This demo is designed to provide you with a test stack to start collecting metrics with.

# Gotcha / Limitations

There are some gotcha and limitations you need to be aware of.  Most of the issues you **MIGHT** run into are listed here, along with some way to address them (if known).

* Sizing constraints - The size requirements the OTEL stack might be too big for your environment.   The number of appliances you have deployed connectors to has a direct correlation on the amount of memory, cpu and disk space allocated.   If you need to, reduce the CPU, followed by Memory.  I don't recommend you reduce the disk space.   Reducing the number of appliance sending in data will reduce the CPU, memory and disk requirements.
* Disk Space - Metric data can grow fast.  The collectors send a large amount of data to the stack which can overload the environment.  The demo environment limits the disk space to 10GB and/or 1 day.   The is controlled via the prometheus cli options in the docker-compose.yml file via the "--storage.tsdb.retention.time=1d" and "--storage.tsdb.retention.size=10GB".
* Mimir overload - when the stack starts up, all metric data that was **NOT** successfully send to the stack is send.   If its been a while since metric data was send, this can be alot of data.   If mimir is overloaded, you'll see messages similar to "the request has been rejected because the tenant exceeded the ingestion rate limit, set to 30000 items/s with a maximum allowed burst of 200000. This limit is applied on the total number of samples, exemplars and metadata received across all distributors".  If you get these for a while, its fine.  Sustained entries that never go away are a problem.  I've tryied to choose values that will allow the sytem to catchup, but if the system never does, they need to be adjusted.   In the mimir.yml configuration file, change the value of "ingestion_rate" to something higher than the current vault.   The value of 30000 was chosen to reflect 10000 per mimir instance.   I never needed to adjust the ingestion_burst_size, but its there in case you need to.
* Memeory and/or CPU overload - If the COS appliance are generating too much metrics, the server running the stack will run really low on memory and the CPU will be overloaded.  There are a couple of options you can try:
  * First, you can remove the applinances the collectors are on and then add then back in one by one as needed to keep the Memory / CPU to accetble levels
  * Incrase the number of CPU.   I'm not sure how effectvee this will be.  I have it sized so there is one CPU per mimir instance and 1 for the OS.   If CPU are less than mimir instances + 1, increaseing the number of CPU's is an easy fix.
  * Increasing the amount of memory - I'm not sure how effective this or if it necessary.   I've had limited success in fixing overload issues by changing the amount of memory.


# Notice

This is reposity is designed for demonostration, non production environments.  Don't even think about running this environment in a production environment.   It probably won't work and you won't be happy with it.  If for some stupid reason you do run this repository as a production instance for heaven's sake, don't call IBM support for help.

# Acknoledgements

This repository would not be possible without the [Play With Grafana Mimir Demo](https://github.com/grafana/mimir/blob/main/docs/sources/mimir/get-started/play-with-grafana-mimir/index.md) developed by the Graphana team.   This repo started from the demo environment.   It was then stripped of all but the configuration necessary to run an environment capable of supporting IBM COS OTEL.  I highly recommend you go through their demo if you can.
