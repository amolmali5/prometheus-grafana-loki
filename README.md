# Prometheus and Grafana
This repo contains installation guide for prometheus and grafana along with some other services


# Pre-requisites
Before we get started installing the Prometheus stack. Ensure you install the latest version of [docker] (https://docs.docker.com/engine/install/) and [docker swarm](https://docs.docker.com/engine/swarm/swarm-tutorial/) on your Docker host machine. Docker Swarm is installed automatically when using Docker for Mac or Docker for Windows.

# Installation & Configuration
Clone the project locally to your Docker host.

If you would like to change which targets should be monitored or make configuration changes edit the [/prometheus/prometheus.yaml](prometheus/prometheus.yaml) file. The targets section is where you define what should be monitored by Prometheus. The names defined in this file are actually sourced from the service name in the docker-compose file. If you wish to change names of the services you can add the "container_name" parameter in the `docker-compose.yaml` file.


Once configurations are done let's start it up. From the /prometheus project directory run the following command:

1. To use the docker-compose file

    ```
    docker compose up -d
    ```

2. To use the docker-stack file
    ```
    HOSTNAME=$(hostname) docker stack deploy -c docker-stack.yaml prom
    ```
    * In order to check the status of the newly created stack:
        ```
        docker stack ps prom
        ```

    * View running services:
        ```
         docker service ls
        ```
    * View logs for a specific service
        ```
        docker service logs prom_<service_name>
        ```

That's it the `docker stack deploy' command deploys the entire Grafana and Prometheus stack automatically to the Docker Swarm. By default cAdvisor and node-exporter are set to Global deployment which means they will propagate to every docker host attached to the Swarm.

This will create a folder named grafana_data for grafana database.

The Grafana Dashboard is now accessible via: `http://<Host IP Address>:3000` for example http://192.168.10.1:3000
```
	username - admin
	password - v12as34t (Password is stored in the `/grafana/config.monitoring` env file)
```
## Add Datasources and Dashboards
Grafana version 5.0.0 has introduced the concept of provisioning. This allows us to automate the process of adding Datasources & Dashboards. The [/grafana/provisioning](grafana/provisioning) directory contains the `datasources` which has two datasouces [prometheus](grafana/provisioning/datasources/datasource.yaml) and [loki](grafana/provisioning/datasources/ds.yaml) and `dashboards` directories. These directories contain YAML files which allow us to specify which datasource or dashboards should be installed. 

If you would like to automate the installation of additional dashboards just copy the Dashboard `JSON` file to `/grafana/provisioning/dashboards` and it will be provisioned next time you stop and start Grafana.


## Install Dashboards the old way

Created a Dashboard template which is available on [Grafana Docker Dashboard](https://grafana.com/grafana/dashboards/179). 

Go to Grafana Dashboard and simply select Import from the Grafana menu -> Dashboards -> New -> Import and provide the Dashboard ID [#179](https://grafana.com/grafana/dashboards/179)

This dashboard is intended to help you get started with monitoring.

Added more dashboards like ID 13112, 16310 and will be adding more in future.

### Add Additional Datasources
Now we need to create the Prometheus Datasource in order to connect Grafana to Prometheus 
* Click the `Grafana` Menu at the top left corner (looks like a fireball)
* Click `Data Sources`
* Click the green button `Add Data Source`.

**Ensure the Datasource name `Prometheus`is using uppercase `P`**


## Alerting

Alerting has been added to the stack with Email and Slack integration. Alerts have been added and are managed

Alerts              - `prometheus/alert.rules`
Email configuration - `alertmanager/config.yaml`

The Email configuration requires to build custom integration.
Edit the [/alertmanager/config.yaml](alertmanager/config.yaml) and add your email address (to and from), password and smarthost (make sure you have enabled SMTP AUTH service for "from" email)  

Slack configuration - `alertmanager/config.yaml`

The Slack configuration requires to build a custom integration.
* Open your slack team in your browser `https://<your-slack-team>.slack.com/apps`
* Click build in the upper right corner
* Choose Incoming Web Hooks link under Send Messages
* Click on the "incoming webhook integration" link
* Select which channel
* Click on Add Incoming WebHooks integration
* Copy the Webhook URL into the `alertmanager/config.yaml` URL section
* Fill in Slack username and channel

View Prometheus alerts `http://<Host IP Address>:9090/alerts`
View Alert Manager `http://<Host IP Address>:9093`

## Test Alerts

A quick test for alerts is to stop a service. Stop the node_exporter container and you should notice shortly the alert arrive in mail and slack. Also check the alerts in both the Alert Manager and Prometheus Alerts just to understand how they flow through the system.

High load test alert - `docker run --rm -it busybox sh -c "while true; do :; done"`

Let this run for a few minutes and you will notice the load alert appear. Then Ctrl+C to stop this container.


## Logs Visulization with Loki and Promtail

To monitor system logs, configuration is aready ready in [/promtal/config.yaml](promtail/config.yaml).

To view the docker container logs, add only labels shown as in example to the docker container/services in docker-compose/docker-stack, whose logs you wants to moniotor. Then go to the grafana and you can see the conatiner names in loki datasouce query section.
```
 nginx-app:
    container_name: nginx-app
    image: nginx
    labels:
      logging: "promtail"
      logging_jobname: "containerlogs"
```

If you want to monitor any other logs then provide that file path in commented lines in [/promtail/config.yaml](promtail/config.yaml).

For mor details go through [Loki] (https://github.com/grafana/loki/tree/main/production)

# To remove everything
1. If docker-compose file used:
    ```
    docker compose down
    ```
2. If docker-stack file used:
    ```
    docker stack rm prom
    ```
