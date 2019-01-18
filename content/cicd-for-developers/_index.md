---
title: "CI/CD for Developers"
date: 2018-12-21T22:21:41+11:00
outputs: ["Reveal"]
reveal_hugo:
  custom_theme: "reveal-hugo/themes/robot-lung.css"
  margin: 0.2
---

# CI/CD

---

## Continuous Integration

> In software engineering, continuous integration (CI) is the practice of merging all developer working copies to a shared mainline several times a day. - https://en.wikipedia.org/wiki/Continuous_integration

{{% note %}}
Integration hell
Automated tests
Creating a build artifact via a CI server
{{% /note %}}

---

## Continuous Delivery

> Continuous delivery (CD) is a software engineering approach in which teams produce software in short cycles - https://en.wikipedia.org/wiki/Continuous_delivery

---

## Continous Deployment

> Continuous deployment (CD) is a software engineering approach in which software functionalities are delivered frequently through automated deployments. - https://en.wikipedia.org/wiki/Continuous_deployment

---

## Positive Feedback Loop

{{% figure src="/feedback.png" attrlink="https://ovh.github.io/tat/overview/lifecycle/" %}}

{{% note %}}
Benefits of CI/CD
{{% /note %}}

---

{{% slide background-image="/pipelines.jpg" %}}

## Pipelines

{{% note %}}
  * Left to right
  * Automatic deploys for 1 environment is not a pipeline
  * Typical durations
{{% /note %}}

---

## Immutable Artifacts

---

## 12-Factor Configuration

https://12factor.net/

---

## Testing

  * {{< frag c="Unit tests" >}}
  * {{< frag c="Integration tests" >}}
  * {{< frag c="System tests" >}}

---

  * Contract tests
  * Performance tests

---

  * Static analysis
  * CVE scanning
  * Vulnerability scanning

---

## Automatic

### not

### automated

---

## Branching Strategies

  1. Trunk-based with Continuous Deployment
  2. Trunk-based with short-lived feature branches
  3. Trunk-based with versioned release branches

https://trunkbaseddevelopment.com/

---

## Environments

{{% note %}}
Static integrated environments
{{% /note %}}

---

### Review Apps

{{% figure src="/continuous-delivery-review-apps.svg" attrlink="https://about.gitlab.com/product/review-apps/" %}}

---

## Deployment Strategies

  * {{< frag c="In-place" >}}
  * {{< frag c="Rolling" >}}
  * {{< frag c="Blue/green" >}}
  * {{< frag c="Rainbow" >}}
  * {{< frag c="Canary" >}}

---

## Database Versioning

---

## Feature flags

{{% note %}}
  * Launch Darkly
  * GitLab
  * DIY
{{% note %}}

---

## Antipatterns

  * {{< frag c="Snowflake CI servers" >}}
  * {{< frag c="Neglecting local development" >}}
  * {{< frag c="Flaky pipelines" >}}
  * {{< frag c="NiH" >}}
  * {{< frag c="Developer abstraction" >}}
  * {{< frag c="Rolling backwards" >}}

---

[Are you doing CI or CI Theatre ?](http://www.multunus.com/blog/2017/05/ci-theatre/)
[Continuous Integration, not Continuous Isolation](https://damianbrady.com.au/2017/07/12/continuous-integration-not-continuous-isolation/)
[It’s not CI, it’s just CI theatre](https://www.gocd.org/2017/05/16/its-not-CI-its-CI-theatre.html)
