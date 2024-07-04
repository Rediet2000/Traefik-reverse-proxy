Introduction
Traefik is a modern HTTP reverse proxy and load balancer that makes deploying microservices and distributed applications effortless. It's designed to integrte seamlessly with your existing infrastructure components and supports various backends, including Docker, Kubernetes, and other orchestration tools. Traefik is highly dynamic, meaning it can automatically detect changes in your infrastructure and update its routing configuration on the fly.

Key Features
   
Dynamic Configuration

Traefik can automatically detect new services and changes in your infrastructure. It watches your orchestrator (like Docker or Kubernetes) and updates its configuration without any manual intervention.	
   
Load Balancing
   	
Traefik distributes incoming requests across multiple instances of your services, improving performance and availability.		
   
SSL Termination

Traefik handles SSL termination and can automatically obtain and renew SSL certificates using Let's Encrypt.
   
Middleware Support

Traefik supports various middlewares for features like authentication, rate limiting, IP whitelisting, and more. These middlewares can be easily configured and applied to your services.
   
Advanced Routing

Traefik offers advanced routing capabilities, including path-based routing, host-based routing, and header-based routing. This allows you to direct traffic to the appropriate service based on various criteria.
   
Monitoring and Metrics
   
Traefik comes with a built-in dashboard and supports multiple metrics backends (like Prometheus), making it easy to monitor the health and performance of your services.

Architecture

Traefik's architecture is composed of three main parts

Entrypoints

Entrypoints define how Traefik receives incoming requests. These can be HTTP, HTTPS, TCP, or UDP.

Routers

Routers analyze the incoming requests and determine which service should handle them based on the defined rules.

Services

Services represent the actual backend services that will handle the requests. They can be simple (like a single container) or complex (like a group of containers managed by an orchestrator).


How To install and configuration Traefik revese by using docker. 

Step 1 create create directory "traefik'

        mkdir traefik

Step 2 create dcoker compose file into docker-compose.yml

        nano docker-compose.yml

        version: "3"
        
        services:
          traefik:
            image: traefik:2.9
            container_name: traefik
            restart: unless-stopped
            security_opt:
              - no-new-privileges:true
            networks:
              - proxy
            ports:
              - 80:80
              - 443:443
            volumes:
              - /etc/localtime:/etc/localtime:ro
              - /var/run/docker.sock:/var/run/docker.sock:ro
              - ./traefik-data/traefik.yml:/traefik.yml:ro
              - ./traefik-data/acme.json:/acme.json
              - ./traefik-data/configurations:/configurations
            labels:
              - "traefik.enable=true"
              - "traefik.docker.network=proxy"
              - "traefik.http.routers.traefik-secure.entrypoints=websecure"
              - "traefik.http.routers.traefik-secure.middlewares=user-auth@file"
              - "traefik.http.routers.traefik-secure.service=api@internal"
        
        networks:
          proxy:
            external: true

step 3

create this directory called traefik-data inside traefik directory 

        mkdir traefik-data

step 4 

create file which is called traefik.yml inside traefik-data 

      nano traefik.yml
        api:
          dashboard: true
        log:
          level: DEBUG
        
        entryPoints:
          web:
            address: :80
            http:
              redirections:
                entryPoint:
                  to: websecure
        
          websecure:
            address: :443
            http:
              middlewares:
                - secureHeaders@file
              tls:
                certResolver: letsencrypt
        
        providers:
          docker:
            endpoint: "unix:///var/run/docker.sock"
            exposedByDefault: false
          file:
            filename: /configurations/dynamic.yml
        
        certificatesResolvers:
          letsencrypt:
            acme:
              email: your_email@example.com #add your email address
              storage: acme.json
              keyType: EC384
              httpChallenge:
                entryPoint: web
        
        serversTransport:
          insecureSkipVerify: true

Step 5 create acme.json file inside traefik-data directory

        touch acme.json
        chmod 600 acme.json 
        
step 6 create configurations

        mkdir configurations

step 7 create dynamic.yml inside configurations directory

        nano dynamic.yml

        http:
          middlewares:
            secureHeaders:
              headers:
                sslRedirect: true
                forceSTSHeader: true
                stsIncludeSubdomains: true
                stsPreload: true
                stsSeconds: 31536000
        
          routers:
            chat_router:
              rule: "Host(`example.domain.com`)" #replace with your actull domain name
              tls:
                certResolver: letsencrypt
              service: chat_service
        
          services:
            chat_service:
              loadBalancer:
                servers:
                  - url: "https://machine-ip:port" #repelace with your acuall ip 
         
        tls:
          options:
            default:
              sniStrict: true
              
Final step. go to traefik directory start application 

        docker-compose up -d 
        
check status application  status
        
        docker-compose logs -f 
