# Infrastructure

We have some developers services:

- [PyPi-Registry](https://github.com/pypiserver/pypiserver)
- [Docker-Registry](https://docs.docker.com/registry/), [github](https://github.com/docker/distribution), [docker.hub](https://hub.docker.com/_/registry)
- [Vault](https://github.com/hashicorp)
- [Prometheus](https://prometheus.io/)

We use a very simple configuration for our needs.

# Vault

Vault is saver of credentials and shared data of our services. We save here shared: 

- urls, ports
- credentials
- configs of services

Now, we have next structure of namespaces:

- service-1
    - environment-1
    - environment-2
    - environment-3
    - local isolated environment
- service-2
    - environment-1
    - environment-2
    - environment-3
    - local isolated environment
    
# PyPi registry

PyPi registry is saver of private python package. This is wsgi server without reverse proxy. We protected them with 
[htpasswd](https://httpd.apache.org/docs/2.4/programs/htpasswd.html) way. 

# Docker registry

This is private docker registry. We save have our docker images of services. We protected them with 
[htpasswd](https://httpd.apache.org/docs/2.4/programs/htpasswd.html) way. More about htpasswd in **Credentials** section

### Communicate with registry

Before push, you need [login](https://docs.docker.com/engine/reference/commandline/login/):

    docker login <url>
    
Set tag for image:

    docker tag <image-id> <url>/<tag>

After that, docker asks you login and password. Now, you can push concrete image by tag:

    docker push <url>/<tag>
    
Now, you can see your image into registry by endpoint (with authentication):

    <url>/v2/_catalog
   
Downloading image from registry by tag:

    docker pull <url>/<tag>
    
More info about docker registry API, you can find [here](https://docs.docker.com/registry/spec/api/).
    
### References

You can see more info:

* [SSL](https://habr.com/ru/post/320884/)
* [docker-registry API](https://docs.docker.com/registry/spec/api/) 

# Prometheus

This is monitoring service. Now, anyone can go here, therefore we don't use credentials. If you want to add new service, you go to `./prometheus/config.yml`. Than, in section `scrape_configs`, you need add any targets:

      - targets: ['<your-host>:9090']
      
After that, rebot container.

# Run

You can run our infrastructure by next way for production:

    RESTART=always docker-compose up
    
Or for locally:

    RESTART=no docker-compose up

It is not a very useful to starting docker compose in local machine with mode `restart: always`. Containers always 
restart. Then killing them is not a very simple case. This is can be outrageously.

# Credentials

If you want to add new user, you can generate [bcrypt-11](http://aspirine.org/htpasswd_en.html). You need to generate 
new hash and pass them into `.htpasswd` as new line.

After update your credentials, you need reboot the service.

### Algorithm of generation

You need to use htpasswd. 

1. First make sure you have the passlib module installed (note that passlib>=1.6 is required), which is needed for 
parsing the Apache htpasswd file specified by the -P, --passwords option (see next steps):
2. Create the Apache htpasswd file with at least one user/password pair (you'll be prompted for a password).
3. Add password to $HOME/.credentionals/htpasswd to a new line and restart docker-registry server

