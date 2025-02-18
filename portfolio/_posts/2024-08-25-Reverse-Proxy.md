---
image: 
layout: post

sitemap: false
---

<img src="/assets/img/portfolio/proxy/dashboard.png" alt="Dashboard">

# Securing IT Infrastructure with Traefik Reverse&nbsp;Proxy


Reverse proxies are important tools to have when securing IT infrastructure. It acts as a barrier between clients and application servers. Proxies are notably known for their ability to speed up web traffic through the use of caching, load balancing, and routing. In this post, I will be demonstrating the traefik reverse proxy.

## What is Traefik

Traefik is a modern reverse proxy and load balancer designed for dynamic service discovery, making it an ideal choice for containerized applications. Unlike traditional proxies that require manual configuration, Traefik can automatically detect and route traffic to new services as they come online.

## Why I Chose Traefik?

<ul> 
    <li>Seamless Docker integration – Since Traefik can run as a Docker container, it integrates well with my Docker Swarm environment.</li>
    <li>Automatic service discovery – No need to manually update configurations when deploying new services.</li>
    <li>Built-in SSL/TLS management – Traefik integrates with Let’s Encrypt to automatically issue and renew SSL certificates.</li>
</ul>


## Deploying the Traefik Container

Since my infrastructure runs on Docker Swarm, I deployed Traefik on a manager node, as this is required for proper service orchestration.
While I will cover key parts of the Traefik configuration, the full docker-compose.yml file can be accessed <a href="https://github.com/yon0-0/homelab/blob/main/traefik/dockercompose.yml" target="_blank">here.</a>

### Configuring Entry Points

For a web application, Traefik requires at least two entry points:


<ul> 
    <li>Port 80 (HTTP) – Used for incoming unencrypted traffic.</li>
    <li>Port 443 (HTTPS) – Used for encrypted traffic.</li>
</ul>

To enforce encryption, I configured Traefik to automatically redirect all HTTP requests to HTTPS. This ensures that no sensitive data is transmitted over an unencrypted connection.

## SSL/TLS Certificates with Let’s Encrypt

One of the most powerful features of Traefik is its ability to automatically obtain and renew SSL/TLS certificates using Let’s Encrypt. Traefik stores these certificates in a JSON file for easy management.

### Verifying Domain Ownership

To obtain a valid SSL certificate, Let’s Encrypt requires domain verification. For this, I:

<ol>

<li>Purchased a domain through <a href="https://www.cloudflare.com/" target="_blank">Cloudflare</a> 
</li>
    <li>Created DNS records for each subdomain corresponding to my deployed services.
</li>
    <li>Configured Traefik’s DNS challenge to validate domain ownership.
</li>
</ol>

### Using a DNS Challenge for SSL

Instead of the HTTP challenge, I used the DNS challenge, which requires providing an API token in the Traefik configuration. This allows Traefik to modify DNS records and prove domain ownership.

With this setup, I can now encrypt and securely route traffic for any services running on my local infrastructure. Below is a crt.sh query to verify the SHA-256 fingerprint of my domain certificates.

<img src="/assets/img/portfolio/cert.png" alt="certificate">


## Traefik Dashboard & Security Enhancements

The Traefik dashboard provides real-time visibility into all services routed through the proxy. It automatically detects new services and updates routes accordingly.

Additionally, Traefik supports middlewares, which can modify HTTP request behavior. I implemented two key security middlewares:

<ol>

<li>Basic Authentication – Restricts dashboard access with a username and password.</li>
    <ul>
        <li>I generated a hashed password using the htpasswd command, which securely stores authentication credentials.</li>
    </ul>

<li>IP Allow List – Restricts access to trusted devices.</li>
    <ul>
        <li>While my services are already isolated on a VLAN, I added this extra layer of security to prevent unauthorized access.</li>
    </ul>

</ol>

## Reflection

Deploying the Traefik reverse proxy has been a valuable experience in my cybersecurity journey. Beyond traffic routing, this project deepened my understanding of network security, encryption, and access control—key components in securing IT infrastructure. It also reinforced the critical role reverse proxies play in protecting backend services from direct exposure. Additionally, I gained a deeper understanding of DNS-based validation and its impact on domain security and API-based authentication. This hands-on experience has strengthened my ability to implement security best practices in real-world environments.
