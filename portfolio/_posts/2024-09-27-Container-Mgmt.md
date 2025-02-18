---
image: 
layout: post
sitemap: false
---


# Cluster Orchestration and Visualization
	
<img src="/assets/img/portfolio/metrics/grafana.png" alt="Grafana">



System monitoring is a crucial aspect of managing any IT infrastructure.Whether for predictive analytics, reporting, or security monitoring, having access to real-time server metrics ensures system reliability and performance. In this post I will walk through the tools I use to manage my Docker Swarm cluster. These tools focus on container orchestration, system monitoring, and cluster visualization.

## Container Management with Portainer

There are numerous tools available for monitoring and managing containerized environments. One of the most effective is Portainer, a web-based UI designed for managing Docker, Kubernetes, and other containerized services. It simplifies container deployment, log access, network management, and more.

Portainer can be used in both self-hosted environments and cloud platforms like AWS and Azure. For a Docker Swarm cluster, the deployment consists of:

<ol>
<li>Portainer Server – Runs on the Swarm manager node and provides the UI.
</li>
<li>Portainer Agent – Runs on each worker node, allowing the server to communicate with them and manage services across the cluster.</li>    
</ol>

The agent listens on a specified port and relays data back to the Portainer server, ensuring a unified management interface across all nodes.

### Networking Considerations

Since my Portainer service is behind a Traefik reverse proxy, it must be part of an overlay network to communicate with other nodes. In Docker, an overlay network is a virtual network that enables services across different physical hosts to interact as if they were on the same local network. This is achieved through tunneling, which abstracts the underlying infrastructure and ensures seamless communication between containers.

I also configured Portainer with a custom domain. This allows the service to benefit from SSL/TLS encryption and automatic certificate management through Traefik.


## Gathering System Metrics

For real-time system monitoring, I implemented Prometheus, an open-source monitoring tool that scrapes and stores time-series data from various sources. Prometheus relies on exporters to collect and format metrics before ingestion.

### Choosing the Right Exporters

<ul>
<li>cAdvisor (Container Advisor) – Developed by Google, this tool monitors containerized applications, providing insights into CPU, memory, disk, and network usage.
</li>
<li>Node Exporter – Captures OS-level metrics from the host machine, such as CPU, memory, disk usage, and network activity.</li>    
</ul>

Since my services are running behind the Traefik reverse proxy, I had to expose a metrics endpoint to allow Prometheus to scrape data. I achieved this by modifying the Traefik configuration with the following settings:


```

Ports:
     - target:1138
       Published:1138 #Expose metrics port on the host
       protocol: tcp
       mode: host	

command:

	- --entrypoints.metrics.address=:1138 # metrics entrypoint
	- --metrics.prometheus=true 
	- --metrics.prometheus.entryPoint=metrics
	- --metrics.prometheus.addEntryPointsLabels=true

```

After configuring cAdvisor and Node Exporter, I assigned each a subdomain and a /metrics path prefix, which Prometheus uses to query and retrieve data.

<img src="/assets/img/portfolio/metrics/prometheus.png" alt="prometheus">


## Data Visualization with Grafana

Once Prometheus was set up to scrape data, I needed a way to visualize and interpret the collected metrics. I chose Grafana, a powerful open-source visualization tool that integrates seamlessly with Prometheus.

<ul>
<li>Custom dashboards with real-time graphs and alerts</li>
<li>Flexible query options using PromQL (Prometheus Query Language)</li>
<li>Integration with multiple data sources, including SQL databases, Elasticsearch, and cloud monitoring services</li>
</ul>

### Configuration & Deployment

Setting up Grafana in my docker-compose.yml file was straightforward. I simply:


<ol>
<li>Defined the domain URL for Grafana</li>
<li>Flexible query options using PromQL (Prometheus Query Language)</li>
<li>Mounted the necessary config and data volumes</li>
<li>Routed traffic through Traefik for secure access
</li>
</ol>

Once deployed, I spent time fine-tuning PromQL queries to display relevant metrics. After some trial and error, I finalized a dashboard that provides real-time insights into my cluster’s performance.


## Reflection

Working on this project deepened my understanding of container orchestration, system monitoring, and API communication. Initially, I had limited experience with setting up metrics collection and visualization tools. However, implementing Prometheus, cAdvisor, Node Exporter, and Grafana has given me hands-on exposure to how these components function in a production-like environment.

Additionally, integrating Traefik as a reverse proxy helped me grasp networking concepts like overlay networks and service discovery in a Docker Swarm cluster. Securing services with SSL/TLS was another critical learning experience.

Overall, this project has been an invaluable step toward understanding and deploying scalable, monitored, and secure IT infrastructures.