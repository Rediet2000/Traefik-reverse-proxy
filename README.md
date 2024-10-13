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
          image: traefik:v3.1
          container_name: traefik
          restart: unless-stopped
          security_opt:
            - no-new-privileges:true
          networks:
            - proxy
          ports:
            - 80:80
            - 443:443
            - 8080:8080
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
            - "traefik.http.routers.traefik-dashboard.rule=Host(`app.example.com`)"
            - "traefik.http.routers.traefik-dashboard.entrypoints=web"
            - "traefik.http.services.traefik-dashboard.loadbalancer.server.port=8080"

networks:
  proxy:
    external: true

step 3

create this directory called traefik-data inside traefik directory 

        mkdir traefik-data

step 4 

create file which is called traefik.yml inside traefik-data 

    nano traefik.yml

.

      # API settings
      api:
        dashboard: true
        insecure: true  # Use this only for local or development environments
      
      # Logging settings
      log:
        level: DEBUG  # Log level set to DEBUG for detailed logging
      
      # Entry points for web and websecure traffic
      entryPoints:
        web:
          address: ":80"  # HTTP entry point
        websecure:
          address: ":443"  # HTTPS entry point
          http:
            middlewares:
              - secureHeaders@file  # Apply security headers defined in external file
            tls:
              certResolver: letsencrypt  # Use Let's Encrypt for certificate resolution
      
      # Providers for Docker and file configurations
      providers:
        docker:
          endpoint: "unix:///var/run/docker.sock"  # Docker socket for communication
          exposedByDefault: false  # Do not expose containers unless explicitly enabled
        file:
          directory: /configurations  # Directory for dynamic file configurations
          watch: true  # Watch the directory for changes
      
      # Certificate resolvers for Let's Encrypt
      certificatesResolvers:
        letsencrypt:
          acme:
            email: mail@example.com  # Email for Let's Encrypt registration
            storage: acme.json  # File to store ACME information
            keyType: EC384  # Use EC384 key type for certificates
            httpChallenge:
              entryPoint: web  # Use HTTP challenge on the web entry point for certificate validation
      
      # Transport settings for servers
      serversTransport:
        insecureSkipVerify: true  # Skip SSL verification (use only in dev or testing environments)

Step 5 create acme.json file inside traefik-data directory

        touch acme.json
        chmod 600 acme.json 
        
step 6 create configurations

        mkdir configurations

step 7 create dynamic.yml inside configurations directory

        nano dynamic.yml
.

        http:
     # Middleware to apply security headers to all routes
     middlewares:
       secureHeaders:
         headers:
           sslRedirect: true             # Redirect all HTTP traffic to HTTPS
           forceSTSHeader: true          # Enforce HTTP Strict Transport Security (HSTS)
           stsIncludeSubdomains: true    # Apply HSTS to all subdomains
           stsPreload: true              # Allow the domain to be preloaded into browsers' HSTS lists
           stsSeconds: 31536000          # HSTS max age set to 1 year (in seconds)
   
     # Routers define how incoming requests are routed to services
     routers:
       example_router:
         rule: "Host(`www.example.com`) || Host(`example.com`)"  # Match requests to 'example.com' or 'www.example.com'
         tls:
           certResolver: letsencrypt      # Use Let's Encrypt to automatically issue SSL certificates
         service: example_service         # Route traffic to 'example_service'
         middlewares:
           - secureHeaders                # Apply secureHeaders middleware for HTTPS and HSTS enforcement
   
       example2_router:
         rule: "Host(`www.example2.com`) || Host(`example2.com`)"  # Match requests to 'example2.com' or 'www.example2.com'
         tls:
           certResolver: letsencrypt      # Use Let's Encrypt for TLS
         service: example2_service        # Route to 'example2_service'
         middlewares:
           - secureHeaders                # Apply secureHeaders middleware
   
       app1_example_router:
         rule: "Host(`app.example.com`)"  # Route traffic for 'ais.example.com'
         tls:
           certResolver: letsencrypt      # Use Let's Encrypt for SSL certificates
         service: app1_example_service    # Route to 'app1_example_service'
         middlewares:
           - secureHeaders                # Apply secureHeaders middleware
   
       app2_example_router:
         rule: "Host(`app2.example.com`)"  # Route traffic for 'app2.example.com'
         tls:
           certResolver: letsencrypt      # Use Let's Encrypt for SSL
         service: app2_example_service    # Route to 'app2_example_service'
         middlewares:
           - secureHeaders                # Apply secureHeaders middleware
   
     # Services define backend servers where requests are forwarded to
     services:
       app1_service:
         loadBalancer:
           servers:
             - url: "http://localhost:port"  # Backend server for 'app1_service'
   
       app2_service:
         loadBalancer:
           servers:
             - url: "http://your-ip:port/app2"  # Backend server for 'app2_service'
   
       example_service:
         loadBalancer:
           servers:
             - url: "https://your-ip:port"  # Backend server for 'example_service'
   
       example2_service:
         loadBalancer:
           servers:
             - url: "https://your-ip:port/example"  # Backend server for 'example2_service'

              
Final step. go to traefik directory start application 

        docker-compose up -d 
        
check status application  status
        
        docker-compose logs -f 
