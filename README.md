# NGINX reverse proxy docker compose example
This project contains a docker-compose file for a NGINX reverse proxy with automatic Lets Encrypt SSL certificate 
generation.

Based on: [nginx-proxy](https://github.com/nginx-proxy/nginx-proxy) and [acme-companion](https://github.com/nginx-proxy/acme-companion).

## How it works
The `nginx-proxy` container acts as a [reverse proxy](https://en.wikipedia.org/wiki/Reverse_proxy) to redirect traffic to
the destination application container based on DNS name. It also provides Lets Encrypt SSL certificates 
to enable HTTPS.

The `nginx-proxy` container will be automatically configured via [docker-gen](https://github.com/nginx-proxy/docker-gen). 
The `docker-gen` application (included in the `nginx-proxy` container) listens to the docker socket. When a new container
is started, it wil generate an SSL certificate with the `acme-companion` container, and generate the config for the `nginx-proxy` container. 
The config will be installed in the `nginx-proxy` container via shared volumes. 

When the expiration date of a certificate is due, the `acme-companion` container automatically renews the certificate.

See the [nginx-proxy README](https://github.com/nginx-proxy/nginx-proxy) for more information about the
reverse proxy and the [acme-companion README](https://github.com/nginx-proxy/acme-companion) for more information about
certificate generation.

![Overview](http://www.plantuml.com/plantuml/proxy?cache=no&src=https://raw.githubusercontent.com/janverhoeckx/nginx-reverse-proxy-example/main/schema.puml)

## How to run
1. Create a docker network "webproxy": `docker network create webproxy`
2. Run docker compose file: `docker-compose up -d`

The reverse proxy container will now discover docker containers in the `webproxy` network. To start a container which should
be served by the reverse proxy with an SSL certificate, configure it with the environment variables:
   1. `VIRTUAL_HOST`: The DNS name of this application.
   2. `VIRTUAL_PORT`: The port this application is served at.
   3. `LETSENCRYPT_HOST`: The DNS name of this application for which an SSL certificate should be created.
   4. `LETSENCRYPT_EMAIL`: Email address which should be notified when there are problems with the SSL certificate renewal.

For example: `docker run --name test-webserver --network="webproxy" --env "VIRTUAL_HOST=my-domain.com" --env "VIRTUAL_PORT=80" --env "LETSENCRYPT_HOST=my-domain.com" --env "LETSENCRYPT_EMAIL=my-email@my-domain.com" nginxdemos/hello`
