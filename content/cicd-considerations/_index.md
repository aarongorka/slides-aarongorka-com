---
title: "Continuous Delivery Considerations"
date: 2021-03-02T15:47:48+11:00
outputs: ["Reveal"]
reveal_hugo:
  custom_theme: "reveal-hugo/themes/robot-lung.css"
  # margin: 0.2
  margin: 0.0
---

# Considerations for moving towards CD

---

## Continuous Delivery

> Continuous delivery (CD) is a software engineering approach in which teams produce software in short cycles - https://en.wikipedia.org/wiki/Continuous_delivery

---

## Why?

  * {{< frag c="Positive feedback loop" >}}
  * {{< frag c="Smaller releases" >}}
  * {{< frag c="Reduced cognitive burden" >}}
  * {{< frag c="Less stressful releases" >}}

{{% note %}}
Smaller, more frequent changes == easier to debug, resolve, smaller blast radius
Reduced cognitive burden enables you to focus on the imporant stuff
Positive feedback loop forces you in to better patterns
If something is painful, do it frequently
{{% /note %}}

---

### Positive Feedback Loop

{{% figure src="/feedback.png" attrlink="https://ovh.github.io/tat/overview/lifecycle/" %}}

---

## Considerations

---

### Always be ready to deploy trunk

{{% note %}}
Don't intentionally merge broken changes
{{% /note %}}

---

### Favour small and frequent changes

![](/ci-certification.png)

---

### Post merge is your responsibility

---

### Backwards compatability

{{% note %}}
Changes must be compatible back 1 version to enable high availability during deployments
{{% /note %}}

---

### Control feature rollout with system tests

---

### Local first development

{{% note %}}
Don't over-rely on the pipeline, feedback is slow
{{% /note %}}

---

### Database & other changes as part of the pipeline

---

### Non-application code as first-class citizens

---

## Troubleshooting

---

### CI

---

### AWS Console

---

### Logs

---

### Metrics

---

### Shell

---

## Links

  * [Are you doing CI or CI Theatre ?](http://www.multunus.com/blog/2017/05/ci-theatre/)
  * [Continuous Integration, not Continuous Isolation](https://damianbrady.com.au/2017/07/12/continuous-integration-not-continuous-isolation/)
  * [It’s not CI, it’s just CI theatre](https://www.gocd.org/2017/05/16/its-not-CI-its-CI-theatre.html)
