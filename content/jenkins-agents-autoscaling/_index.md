---
title: "Jenkins Agents Autoscaling"
outputs: ["Reveal"]
reveal_hugo:
  custom_theme: "reveal-hugo/themes/robot-lung.css"
  margin: 0.2
---

# Jenkins Agents Autoscaling

---

## Architecture

---

![architecture](/jenkins_architecture.png)

---

## Jenkins Swarm Plugin

https://wiki.jenkins.io/display/JENKINS/Swarm+Plugin

---

  * **Unrelated to Docker Swarm**
  * Default agent jar requires click-to-setup for each node
  * Swarm allows an arbitrary number of agents to connect
  * Fully automated

---

```bash
echo "JENKINS_AGENTS_PASSWORD=$(aws --region=ap-southeast-2 ssm get-parameters --names "JENKINS_AGENTS_PASSWORD" --with-decryption | jq -r '.["Parameters"][0]["Value"]')" >> /var/lib/jenkins/systemd.env

curl -Ls https://repo.jenkins-ci.org/releases/org/jenkins-ci/plugins/swarm-client/3.9/swarm-client-3.9.jar -o /swarm-client.jar

cat <<EOF >> /etc/systemd/system/jenkins-agent.service
[Unit]
Description=Jenkins Agent

[Service]
User=jenkins
WorkingDirectory=/var/lib/jenkins
ExecStart=/usr/bin/java -jar /swarm-client.jar -master http://jenkins.example.com -tunnel jenkins-master.example.com:43863 -fsroot /var/lib/jenkins -username agents -passwordEnvVariable JENKINS_AGENTS_PASSWORD
EnvironmentFile=/var/lib/jenkins/systemd.env
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

--- 

# Autoscaling?

:thinking:

---

### docker-jenkins-autoscaler

https://github.com/aarongorka/docker-jenkins-autoscaler

![aarongorka/docker-jenkins-autoscaler](/jenkins_autoscaler.png)

---

**A utility container that helps autoscaling your Jenkins nodes in AWS.**

It does two things:

  1. Provides a custom metric that the node ASG can use to scale off of: idle executors
  2. Protects busy agents from being terminated during scale-in

---

  * Docker container that runs on the master
  * Written in Python
  * Uses `requests` to scrape Jenkins API
  * Uses `boto3` to make API calls to AWS

---

```python
if __name__ == "__main__":
    while True:
        put_cw_metrics()
        protect_busy_nodes()
        time.sleep(10)


def put_cw_metrics(master, username, password, url):
    """Poll Jenkins API for number of busy/idle executors."""

    response = requests.get("http://localhost:8080/computer/api/python?pretty=true")
    # ...


def protect_busy_nodes(username, password, url):
    """Polls Jenkins for idle nodes. Sets scale-in protection on those that aren't idle, and removes it from those that are."""

    response = requests.get("http://localhost:8080/computer/api/json?tree=computer[idle,displayName]").json()
    # ...
```

---

![scale-in](/scale-in.png)

---

```hcl
resource "aws_autoscaling_policy" "jenkins-agents-scale-in-policy" {
  name                   = "jenkins-agents-scale-in-policy"
  autoscaling_group_name = "${aws_autoscaling_group.jenkins-agents.name}"
  scaling_adjustment     = "-1"
  adjustment_type        = "ChangeInCapacity"
}
resource "aws_cloudwatch_metric_alarm" "jenkins-agents-scale-in-alarm" {
  alarm_name          = "jenkins-agents-scale-in"
  comparison_operator = "GreaterThanOrEqualToThreshold"
  metric_name         = "FreeExecutors"
  namespace           = "Jenkins"
  dimensions = {
    JenkinsMaster = "jenkins.example.com"
  }
  period              = "30"
  evaluation_periods  = "6"
  statistic           = "Average"
  threshold           = "5"

  alarm_description = "Scale in Jenkins agents"
  alarm_actions     = ["${aws_autoscaling_policy.jenkins-agents-scale-in-policy.arn}"]
}

resource "aws_autoscaling_policy" "jenkins-agents-scale-out-policy" {
  name                   = "jenkins-agents-scale-out-policy"
  autoscaling_group_name = "${aws_autoscaling_group.jenkins-agents.name}"
  scaling_adjustment     = "2"
  adjustment_type        = "ChangeInCapacity"
}
resource "aws_cloudwatch_metric_alarm" "jenkins-agents-scale-out-alarm" {
  alarm_name          = "jenkins-agents-scale-out"
  comparison_operator = "LessThanOrEqualToThreshold"
  metric_name         = "FreeExecutors"
  namespace           = "Jenkins"
  dimensions = {
    JenkinsMaster = "jenkins.example.com"
  }
  period              = "30"
  evaluation_periods  = "1"
  statistic           = "Minimum"
  threshold           = "0"

  alarm_description = "Scale in Jenkins agents"
  alarm_actions     = ["${aws_autoscaling_policy.jenkins-agents-scale-out-policy.arn}"]
}
```

---

![alarms](/alarms.png)

---

![metrics](/metrics.png)

---

The End
