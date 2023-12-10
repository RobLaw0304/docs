
# Advanced Manual Proxying Between Saltboxes

The recommended approach to instantiating MediaBox and FeederBox is to utilize two different public IP addresses or two different domain names.  If that is not possible, some manual work needs to be done in order to proxy requests through to the correct host where the services are running.

For Example, you set up a MediaBox and get every service working through your domain that you want.  When you go to install FeederBox on another machine within your network, without prior work, none of those services will be able to be routed to any box other than your MediaBox.

This limitation can be bypassed by performing several manual steps.

1. Choose a Box to be your "master"
2. Remove Authelia from the other box
3. Set authelia -> master to no on the non-master box
4. Create a file custom.yml in your /opt/traefik directory
5. add this code to the file for each service you want to proxy
6. 
IMPORTANT: `APPNAME` and `DOMAIN.TLD` are placeholders.  *You need to change that* **everywhere it appears** to match the application you are proxying.

## Docker Compose

=== "Using Traefik (Authelia)"
    ```yaml
http:
  routers:
    APPNAME-http: 
      entryPoints:
      - "web" 
      rule: "Host(`APPNAME.DOMAIN.TLD`)" 
      middlewares:
      - globalHeaders
      - redirect-to-https@docker
      - cloudflarewarp@docker
      priority: 20
      service: APPNAME
    APPNAME: 
      entryPoints:
      - "websecure"
      rule: "Host(`APPNAME.DOMAIN.TLD`)" 
      middlewares:
      - globalHeaders 
      - secureHeaders
      - cloudflarewarp@docker
      priority: 20
      service: APPNAME 
      tls:
        certresolver: cfdns
        options: securetls@file

    networks: # (21)!
      saltbox:
        external: true
    ```

    3.  Defines the name of the container.
    6.  Defines the subdomain and domain you want to be able to reach this service on.
    7.  Adds saltbox headers for cloudflare, redirecting to https, and global default.
    12. Defines the name of the service which will contain the local url and port we use to access it locally
    6.  Defines which Docker networks the container will join upon creation.
