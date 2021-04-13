
[Refrence-packt-DockerDCA](https://github.com/PacktPublishing/Docker-Certified-Associate-DCA-Exam-Guide)


Customizing docker - key.json,daemon.json, Environment variables(/etc/systemd/system/docker.service.d/http-proxy.conf)
```sh
sudo cat /etc/docker/key.json |jq
sudo cat /etc/docker/daemon.json
%programdata%\docker\config\daemon.json #windows 

```
```
sudo systemctl status docker
sudo systemctl enable docker
sudo systemctl start docker
```
>Docker client custom to use proxy = $HOME/.docker/config.json on Linux or %USERPROFILE%/.docker/config.json on
Windows)

>Docker Client server sercurity =  socket uses unix,tcp and others (/var/run/docker.sock) daemon will listen on local unix socket
by default listens tcp port 2375 for the daemon communication TLS-based

```
docker system info --format '{{json .}}'
docker system info --format '{{json .SecurityOptions}}'
```
> add docker user to the docker group if you tend to get permission error , current user $USER will be added to docker group
```
sudo usermod -a -G docker $USER
newgrp docker
```

> run alpine image to check out the docker process,find containerd,dockerd, runtime-run, -namespace, find the hash value replace
> it after the exec 5cc30
```
docker run -d nginx:alpine
ps -efa|grep -v grep|egrep -e containerd -e nginx
docker container exec 5cc39 ps -ef
```

Docker seccomp capability management, we can see that 1st command will run as root, lets drop all the capability on 2nd command 
expected fails due to the chown issue, lets use CHOWN on the 3rd so that we can access it.
```
docker container run --rm -it alpine sh -c "chown nobody /etc/passwd; ls -l /etc/passwd"
docker container run --rm -it --capdrop=ALL alpine sh -c "chown nobody /etc/passwd; ls -l /etc/passwd"
docker container run --rm -it --capdrop=ALL --cap-add CHOWN alpine sh -c "chown nobody /etc/passwd; ls -l /etc/passwd"
```
```
docker container ls
docker ps -a
docker container ls --all
docker run -d nginx:alpine
docker images ls
`````

###### Creating a docker file , save this content as dockerfile
dockerfile created will  have same algorithm:hexadecimal_code_using_algorithm format which is created based on the content 
and when rebuild using the same docker content with different name you could find the imageID to be same.
```
FROM ubuntu:18.04
RUN apt-get update -qq && apt-get install -qq package1 package2
COPY . /myapp
RUN make /myapp
CMD python /myapp/app.py
```
CoW copy on write - uses 3 methods.
>AUFS and overlay-based drivers, Docker uses union filesystems
>BTFS and ZFS drivers, Docker uses filesystem snapshots
>device-mapper, Docker uses an LVM snapshot for blocks
it mainly focuses on reducing the storage consuption and making it maximum efficient, providing a mechanism to stack the storage layer 
based on the contaienrs
> this command will returns size of read-only data used from the original image , this will not include logs or volumes aligned 
```
docker container ls --size
```
#### Buiding docker image ref, using code block for specialcases.
```
FROM <image>[:tag] or FROM <image>[@digest]
>ARG <variable>=<value>
>LABEL( LABEL version="1.0",LABEL description="HELLO", LABEL maintainer=" omkar b <omkar219@gmail.com>",environment="preproduction")
>ENV (ENV DATABASE=TEST)
>WORKDIR(sets to current director as wworking director ie: WORKDIR /myworkspace)
>RUN(can be used to run sh commands ie: apt-get update -qq && apt-get upgrade)
>COPY(copying file from local to container ie: COPY myapp/* /myapp
>ADD(can be used to add url,tar,otherfiles ie:ADD http://example.com/bigpackagefile.tar.gz /myapp)
>USER(Creates user ie: USER www-data:www-data) its best to run non-root user to run process in production.
>VOLUME(creates a mount point for the respect directory ie: VOLUME /mydata)
>EXPOSE(publishes the port making docker daemon to listentoport ie: EXPOSE 80/tcp)
>CMD(defines default process and overrides on ENTRYPOINT even if it's defined ie: CMD /usr/bin/curl -I https://www.example.com)
>ENTRYPOINT(will set directive container will run as executable to accept arguments ie: ENTRYPOINT [app.py] )  ["executable", "argument1", "argument2"]
>HEALTHCHECK(monitoring processes inside container, by default it will monitor main process 
HEALTHCHECK \
--interval=DURATION (default: 30s) \
--timeout=DURATION (default: 30s) \
--start-period=DURATION (default: 0s) \
--retries=N (default: 3) \
CMD /bin/myverificationscript
```

##### DOCKER process actions
```
ls, prune, rm, and tag #for managing docker
history,inspect and info # for information
pull, push, load, import, and save # to share btn hosts
build # used to create image by using file or zip or content of text 
docker image build [options] <context>
--add-host
--build-arg constraint:ostype==linux o
--file or -f 
--force-rm # keep env clean, intermediate containers will be removed after cleaning
--isolation # is mandate when building in windows
--label # add meta-information in key-value pair fmt
--no-cache # by default dockerd uses host cached layers 
--tag or -t # dockerrepo.io/newimage:version1 
--target we can have multiple build stages
--cpu-quota, --cpu-shares, and --memory # for resource management to set containers threshold
docker image build [-t MY_TAG] [--label MY_LABEL=VALUE] [--file MY_DOCKERFILE] [BUILD_CONTEXT]
docker build --file Dockerfile.application -t templated:production --build-arg ENVIRONMENT=production .
docker image ls --filter label=environment
docker image ls --filter label=environment=test
docker image ls --filter label=environment=production
docker image inspect myapp:1.0 --format "{{ index .Config.Labels }}" map[environment:production]
docker image prune --force --filter label=environment=test
docker container ls --filter ancestor=<image_to_be_removed>
docker system df --verbose
docker images --filter=reference='alpine:latest'
docker image rm cc0abc535e36 --force
```
##### DOCKER repo, where are images stored?
/var/lib/docker/image in Linux and c:\programdata\docker\image
create a local registry to store images , can be configured 
>/etc/docker/registry/config.yml

there are local, root cloud based repo (dockerhub.io), then there user based or Org based images, full registry format 
> example for full registry fmt dtr.myorganization.com[:my_registry_port][/myteam or/myoraganization][/myusername]/<repository>[:tag].

> example  docker.io/omkar219/alpine-nginx:1.13
 
```
#listing  images in a table format 
docker image ls --format "table {{.ID}}\\t{{.Repository}}:{{.Tag}}\\t{{.CreatedAt}}"
```

```
we can download all the tags (not recommended)
docker image pull --all-tags --quiet
```
Docker requires login if you're using private repo , docker login from ECR,ACR,GCR all requires a login where information can be found
once you have created a container images repo in these clouds.

Better practise when using docker images, we will save that -o (output) as a tar and use it later on, this experiment shows about expose ports 
```
docker image save docker.io/codegazers/colors:test -o /tmp/codegazers_colors_test.tar
file /tmp/codegazers_colors_test.tar /tmp/codegazers_colors_test.tar: POSIX tar archive
tar -tf /tmp/codegazers_colors_test.tar
docker import /tmp/codegazers_colors_test.tar
docker inspect codegazers/colors:test --format '{{json .Config.ExposedPorts }}'
docker inspect 5bd30fec31de --format '{{json .Config.ExposedPorts }}'
docker image rm codegazers/colors:test
docker image load </tmp/codegazers_colors_test.tar
docker inspect codegazers/colors:test --format '{{json .Config.ExposedPorts }}'
```


##### DOCKER multi stage build
having 2 alpine images build out, transfering files from one to another container. Using keyword "AS" will set 
```
FROM alpine AS sdk
RUN apk update && \
apk add --update --no-cache alpine-sdk
RUN mkdir /myapp
WORKDIR /myapp
ADD hello.c /myapp
RUN mkdir bin
RUN gcc -Wall hello.c -o bin/hello
FROM alpine
COPY --from=sdk /myapp/bin/hello /myapp/hello
CMD /myapp/hello
```
##### DOCKER VOLUME echo> wont work in dockerfile test
```
FROM alpine
RUN mkdir /data
VOLUME /data
RUN echo "hello world" > /data/helloworld
```
the above will faile to write bcz volume will bypass COW, containers will not maintain volume content because the commit action will
just transform container content into images, and volumes are not found inside containers.
##### running docker images on local registry 
the image's data will be stored on the host under the
```
docker container run -d -p 5000:5000 --restart=always --name registry registry:2
docker pull alpine
docker tag alpine localhost:5000/my-alpine
docker image push localhost:5000/my-alpine
docker images --filter=reference='alpine:latest'
docker image rm <imageid> --force
docker pull localhost:5000/my-alpine
docker container run localhost:5000/myalpine:latest ls /tmp 
docker 
```
>/var/lib/docker/volumes/REGISTRY_DATA/_data

REGISTRY_DATA named volume, so the registry data will remain even if we remove the registry container.

##### DOCKER image templating 
in this we will have development and production templates
```
Development – Dockerfile.nginx-dev:
FROM nginx:alpine
RUN apk update -q
RUN apk add \
curl \
httpie

Production – Dockerfile.nginx:
FROM nginx:alpine
RUN apk update -q


docker image build --file Dockerfile.nginx-dev -t baseimage:development --label lab=lab4 --quiet .
docker image build --quiet --file Dockerfile.nginx -t baseimage:production --label lab=lab4 .
#lets compare the size of dev and prod
docker image ls --filter label=lab=lab4
#create a Dockerfile.application file
ARG ENVIRONMENT=development
FROM baseimage:${ENVIRONMENT}
COPY html/* /usr/share/nginx/html

### create a html folder to copy htmlfile to the container
mkdir html
echo "This is a simple test and of course it is not an application!!!" > html/index.html
### we just compile both images for the DEV and PROD environments. we use the ENVIRONMENT argument to run development or production if needed
docker image build --file Dockerfile.application -t templated:development --build-arg ENVIRONMENT=development --label lab=lab4 .
docker image build --file Dockerfile.application -t templated:production --build-arg ENVIRONMENT=production --label lab=lab4 .
```

##### DOCKER host resources 
--cpuset-cpus can be used for how many cpu to use.
--cpu-quota limiter for cpu
--cpu-shares cpu cycles will be shared but the default value would be 1024
-cpus = 1.5 will guarantee half of the cpu resources to the container
--memory , we can disable oom-killer using --oom-kill-disable 
--memory-reservation : threshold 
```
docker stats --all --format "table [{{.Container}}] {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
docker container top webserver
```
>Converting container into images

```
commit
export --output or -o for output file 
docker container diff webserver
```
>Formatting and filtering 

--no-trunc to disable the truncation, --format='{{json .}} , 
```
'{{json .Mounts}}' or '{{split .Image ":"}}'
'{{title .Name}}' || can use lower or upper or title
'{{range <JSON keys> }}{{end}}' define range 
docker container ls --all --format "table {{.Names}}: {{.Image}} {{.Command}}" --no-trunc
docker container ls --all --format='{{json .}}')
```
##### Manging devices 
You can give sound to the dockercontainer 
```
docker run -ti --cap-add SYS_ADMIN --device /dev/mapper/centosroot:/dev/sdx centos
docker container run -ti --device /dev/snd alpine
docker container --help

```
##### Executing containers

we are renaming docker, if we forgot to give it a name 
```
docker container run -ti -d alpine
docker container rename $(docker container ls -ql) myalpineshell\
docker container attach myalpineshell ## will attach terminal to running shell
docker container ls --all --filter name=myalpineshell --filter name=secondshell
docker container start -a -i myalpineshell
docker container exec -ti myalpineshell sh
docker container ls --all --filter name=myalpineshell

```
#####  limiting container resources 
```
docker container stats ## below image is for stress testing only dont use it for fun
docker container run --memoryreservation=250m --name 2GBreserved -d frjaraur/stress-ng:alpine --vm 2 -vm-bytes 1024M
docker container rm -f 2GBlimited ## to remove limited
docker container run -d --cpus=1 --name CPU2vs1 frjaraur/stress-ng:alpine --cpu 2 --timeout 120 ## CPU
```

##### formating and filtering container list output 

```
docker run -d --name web1 --label stage=production nginx:alpine
docker run -d --name web2 --label stage=development nginx:alpine
docker run -d --name web3 --label stage=development nginx:alpine
docker container ls --format "table {{.Names}} {{.Command}}\\t{{.Labels}}" ## labels will help u find which container belongs to stages 
docker container ls --format "table {{.Names}} {{.Command}}\\t{{.Labels}}" --filter label=stage=development
docker container kill $(docker container ls --format "{{.ID}}" --filter label=stage=development)
docker container ls --format "table {{.Names}}\\t{{.Labels}}"
```

#### Docker container networking 

```
FROM alpine:3.10
RUN set -ex; \
postgresHome="$(getent passwd postgres)"; \
postgresHome="$(echo "$postgresHome" | cut -d: -f6)"; \
[ "$postgresHome" = '/var/lib/postgresql' ]; \
mkdir -p "$postgresHome"; \
chown -R postgres:postgres "$postgresHome"
...
...
RUN mkdir -p /var/run/postgresql && chown -R postgres:postgres /var/run/postgresql && chmod 2777 /var/run/postgresql
ENV PGDATA /var/lib/postgresql/data
RUN mkdir -p "$PGDATA" && chown -R postgres:postgres "$PGDATA" && chmod 777 "$PGDATA"
VOLUME /var/lib/postgresql/data
COPY docker-entrypoint.sh /usr/local/bin/
ENTRYPOINT ["docker-entrypoint.sh"]
EXPOSE 5432
CMD ["postgres"]
```
There are many types of volume storage:
Unamed volumes - its hard to track them on local as they are unnamed, this will grow in docker data root path. 
Named valued - the volumes can be used with volume set to a driver or NFS for example.
Localhost directories or files - any special file or directories in your local be set too.

```
create - can create a volume (docker volume create or --opt or -o to add options for drivers)
docker volume inspect 
docker volume ls
docker volume prune
docker volume rm 
docker image inspect postgres:alpine --format "{{ .Config.Volumes }} " map[/var/lib/postgresql/data:{}]
docker container run -d --name mydb postgres:alpine

### this is the directory volume which we can bypass it to by pass CoW, making it to allow R/W/Modify content in the /var/lib/postgresql/data direcotry.

docker container inspect mydb --format "{{ .Mounts }} " [{volume c888a831d6819aea6c6b4474f53b7d6c60e085efaa30d17db60334522281d76f /var/lib/docker/volumes/c888a831d6819aea6c6b4474f53b7d6c60e085efaa30d17db60334522281d76f/_data/var/lib/postgresql/data local true }]

docker volume inspect c888a831d6819aea6c6b4474f53b7d6c60e085efaa30d17db60334522281d76f 
## this will show us that it has been mounted, and we can access the data locally.
docker volume ls --filter name=c888a831d6819aea6c6b4474f53b7d6c60e085efaa30d17db60334522281d76f
## lets remove and align this to a new container -- attaching the volume
ocker container stop mydb
docker container rm mydb
docker volume rm c888a831d6819aea6c6b4474f53b7d6c60e085efaa30d17db60334522281d76f
### creating new my data to map it to volume : dir or file 
docker volume create mydata
docker container run --name c1 -v mydata:/data -ti alpine
docker container run --name c2 -v mydata:/tmp -ti alpine ls -lart /tmp
docker volume rm mydata
docker container rm c1 c2
docker volume rm mydata

```


##### Networking in containers 
```
docker network ls 
> --attachable : option enables container attachement 
> --aux-address : add host and its address to this n\w ie: --auxaddress="mygateway=192.168.1.10"
> --config-from and --config-only : useful for building configurations using automation tools
> --driver or -d and --opt : By default, we can only use macvlan, none, host, and bridge.
> --gateway : overwrite the default gateway.
> --ingress : internal service management.
> --internal : on overlay networks, will be attached to the docker_gwbridge bridge network.
> --ip-range : range of IP addresses for containers
> --ipam-driver : external IP address management
> --ipv6 : We will use this option to enable IPv6 on this network.
> --label :  we can add metadata information to networks for better filtering
> --scope : scope of operation is it in local or swarm
> --subnet : CIDR format that represents a network segment.
Docker manipulates the iptables rules for you every time a network is created or some connection.
recommend allowing the Docker daemon to manage these rules for you.
```

##### default bridge network

bridge is the default network type for all containers.
others can be declared on container creation or exection by using --network.

```
docker container run -ti -d --name c1 alpine ping 8.8.8.8
docker container run -ti -d --name c2 alpine ping 8.8.8.8
docker container inspect c1 --format "{{ .NetworkSettings.Networks.bridge.IPAddress }}"
docker container inspect c2 --format "{{ .NetworkSettings.Networks.bridge.IPAddress }}"
docker container inspect c1 --format "{{json .NetworkSettings.Networks }}"
## when u get the output by default it would be using bridge not ipv6
## containers created subnet 172.17.0.0/16 all containers will get IP from this segement 
## gateway default would be 172.17.0.1 
## containers will have virtual mac addresses , ip address 


``` 
##### Null networks
```
docker run -ti --network none alpine 
ip add
## c
reating null will only have access to its required resources, by default bridge n/w specifies none

#####Host network

docker run -ti --network host alpine 
## container is using namespace called host which can be directly talk to local interface which is risky you can create your own custom bridge 
> custom bridge: - isolation : with iptables
> internal DNS : --link functionality 
> On-the-fly container attachment: network bridge or custom can be attached even if we create a null network container.
### Creating a custom network for understanding 
docker network create --driver bridge --internal --subnet 192.168.30.0/24 --label internal-only internal-only
docker network inspect internal-only
docker container run --network internal-only -ti --name intc1 alpine sh
ping intc1 -c2
docker container run --network internal-only -ti --name intc2 alpine sh

## run this is local docker machine iptables -L to see the docker rules
```

##### Network / external world
--ip-forward=false : Ip forwarding for containers can be done at kernal level. 
iptables : will run rules , instead of allowing dockerd to take care. --iptables=false 
DOCKER and DOCKER-ISOLATION , DOCKER-USER chain. 

##### inter-container communication 
this can be done b inter-container communication with ip forwarding and iptables , --internal network creation. --icc=false inter-container communication is off. most secure network connection can be done by --link . 

##### DNS on custom bridge networks
 custom bridge allows internal DNS to communicate. 127.0.0.11. which can be modified.
 ```
 --network-alias=ALIAS : add name to a container internal DNS
 --link=CONTAINER_NAME:ALIAS  : This use case is different to --network-alias,internal DNS to allow  the resolution of CONTAINER_NAME
--dns, --dns-search, and --dns-option : manage forwarded DNS resolution,can add a forwarder DNS
#### publishing applications = exposing imags 
docker container run -d --name webserver nginx:alpine
docker container inspect webserver --format "{{json .NetworkSettings.Networks.bridge.IPAddress }}"
#172.17.0.1 in this case, and we can reach container port 80
--publish or -p  [HOST_IP:][HOST_PORT:]CONTAINER_PORT[/PROTOCOL] TCP
declare a range of ports in the form --publish StartPort-EndPort[/PROTOCOL]

docker container run -d --name public-webserver --publish 80 nginx:alpine
docker container ls --filter name=public-webserver
### requirements.txt should contain python
### Excerise 
FROM python:alpine
WORKDIR /app
COPY ./requirements.txt requirements.txt
RUN pip install -r requirements.txt
COPY app.py .
COPY templates templates
EXPOSE 5000
CMD ["python", "app.py"]


docker image build -q -t simpleapp:v1.0
docker container run -d --name v1.0 simpleapp:v1.0
docker container ls --filter name=v1.0
docker container inspect v1.0 --format "{{.NetworkSettings.Networks.bridge.IPAddress }}"
### binding it with volume so we could modify html file 
docker container run -d --name v1.0-bindmount -v $(pwd)/templates:/app/templates simpleapp:v1.0
docker container inspect v1.0-bindmount --format "{{.NetworkSettings.Networks.bridge.IPAddress }}"


#####Mounting SSHFS PLUGIN
docker plugin install vieux/sshfs
#plugin used for sshfileshare
# just give enter and Y
sudo systemctl status ssh
docker plugin ls
docker volume create -d vieux/sshfs -o sshcmd=ssh_user@127.0.0.1:/tmp -o password=ssh_userpasswd \ sshvolume
docker container run --rm -it -v sshvolume:/data alpine sh

``` 
#### multi homed containers 
```
docker network create zone-a
docker network create zone-b
docker container run -d --name cont1 --network zone-a alpine sleep 3000
docker network connect zone-b cont1
docker container run -d --name cont1 --network zone-b alpine sleep 3000
docker exec cont1 ip add
##shows subnets of zoneb
docker container run -d --name cont2 --network zone-b --cap-add NET_ADMIN alpine sleep 3000
we can run 2 containers with 1 interface 
docker container run -d --name cont3 --network zone-a --cap-add NET_ADMIN alpine sleep 3000
docker exec cont2 ip route
## 172.20.0.0/16 dev eth0 scope link src 172.20.0.3
docker exec cont3 ip route
## 172.19.0.0/16 dev eth0 scope link src 172.19.0.3
## if we want cont2 - cont 3 to communicate we have to add route via cont1 
docker exec cont2 route add -net 172.19.0.0
docker exec cont2 ip route
docker exec cont3 route add -net 172.20.0.0
##172.19.0.0/24 via 172.20.0.2 dev eth0
##172.20.0.0/16 dev eth0 scope link src 172.20.0.3
docker exec cont3 ip route
##172.19.0.0/16 dev eth0 scope link src 172.19.0.3
##172.20.0.0/24 via 172.19.0.2 dev eth0

docker exec cont3 ping -c 3 cont2
##FAILS bcz of name resolution fail foor cont2


docker exec cont3 ping -c 3 cont1
#PING cont1 (172.19.0.2): 56 data bytes
docker exec cont3 ping -c 3 172.20.0.3


```
##### Publishing applications 
creating a simple 3 layer application , where 2 layer application of load balancer.
```
docker network create simplenet
docker container run -d  --name simpledb --network simplenet --env "POSTGRES_PASSWORD=changeme" 
codegazers/simplestlab:simpledb
docker container run -d \
--name simpleapp --network simplenet --env dbhost=simpledb --env dbname=demo \
--env dbuser=demo --env dbpasswd=d3m0 codegazers/simplestlab:simpleapp

docker network inspect simplenet --format "{{range .Containers}} {{.IPv4Address }} {{.Name}} {{end}}"
docker inspect codegazers/simplestlab:simpledb --format "{{json .Config.ExposedPorts }}"
docker inspect codegazers/simplestlab:simpleapp --format "{{json .Config.ExposedPorts }}"

## Creating loadbalance after db and app 
docker container run -d \
--name simplelb --env APPLICATION_ALIAS=simpleapp --env APPLICATION_PORT=3000 \
--network simplenet --publish 8080:80 codegazers/simplestlab:simplelb
## run local iptables to check 
sudo iptables -L DOCKER -t nat --linenumbers --numeric


##### Using Docker Compose to deploy multiple container 









