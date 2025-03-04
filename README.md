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

On you virtual machine, clone the repostitory via the 
# Notice

This is reposity is designed for demonostration, non production environments.  Don't even think about running this environment in a production environment.   It probably won't work and you won't be happy with it.  If for some stupid reason you do run this repository as a production instance for heaven's sake, don't call IBM support for help.

# Acknoledgements

This repository would not be possible without the [Play With Grafana Mimir Demo](https://github.com/grafana/mimir/blob/main/docs/sources/mimir/get-started/play-with-grafana-mimir/index.md) developed by the Graphana team.   This repo started from the demo environment.   It was then stripped of all but the configuration necessary to run an environment capable of supporting IBM COS OTEL.  I highly recommend you go through their demo if you can.
