---
title: "CI/CD"
date: 2018-12-21T22:21:41+11:00
draft: true
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

## Developer Experience

{{% note %}}
  * Reducing friction to deploy to production
{{% /note %}}

---

## Feedback Loop

Confidence & velocity

{{% note %}}
Benefits of CI/CD
{{% /note %}}

---

## Deployment Benchmarks

  * {{< frag c="Production deployments per day" >}}
  * {{< frag c="Speed" >}}
  * {{< frag c="Collect metrics" >}}

{{% note %}}
  1. Amazon: every 11.7 seconds
  2. amaysim: many times a week
  3. Enterprise: every 6 weeks
{{% /note %}}

---

## Pipelines

{{% note %}}
  * Automatic deploys for 1 environment is not a pipeline
{{% /note %}}

---

## Immutable Artifacts

---

## 12-Factor Configuration

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

---

## Environments

{{% note %}}
Static integrated environments
{{% /note %}}

---

### Slice model/review apps

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
{{% note %}}

---

## Antipatterns

  * {{< frag c="Black box CI servers" >}}
  * {{< frag c="Snowflake CI servers" >}}
  * {{< frag c="Neglecting local development" >}}
  * {{< frag c="Flaky pipelines" >}}{{< frag c=" (especially flaky tests)" >}}
  * {{< frag c="NiH" >}}{{< frag c=" vs Square peg in a round hole" >}}
  * {{< frag c="Developer abstraction" >}}{{< frag c="--- pipeline & infrastructure ownership" >}}
  * {{< frag c="Rolling backwards" >}}
  * {{< frag c="Stale builds" >}}

---

# Thanks!

Questions?
