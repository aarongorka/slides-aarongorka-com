---
title: "deploying to amaysim ECS"
date: 2018-08-14T13:14:32+10:00
outputs: ["Reveal"]
featuredImage: "/media/deploying-to-amaysim-ecs-preview.png"
#reveal_hugo:
#  transition: "convex"
---

<style>
.reveal {
  font-family: MarkOT,sans-serif;
  letter-spacing: -0.4px
}

.reveal h1,
.reveal h2,
.reveal h3,
.reveal h4,
.reveal h5,
.reveal h6 {
  font-family: MarkOT,sans-serif;
  letter-spacing: -0.6px;
  text-transform: none;
}

.reveal a {
  color: #FF5500;
  text-decoration: underline;
}

.reveal section.orange a {
  color: #680991;
  text-decoration: underline;
}
</style>

{{% slide background-image="/media/W7I1OB1A.png" class="orange" %}}

# deploying to amaysim ECS

{{% /slide %}}

---

{{% slide background-image="/media/W7I1OB1A.png" class="orange" %}}

  * Introduction to amaysim's ECS platform
  * Setting up a project with ecs-utils
  * Common troubleshooting steps

{{% /slide %}}

---

{{% slide background-image="/media/JSbQ6paw.png" %}}

## why ECS?

  * Overcome shortcomings of Rancher
  * Easy to manage compared to Kubernetes
  * Multi-tenanting
  * Very cost-efficient with spot instances
  * Container autoscaling
  * IAM roles for containers

{{% note %}}
Rancher issues:

  * 503s
  * manual deploys
  * inability to upgrade
  * support issues
  * lack of APIs
  * no container autoscaling
{{% /note %}}

{{% /slide %}}

---

{{% slide background-image="/media/JSbQ6paw.png" %}}

## goals of our ECS platform

  * Fully automated, zero downtime blue/green deployments
  * Multi-tenanted Docker platform
  * Container autoscaling
  * Spot Fleet
  * Flexible but opinionated architecture

{{% /slide %}}

---

{{% slide background-image="/media/JSbQ6paw.png" %}}

# architecture overview

{{% /slide %}}

---

{{% slide background-image="/media/JSbQ6paw.png" %}}

![amaysim ECS Overview](/media/amaysim ECS Overview.svg)

{{% /slide %}}

---

{{% slide background-image="/media/JSbQ6paw.png" %}}

![amaysim ECS Overview](/media/amaysim ECS Overview V2.svg)

{{% /slide %}}

---

{{% slide background-image="/media/JSbQ6paw.png" %}}

## autoscaling

{{% figure src="/media/ecs_schedule_container.png" attr="Image by Philipp Garbe via https://garbe.io" attrlink="https://garbe.io/blog/2017/04/12/a-better-solution-to-ecs-autoscaling/" %}}

{{% note %}}
Explain two types of scaling, instance and container
{{% /note %}}

{{% /slide%}}

---

{{% slide background-image="/media/JSbQ6paw.png" %}}

  * Autoscaling using [Target Tracking] on _reservation_ metrics
  * All traffic goes through shared CloudFront
  * We use Application Load Balancer (ALB), not Classic ELB
  * Multiple instance types via Spot Fleet

[Target Tracking]: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-autoscaling-targettracking.html

{{% /slide %}}

---

{{% slide background-image="/media/W7I1OB1A.png" class="orange" %}}

# prerequisite: the application stack

{{% /slide %}}

---

{{% slide background-image="/media/W7I1OB1A.png" class="orange" %}}

![Application Stack](/media/Application Stack.svg)

{{% /slide %}}

---

{{% slide background-image="/media/W7I1OB1A.png" class="orange" %}}

[docker-ecs-utils/ecs-cluster-application.yml](https://github.com/aarongorka/docker-ecs-utils/blob/master/ecs-cluster-application.yml)

![application template in Github](/media/application_template_github2.png)

{{% /slide %}}

---

{{% slide background-image="/media/W7I1OB1A.png" class="orange" %}}

Continuous Deployment using Stacker:

```yaml
  - name: ECS-Dev-App-Example
    template_path: ecs-cluster-application.yml
    region: ap-southeast-2
    profile: nonprod
    requires:
      - ECS-Dev
    variables:
      Name: Example
      Environment: Dev
      ClusterName: Dev
      HostedZoneName: dev-apps.amaysim.net.
      LBType: ALB
      Scheme: External
      SSLCertificateARN: arn:aws:acm:ap-southeast-2:999999999999:certificate/4013c1bc-a532-4fcc-9f90-123456789876  # *.dev-apps.amaysim.net
      Subnets: subnet-12345678,subnet-22345678,subnet-3456789e  # Dev Public x
      VpcId: vpc-abc234ad
```

{{% /slide %}}

---

{{% slide background-image="/media/JSbQ6paw.png" %}}

# setting up a project with ecs-utils

{{% /slide %}}

---

{{% slide background-image="/media/JSbQ6paw.png" %}}

[![ecs-utils repo](/media/Screen Shot 2018-08-16 at 10.41.50-fullpage.png)](https://github.com/amaysim-au/docker-ecs-utils)

{{% /slide %}}

---

{{% slide background-image="/media/JSbQ6paw.png" %}}

  1. Add [ecs.json] and [ecs-config.yml]
  1. Add `deploy`, `cutover` and `autocleanup` targets to Makefile
  1. Add `amaysim/ecs-utils` to docker-compose.yml
  1. Update .env and .env.template with environment variables

[ecs.json]: https://github.com/amaysim-au/docker-ecs-utils/blob/master/example/deployment/ecs.json
[ecs-config.yml]: https://github.com/amaysim-au/docker-ecs-utils/blob/master/example/deployment/ecs-config.yml

{{% /slide %}}

---

{{% slide background-image="/media/JSbQ6paw.png" %}}

## ecs.json

```json
{
    "containerDefinitions": [
        {
            "essential": true,
            "image": "123456789987.dkr.ecr.ap-southeast-2.amazonaws.com/devops/ok:${BUILD_VERSION}",
            "name": "${ECS_APP_NAME}",
            "linuxParameters": {
                "initProcessEnabled": true
            },
            "portMappings": [
                {
                    "containerPort": 8888
                }
            ],
            "logConfiguration": {
                "logDriver": "awslogs",
                "options": {
                    "awslogs-group": "ecs-${ECS_APP_NAME}-${ENV}",
                    "awslogs-region": "ap-southeast-2",
                    "awslogs-stream-prefix": "${BUILD_VERSION}"
                }
            }
        }
    ],
    "family": "${ECS_APP_NAME}-${ENV}",
    "volumes": [],
    "memory": "128",
    "cpu": "128"
}
```

{{% note %}}

  * Capacity allocation
  * [Docker logdrivers] & [Sumo log shipping]
  * Environment variables
  * [Task definition reference]

[Docker logdrivers]: https://docs.docker.com/config/containers/logging/awslogs/
[Sumo log shipping]: https://github.com/amaysim-au/serverless-sumologic
[Task definition reference]: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html

{{% /note %}}

{{% /slide %}}

---

{{% slide background-image="/media/JSbQ6paw.png" %}}

## ecs-config.yml

```yaml
---
lb_health_check: /app/healthcheck
lb_health_check_grace_period: 30
lb_deregistration_delay: 60

autoscaling: Enable
autoscaling_target: 60  # what level of CPU utilisation to maintain
autoscaling_min_size: 3
autoscaling_max_size: 20
```

{{% /slide %}}

---

{{% slide background-image="/media/JSbQ6paw.png" %}}

## Makefile & .env

```Makefile
deploy: $(ENV_RM_REQUIRED) $(DOTENV_TARGET) $(ASSUME_REQUIRED)
	docker-compose run --rm ecs make -f /scripts/Makefile deploy

cutover: $(ENV_RM_REQUIRED) $(DOTENV_TARGET) $(ASSUME_REQUIRED)
	docker-compose run --rm ecs make -f /scripts/Makefile cutover

autocleanup: $(ENV_RM_REQUIRED) $(DOTENV_TARGET) $(ASSUME_REQUIRED)
	docker-compose run --rm ecs make -f /scripts/Makefile autocleanup
```

```bash
ENV=Dev
REALM=NonProd
ECS_APP_NAME=Example
ECS_CLUSTER_NAME=Dev
BUILD_VERSION=10-b4cce22

AWS_DEFAULT_REGION=ap-southeast-2
AWS_HOSTED_ZONE=www-dev.amaysim.com.au
BASE_PATH=/551f7c62858899445e42d904170f56ca
```

{{% /slide %}}

---

{{% slide background-image="/media/W7I1OB1A.png" class="orange" %}}

# deployment walkthrough

{{% /slide %}}

---

{{% slide background-image="/media/W7I1OB1A.png" class="orange" %}}

![Version Stack](/media/Version Stack.svg)

{{% /slide %}}

---

{{% slide background-image="/media/W7I1OB1A.png" class="orange" %}}

  * Autoscaling using [Target Tracking]
  * [Placement strategies]

[Target Tracking]: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-autoscaling-targettracking.html
[Placement strategies]: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ecs-service-placementstrategies-placementstrategy.html

{{% /slide %}}

---

{{% slide background-image="/media/W7I1OB1A.png" class="orange" %}}

## cutting over

  * Changing default listener rule
  * Checking the live version
  * Setting desired number of containers
  * blue/green

{{% /slide %}}

---

{{% slide background-image="/media/JSbQ6paw.png" %}}

# verifying and troubleshooting

{{% /slide %}}

---

{{% slide background-image="/media/JSbQ6paw.png" %}}

  1. Check the CloudFormation events
  1. Check the service events and state
  1. Check the task exit reason
  1. Check the cluster state
  1. Check the app logs
  1. Check response directly from load balancer or even container
  1. Check the target group healthchecks
  1. Check CloudWatch metrics
  1. If all else fails: SSH to EC2 instance

{{% note %}}
cookiecutter
.env
path: /551f7c62858899445e42d904170f56ca
export BUILD_VERSION
dockerBuild
dockerPush - fails because wrong account ID
Deploy `example` from laptop to Dev
Note that it is run the same way in GoCD or any CD tool.
Deploy it with a broken image
Demonstrate how to view the container exit status.
Increment version, then build/push/deploy
Demonstrate that the URL does not work
Show ALB rules
Access version through wildcard CloudFront
Run cutover
Demonstrate working app
{{% /note %}}

{{% /slide %}}

---

{{% slide background-image="/media/W7I1OB1A.png" class="orange" %}}

# Thanks!
Questions?

{{% /slide %}}
