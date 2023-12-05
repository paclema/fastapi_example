# FastAPI example

Install fastapi with:
    
```
pip install fastapi
pip install "uvicorn[standard]"

pip install python-multipart    # for file upload
```
or using the requirements.txt file:

```
pip install -r requirements.txt
```

Run the server with:

```
uvicorn main:app --reload
```

## Using FastAPI example on a Docker container:

1. Build the image with:

```
docker build -t fastapi_app_example .
```

2. Run the container with:

```
docker run -d --name fastapi_app_example_container -p 8080:8080 fastapi_app_example
```

If you need to remove a previous version of the container, you can do it by running:

```
docker rm -f fastapi_app_example_container
```

## Using FastAPI example on a Docker container and Nginx Reverse Proxy:

In this project [nginx](https://www.nginx.com) is used as a reverse proxy to serve multiple application instances running as Docker containers.

For this purpose besides nginx, [nginx-proxy/docker-gen](https://github.com/nginx-proxy/docker-gen) and [nginx-proxy/acme-companion](https://github.com/nginx-proxy/acme-companion) are also utilized.

Each of nginx, nginx-proxy/docker-gen and nginx-proxy/acme-companion run in their respective Docker containers and has the following responsibilities
* _nginx_ runs as a reverse proxy routing external requests to the corresponding application instances.
* _nginx-proxy/docker-gen_ keeps track of Docker containers and updates nginx configuration accordingly. Hence, as long as the containers to be servered are started with proper arguments, no manual intervention is necessary.
* _nginx-proxy/acme-companion_, on the other hand, handles SSL certificate creation and renewal activities, using [Let's Encryp](https://letsencrypt.org) certificates.

If you are runnning the container using also reverse proxy (installation guides can be found [here](https://github.com/nginx-proxy/acme-companion/blob/main/docs/Advanced-usage.md)), you can use the following command:


0. Download and mount the template file nginx.tmpl into the docker-gen container:
```
sudo curl https://raw.githubusercontent.com/nginx-proxy/nginx-proxy/main/nginx.tmpl > /var/opt/fastapiappexample/nginx/nginx.tmpl
```

1. Run the nginx container with:
```
docker run -d --restart unless-stopped \
    --name fastapiappexample-nginx-proxy \
    --publish 80:80 \
    --publish 443:443 \
	--publish 1443:1443 \
    --volume /var/opt/fastapiappexample/nginx/conf:/etc/nginx/conf.d  \
    --volume /var/opt/fastapiappexample/nginx/vhost:/etc/nginx/vhost.d \
    --volume /var/opt/fastapiappexample/nginx/html:/usr/share/nginx/html \
    --volume /var/opt/fastapiappexample/nginx/certs:/etc/nginx/certs \
    nginx

```

2. Run the docker-gen container with:
```
docker run -d --restart unless-stopped \
    --name fastapiappexample-nginx-proxy-gen \
    --volumes-from fastapiappexample-nginx-proxy \
    --volume /var/opt/fastapiappexample/nginx/nginx.tmpl:/etc/docker-gen/templates/nginx.tmpl:ro \
    --volume /var/run/docker.sock:/tmp/docker.sock:ro \
    nginxproxy/docker-gen \
    -notify-sighup fastapiappexample-nginx-proxy -watch -wait 5s:30s /etc/docker-gen/templates/nginx.tmpl /etc/nginx/conf.d/default.conf
```

3. Run the acme-companion container for autimated creation/renewal Let's Encrypt  certificates with:
(Select your email here)
```
docker run -d --restart unless-stopped \
    --name fastapiappexample-nginx-proxy-acme \
    --volumes-from fastapiappexample-nginx-proxy \
    --volume /var/run/docker.sock:/var/run/docker.sock:ro \
    --volume /var/opt/fastapiappexample/nginx/acme:/etc/acme.sh \
    --env "NGINX_DOCKER_GEN_CONTAINER=fastapiappexample-nginx-proxy-gen" \
    --env "DEFAULT_EMAIL=email@example.com" \
    nginxproxy/acme-companion
```

4. Start the application container adding the proxy configurations:
First create a docker network and connect the nginx and docker-gen containers to it:
```
docker network create network_fastapi_app_example
docker network connect network_fastapi_app_example fastapiappexample-nginx-proxy
docker network connect network_fastapi_app_example fastapiappexample-nginx-proxy-gen
```

Configure your endpoint (in this case ) And start the application on this network too:
(Select your email here)

```
docker run -d --restart unless-stopped \
    --expose 8080 \
    --network network_fastapi_app_example \
    --name fastapi_app_example_container \
    --env "VIRTUAL_HOST=subdomain.example.domain.com" \
    --env "LETSENCRYPT_HOST=subdomain.example.domain.com" \
    --env "VIRTUAL_PROTO=http" \
    --env "VIRTUAL_PORT=8080" \
    fastapi_app_example
```

If you would like to access to the container server locally, you can add the parameter `-p 8080:8080`

If you would like to deactivate fastAPI docs, you can add the parameter `--env "FASTAPI_DISABLE_DOCS=1"`

```
docker run -d --restart unless-stopped \
    --expose 8080 \
    -p 8080:8080 \
    --network network_fastapi_app_example \
    --name fastapi_app_example_container \
    --env "VIRTUAL_HOST=subdomain.example.domain.com" \
    --env "LETSENCRYPT_HOST=subdomain.example.domain.com" \
    --env "VIRTUAL_PROTO=http" \
    --env "VIRTUAL_PORT=8080" \
    --env "FASTAPI_DISABLE_DOCS=1" \
    fastapi_app_example
```