
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
