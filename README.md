# squid
Squid Proxy build scripts

## What is it

Dockerfile running Squid Proxy (v4.17) & (v5.4) using SSL on an Alpine base image.

[squid] (http://www.squid-cache.org/Intro/)

## Why you'd use it

[why?] (http://www.squid-cache.org/Intro/why.html)

## How to use
> Note **If you're using linux, you need to run following commands using sudo**

build image or pull from dockerhub
**Squid Configuration Files** -> `config/`

For now we have `squid.conf.basic` which has a basic configuration for running squid.
and `squid.conf.ssl` which enables a basic SSL configuration.
You can apply changes directly to any of therse files & it will be copied to the container.

Check more [configuration file examples] (https://wiki.squid-cache.org/ConfigExamples)

- **Using external configuration file**
To use external file, copy it config/ directory in this repository and it will be copied to the container.

The default Squid version is 4.17, but the following commands will help you run both versions concurrently on your local host.

- **Build Docker image**

**Squid4**

`sudo docker build --pull --rm -t squid4:latest "."`

**Squid5**

`sudo docker build --pull --rm -t squid5:latest --build-arg version=5 "."`

Since `-t` refers to tags, `squid4` & `squid5` will be reffered to as `{tag_name}` in the following commands.

- **Run Docker container on port forwarding**


`sudo docker run --name squid_proxy -it -d -p {host_port_number}:3128 {tag_name}`

Since `--name` refers to container name, make sure that you don't give the same name to more than 1 container, and that different versions are assigned different ports.
`squid_proxy` will be refrred to as `{container_name}` in the following commands.

- **Run Squid**

`sudo docker exec -d {container_name} squid`

- **Verify your container is running**

`sudo docker ps -a`

You should be able to see that the status is Up.

```
CONTAINER ID   IMAGE          COMMAND                  CREATED              STATUS              PORTS                                       NAMES
be0a7b18fef9   dsquid         "/docker-entrypoint.…"   22 seconds ago       Up 21 seconds       0.0.0.0:5001->3128/tcp, :::5001->3128/tcp   squid5_proxy
cd7bb12b2d59   3a61b71fe081   "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:5000->3128/tcp, :::5000->3128/tcp   squid4_proxy
```

- **Verify Squid is running on your local host**

`curl -I http://localhost:{host_port_number}`

You should be able to see the following response, including Squid's version.


```
HTTP/1.1 400 Bad Request
Server: squid/4.17
Mime-Version: 1.0
Date: Thu, 03 Feb 2022 14:04:27 GMT
Content-Type: text/html;charset=utf-8
Content-Length: 3509
X-Squid-Error: ERR_INVALID_URL 0
Vary: Accept-Language
Content-Language: en
X-Cache: MISS from cd7bb12b2d59
X-Cache-Lookup: NONE from cd7bb12b2d59:3128
Via: 1.1 cd7bb12b2d59 (squid/4.17)
Connection: close
```

- **Copy cert to import into client browser**

`sudo docker cp {container_name}:/etc/squid/cert/ca_cert.der .`

In your client browser, go to proxy settings > Manual proxy configuration > add `127.0.0.1` as the HTTP Proxy & port_number used when running the container.

[x] Also use this proxy for HTTPS.

- **Access container's terminal**

`sudo docker exec -it {container_id} /bin/ash`

For debug and error messages generated by Squid: 

`tail -f /var/log/squid/cache.log`

For key information about HTTP transactions [client IP address (or hostname), requested URI, response size, etc.]:

`tail -f /var/log/squid/access.log`

- **To use config file***

In container's terminal run
`squid -f /etc/squid/configFileName`
