---
title: "AWS and Cloud"
date: 2019-03-05T13:40:10+11:00
outputs: ["Reveal"]
reveal_hugo:
  custom_theme: "reveal-hugo/themes/robot-lung.css"
  margin: 0.2
---

# AWS and Cloud

---

## APIs and Resources

![](/ec2_resource.png)

---

![Creating a cluster](/ec2_api.png)

---

![boto3 create_cluster](/boto3_ec2_api.png)

---

## Idempotency

>In computing, an idempotent operation is one that has no additional effect if it is called more than once with the same input parameters.
https://stackoverflow.com/a/1077421

---

## Infrastructure as Code

---

### Terraform

![Terraform](/ec2_terraform.png)

---

![Terraform](/terraform.png)

---

### CloudFormation

![CloudFormation](/ec2_cloudformation.png)

---

![CloudFormation console](/cloudformation_console.png)

---

![Stacker](/stacker.png)

---

## Ephemeral and Stateless

![Auto Scaling Group Events](/auto_scaling_group_events.png)

---

# AWS Pricing

>AWS offers you a pay-as-you-go approach for pricing

---

# Auto Scaling

![Auto Scaling Jenkins](/jenkins_autoscaling.png)

---

# Resiliency

  * Loose coupling between services
  * Retries with exponential backoff
  * Circuit breaking
  * Idempotent operations

---

![YC&I Architecture](/ecs_diagram.png)
