---
title: "3 Musketeers"
date: 2020-11-18T09:07:50+11:00
outputs: ["Reveal"]
reveal_hugo:
  custom_theme: "reveal-hugo/themes/robot-lung.css"
  margin: 0.2
---

<style>
.reveal section img {
    border: none;
    box-shadow: none;
}
</style>

# 3 Musketeers

---

{{% slide background-image="/3musketeers.png" background-size="50%" %}}

---

{{< table_of_contents >}} 

---

## What is it, what is it not?

  * 3 Musketeers is a pattern
  * Not a product
  * Technology agnostic

---

## What problems does it solve?

  * Magic pipelines
  * Snowflake CI/CD agents
  * Impossible to understand build/deploy scripts

---

## How does it help?

  * Well-documented build/deploy environments - as code
  * Empower developers to maintain dependencies
  * Parity between local and CI/CD environments
  * Consistency in the commands that run your build/deploy

---

## Containerising your CI/CD environment

---

  * Run _everything_ inside a container in your CI/CD pipeline
  * Use **bind mounting** to enable "tool containers"
  * Tool containers are built in separate pipelines and versioned with semver, stored in container registry

---

Bind mount example:

```
docker run -it --rm -v "$(pwd):/work" "${HOME}/.kube:/root/.kube" -w /work aarongorka/kubernetes-utils:1.8.4 kubectl apply -f k8s-config/deployment.yml
```

---

## Orchestrating your containers

---

`docker run -it --rm -v "$(pwd):/work:Z" -v "${HOME}/.ssh:/root/.ssh:Z" -v "${HOME}/.aws:/root/.aws:Z" -e MY_ENV_VAR -e MY_ENV_VAR_2 -w /work ubuntu:3.5.8 deploy.sh`

{{% fragment %}}
vs.

`docker-compose run deploy`
{{% /fragment %}}

---

`docker-compose.yml`:
```yaml
version: '2.0'
services:
  kubectl:
    image: aarongorka/kubernetes-utils:1.8.4
    volumes:
      - .:/work:Z
      - ~/.kube:/root/.kube:Z
    working_dir: /work

  builder:
    image: aarongorka/java-base-builder:latest
    volumes:
      - .:/work:Z
      - ~/.m2:/root/.m2:Z
    working_dir: /work
```

---

## Using a task runner

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

## Environment variables

---

`docker-compose.yml`:
```yaml
version: '2.0'
services:
  kubectl:
    image: aarongorka/kubernetes-utils:1.8.4
    env_file: .env  # <------- here
    volumes:
      - .:/work:Z
      - ~/.kube:/root/.kube:Z
    working_dir: /work
```

`.env`:
```
PULUMI_STACK
AWS_PROFILE
AWS_DEFAULT_REGION
```

---

`envvars.yml`:
```
envvars:
  - name: AWS_DEFAULT_REGION
    desc: AWS Region
    optional: true
    tags:
      - aws
  - name: AWS_PROFILE
    desc: The profile to use for Pulumi state access
    tags:
      - pulumi
      - aws
  - name: PULUMI_STACK
    desc: The pulumi stack to select
    tags:
      - pulumi
```

---

`Makefile`:
```Makefile
up: .env
	docker-compose run --rm envvars ensure --tags pulumi,aws
    # pulumi commands go here...

.env:
	touch .env
	docker-compose run --rm envvars validate
	docker-compose run --rm envvars envfile --overwrite
```

---

## Docker + docker-compose + Make

{{% fragment %}}={{% /fragment %}}

{{% fragment %}}3 Musketeers{{% /fragment %}}

{{% fragment %}}https://3musketeers.io{{% /fragment %}}

---

## Alternative task runners

https://taskfile.dev/#/

`Taskfile.yml`:
```
version: '2'
tasks:
  deploy:
    cmds:
      - docker-compose run --rm serverless npx --no-install serverless
  deps:
    cmds:
      - docker-compose run --rm serverless npm i
```

---

##  Conclusion

  * Consistent build environments
  * Control over dependencies for build/deploying applications
  * Confidence that local testing has parity with pipeline testing
