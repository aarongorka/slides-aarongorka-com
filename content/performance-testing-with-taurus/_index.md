---
title: "Performance Testing with Taurus"
outputs: ["Reveal"]
reveal_hugo:
  custom_theme: "reveal-hugo/themes/robot-lung.css"
#  margin: 0.2
---

# Performance Testing with Taurus

---

![](/taurus_interface.png)

---

![](/blazemeter_report.png)

---

## What is Taurus

---

[![](/taurus_website.png)](http://gettaurus.org/)

---

![](/taurus_github.png)

---

![](/taurus_dockerhub.png)

---

![](/taurus_documentation.png)

---

# Using Taurus

---

{{% slide background-image="/3musketeers.png" background-size="50%" %}}

---

`test.yml`:
```yaml
---
execution:
- concurrency: 20000
  ramp-up: 10m
  scenario: simple

scenarios:
  simple:
    think-time: 0.75
    requests:
      - https://test.internal.perf.au.edge.zip.co/healthz

reporting:
  - module: passfail
    criteria:
      - avg-rt >1000ms for 10s, stop as failed
      - fail >0% for 10s, stop as failed
```

---

`docker-compose.yml`:
```yaml
version: '3.7'

services:
  taurus:
    image: blazemeter/taurus
    network_mode: host
    ulimits:
      nproc: 65535
      nofile:
        soft: 90000
        hard: 90000
    volumes:
      - .:/bzt-configs
```

---

`Makefile`:
```Makefile
performanceTest:
    docker-compose run --rm taurus test.yml -report
```

---

Thanks!

Questions?

