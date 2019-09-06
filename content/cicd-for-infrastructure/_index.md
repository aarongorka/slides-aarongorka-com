---
title: "CI/CD for Infrastructure"
outputs: ["Reveal"]
reveal_hugo:
  custom_theme: "reveal-hugo/themes/robot-lung.css"
  margin: 0.2
---

# CI/CD for Infrastructure

---

## Continuous Integration

> In software engineering, continuous integration (CI) is the practice of merging all developer working copies to a shared mainline several times a day. - https://en.wikipedia.org/wiki/Continuous_integration

{{% note %}}
Created to avoid Integration hell - delaying your PR because it's painful
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
Fast and consistent feedback from deploying to _production_
{{% /note %}}

---

## Branching Strategies

  1. {{< frag c="Trunk-based, commit to trunk" >}}
  2. {{< frag c="Trunk-based with short-lived feature branches" >}}
  3. {{< frag c="Trunk-based with versioned release branches" >}}

---

## Pipelines

![](/simple_pipeline.svg)

{{% note %}}
  * Left to right
  * Blocks on failure - deploy broken code and then iterate
  * Automatic deploys for 1 environment is not a pipeline
{{% /note %}}

---

![](/feature_branch_pipeline.svg)

---

## Immutable Artifacts

{{% note %}}
  * Test the same code from dev to production
  * Do not change code to test in environments
{{% /note %}}

---

## Testing

  * {{< frag c="Static analysis" >}}
  * {{< frag c="Dry run" >}}
  * {{< frag c="System tests" >}}

---

## Environments

---

## Antipatterns

  * {{< frag c="Continuous Isolation" >}}
  * {{< frag c="Testing features by drifting environments" >}}
  * {{< frag c="Delaying the pain of deploying to production" >}}
  * {{< frag c="Relying on manual gating to avoid automated testing" >}}

---

  * [Trunk-based Development](https://trunkbaseddevelopment.com/)
  * [Are you doing CI or CI Theatre?](http://www.multunus.com/blog/2017/05/ci-theatre/)
  * [Continuous Integration, not Continuous Isolation](https://damianbrady.com.au/2017/07/12/continuous-integration-not-continuous-isolation/)
  * [It’s not CI, it’s just CI theatre](https://www.gocd.org/2017/05/16/its-not-CI-its-CI-theatre.html)
