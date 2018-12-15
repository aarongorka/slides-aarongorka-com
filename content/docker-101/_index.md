---
title: "Docker 101"
outputs: ["Reveal"]
featuredImage: "/docker-containerized-and-vm-transparent-bg.png"
reveal_hugo:
  custom_theme: "reveal-hugo/themes/robot-lung.css"
  margin: 0.2
---

# Docker 101

---

## Docker Concepts

---

### Containerisation

---

{{% figure src="/docker-containerized-and-vm-transparent-bg.png" attr="Image from Docker, Inc. via https://www.docker.com" attrlink="https://www.docker.com/resources/what-container" %}}

{{% note %}}
Containers are a much more lightweight version of VMs
Shared kernel
Can pack more containers in to a host
{{% /note %}}

---

{{% figure src="/container_evolution.jpg" attr="Image from https://developer.ibm.com" attrlink="https://developer.ibm.com/code/2017/07/25/oci-reaches-major-milestone/" %}}

{{% note %}}
Containers have been around for a long time
Very mature
Containers are nothing new, so why Docker?
{{% /note %}}

---

### Developer Experience

{{% note %}}
  * Infrastructure as Code
  * Artifact store
  * Portability
  * Consistency
  * Layer caching
{{% /note %}}

---

{{% figure src="/docker.jpg" attr="Image from https://cultivatehq.com" attrlink="https://cultivatehq.com/posts/docker/" %}}

{{% note %}}
Opinionated workflow
{{% /note %}}

---

### Not Just For Servers

{{% note %}}
Build, test and deploy

  * Version pinning
  * Manage your own dependencies without Jenkins
  * Consistency
{{% /note %}}

---

## Using Docker

---

Windows:

  * [Link to skip signing up](https://download.docker.com/win/stable/Docker%20for%20Windows%20Installer.exe)

Mac:

  * https://docs.docker.com/install/#releases
  * `pip install --user docker-compose`
  * `brew cask install minikube`

{{% note %}}
Avoid logging in to Docker using Docker Hub credentials
{{% /note %}}

---

Docker command cheatsheet:

  * `docker run -it <image>`
  * `docker build -t <image> .`
  * `docker push <image>`
  * `docker pull <image>`

---

## Dockerfile

---

```Dockerfile
FROM centos:7
COPY target/app.jar /srv/app/app.jar
RUN yum install -y openjdk8
ENTRYPOINT ["java", "-jar"]
CMD "/srv/app/app.jar"
```

---

## Docker Features

---

  * Access files inside container: `docker run -v c:/Users/aarongorka/.ssh:/root/.ssh ubuntu:latest`
  * {{% fragment %}}Open a port on your workstation: `docker run -p 8080:8080 nginx:latest`{{% /fragment %}}
  * {{% fragment %}}Pass in environment variables: `docker run -e DB_USERNAME=foobar drupal`{{% /fragment %}}
  * {{% fragment %}}Run in interactive mode: `docker run -it ubuntu bash`{{% /fragment %}}

{{% note %}}
Docker's isolation is about resource isolation, not security
You must explicitly pass in any external resources
{{% /note %}}

---

Browse [Docker Hub][]:

![Docker Hub screenshot](/dockerhub_screenshot.png)

[Docker Hub]: https://hub.docker.com/

---

# Questions about Docker so far?

---

# docker-compose

---

`docker run -it --rm -v "$(pwd):/srv/app:Z" -v "${HOME}/.ssh:/root/.ssh:Z" -v "${HOME}/.aws:/root/.aws:Z" -e MY_ENV_VAR -e MY_ENV_VAR_2 -w /srv/app ubuntu:3.5.8 deploy.sh`

{{% fragment %}}
vs.

`docker-compose run deploy`
{{% /fragment %}}

---

Example docker-compose.yml:

```yaml
version: '2.0'
services:
  kubectl:
    image: aarongorka/kubernetes-utils:1.8.4
    volumes:
      - .:/srv/app:Z
      - ~/.kube:/root/.kube:Z
    working_dir: /srv/app

  builder:
    image: aarongorka/java-base-builder:latest
    volumes:
      - .:/srv/app:Z
      - ~/.m2:/root/.m2:Z
    working_dir: /srv/app
```

---

# Make

---

```bash
docker login nexus.example.com -u "${NEXUS_USERNAME}" -p "${NEXUS_PASSWORD}"
docker-compose run --rm builder mvn install -B
docker build -t ${IMAGE_NAME}:${BUILD_VERSION} .
docker push ${IMAGE_NAME}:latest
docker-compose run --rm builder sh -c "envsubst < k8s-config/deployment.tmpl.yml > k8s-config/deployment.yml"
docker-compose run --rm kubectl apply -f k8s-config/deployment.yml
```

{{% fragment %}}
vs.

```bash
make nexusLogin build dockerBuild dockerPush deploy
```
{{% /fragment %}}

{{% note %}}
Make is a runner for multiple Docker/docker-compose commands
OS agnostic
{{% /note %}}

---

Example Makefile:

```Makefile
build:
	docker-compose run --rm builder mvn install -s settings.xml

dockerBuild:
	docker build -t $(IMAGE_NAME):$(BUILD_VERSION) .

nexusLogin:
	docker login nexus.example.com -u "$(NEXUS_USERNAME)" -p "$(NEXUS_PASSWORD)"

deploy:
	docker-compose run --rm --entrypoint=sh builder -c "envsubst < k8s-config/deployment.tmpl.yml > k8s-config/deployment.yml"
	docker-compose run --rm kubectl apply -f k8s-config/deployment.yml
```

---

##### Docker + docker-compose + Make

{{% fragment %}}### ={{% /fragment %}}

{{% fragment %}}# 3 Musketeers{{% /fragment %}}

{{% fragment %}}https://3musketeers.io{{% /fragment %}}

---

# Running Docker in Production

---

  * Deployment?
  * Secrets?
  * Configuration?
  * Scheduling?
  * Networking?

{{% note %}}
Logging in to servers and starting containers
Restarting failed containers
Pulling images
Managing n number of containers
{{% /note %}}

---

{{% figure src="/container_disaster.jpg" %}}

---

>The purpose of Kubernetes is to make it easier to organize and schedule your application across a fleet of machines. At a high level it is an operating system for your cluster.

---

_To be continued..._
