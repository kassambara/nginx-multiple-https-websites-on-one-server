
<!-- README.md is generated from README.Rmd. Please edit that file -->

# Host Multiple HTTPS Websites on One Server

provides a template to easily configure the deployement of multiple
websites on a single server. Reverse-proxy, nginx configuration files
and SSL certificate are created automatically for each website running
in a Docker cntainer.

A detailed explanation is provided at: [How To Host Multiple HTTPS
Websites on One
Server](https://www.datanovia.com/en/lessons/how-host-multiple-https-websites-on-one-server/)

## Prerequisites

### Install required tools and create domain names

  - Git, docker and docker-compose are installed on your server
  - Several websites run inside Docker containers on a single server.
    (Each one could either be a static files server, or Wordpress
    running on Apache, etc.
  - The domain name for each website is configured to point to the IP of
    the server. Your host must be publicly reachable on both port `80`
    and `443`.Check your firewall rules to make sure that these ports
    are open.

### Create websites directories

``` bash
# 0. settings
web_dir=/srv/www
myusername=kassambara
# 1. Create the website directory
sudo mkdir -p $web_dir
# 2. set your user as the owner
sudo chown -R $myusername $web_dir
# 3. set the web server as the group owner
sudo chgrp -R www-data $web_dir
# 4. 755 permissions for everything
sudo chmod -R 755 $web_dir
# 5. New files and folders inherit 
# group ownership from the parent folder
chmod g+s $web_dir
```

## Project structure

**Download a template into your website directories www**:

``` bash
web_dir=/srv/www
git clone https://github.com/kassambara/nginx-multiple-https-websites-on-one-server $web_dir
```

Inside `/nginx-proxy`, there are four empty directories: `conf.d`,
`vhost.d`, `html` and `certs`. These are used to store the nginx and the
Let’s Encrypt configuration files.

**Download the latest updated version of
[nginx.tmpl](https://raw.githubusercontent.com/jwilder/nginx-proxy/master/nginx.tmpl)**:

``` bash
curl -s https://raw.githubusercontent.com/jwilder/nginx-proxy/master/nginx.tmpl> $web_dir/nginx-proxy/nginx.tmpl
```

## Run the nginx reverse proxy

``` bash
# 1. Create the docker network
docker network create nginx-proxy

# 2. Create the reverse proxy with the 
# nginx, nginx-gen and nginx-letsencrypt containers
cd /srv/www/nginx-proxy/
docker-compose up -d
```

## Link a website to the running nginx-proxy

The `docker-compose.yml` file of the website, you want to link, should
include the following instructions provided in the template available in
the folder `your-website-one.com` (**not** the one from nginx-proxy
above). The content of the template looks like this:

``` yaml
version: '3.6'

services:
  my-app:
    image: nginx
    restart: always
    environment:
      # NGINX-PROXY ENVIRONMENT VARIABLES: UPDATE ME
      - VIRTUAL_HOST=your-website-one.com 
      - VIRTUAL_PORT=80
      - LETSENCRYPT_HOST=your-website-one.com 
      - LETSENCRYPT_EMAIL=your.email@domain.com
      # END NGINX-PROXY ENVIRONMENT VARIABLES
    expose:
      - 80

networks:
  default:
    external:
      name: nginx-proxy
```

1.  **Environment variables**:
      - `VIRTUAL_HOST`: your domain name, used in the nginx
        configuration.
      - `VIRTUAL_PORT`: (optional) the port your website is listening to
        (default to 80).
      - `LETSENCRYPT_HOST`: your domain name, used in the Let’s Encrypt
        configuration.
      - `LETSENCRYPT_EMAIL`: your email, used in the Let’s Encrypt
        configuration.
2.  **Ports**:
      - the exposed port (here 80) should be the same as the
        `VIRTUAL_PORT` above.
3.  **Network**:
      - your website container should be linked to the external docker
        network named `nginx-proxy` \`\`\`

Once the update of the `docker-compose.yml` file is done, you can
**start the website with**:

``` bash
cd /srv/www/your-website-one.com
docker-compose up -d
```

The website is automatically detected by the reverse proxy, has a HTTPS
certificate and is visible at <https://your-website-one.com>.

You can repeat this last step for any other container you want to proxy

## References

  - [Host multiple websites with HTTPS on a single
    server](https://medium.com/@francoisromain/host-multiple-websites-with-https-inside-docker-containers-on-a-single-server-18467484ab95)
  - [Automated nginx proxy for Docker containers using
    docker-gen](https://github.com/jwilder/nginx-proxy)
  - [LetsEncrypt companion container for
    nginx-proxy](https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion)
