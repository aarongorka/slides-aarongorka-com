---
title: "Working Effectively with CI/CD Pipelines"
date: 2020-05-12T15:03:19+10:00
outputs: ["Reveal"]
reveal_hugo:
  custom_theme: "reveal-hugo/themes/robot-lung.css"
  margin: 0.2
---

# Working Effectively with CI/CD Pipelines

---

## Overview

  * When and when not to leverage the CI/CD pipeline
  * Assumes familiarity with CI/CD basics
  * Does not cover all edge-cases, there are exceptions

---

## Not all changes are equal

  1. Updating Security Group with new IP address
  1. Deploying new AWS resource e.g. AWS Neptune
  1. Developing script e.g. to orchestrate DNS
  1. Updating IAM permissions for AWS Role

---

## What should I use a CI/CD pipeline for?

  * {{% fragment %}}Updating something when the outcome is very predictable{{% /fragment %}}
  * {{% fragment %}}Validating more complex changes that have been sufficiently tested{{% /fragment %}}
  * {{% fragment %}}Copying existing, working code and using it in other environments{{% /fragment %}}
  * {{% fragment %}}Double checking that your new feature passes tests in a consistent environment{{% /fragment %}}
  * {{% fragment %}}Testing changes to the pipeline itself{{% /fragment %}}

---

## Feedback loop

  * {{% fragment %}}Running on workstation with hot reload - instant (milliseconds){{% /fragment %}}
  * {{% fragment %}}Compiling on workstation - fast (seconds){{% /fragment %}}
  * {{% fragment %}}Deploying Lambda from workstation - okay (tens of seconds){{% /fragment %}}
  * {{% fragment %}}Deploying Lambda/Docker from pipeline - slow (minutes){{% /fragment %}}
  * {{% fragment %}}Deploying AWS infrastructure from workstation - very slow (many minutes){{% /fragment %}}
  * {{% fragment %}}Baking an AMI, deploying it to an ASG with rolling updates - extremely slow (tens of minutes){{% /fragment %}}

---

## What should I _not_ use a CI/CD pipeline for?

  * {{% fragment %}}_Developing_ code{{% /fragment %}}
  * {{% fragment %}}_Experimentation_ for proof of concepts{{% /fragment %}}
  * {{% fragment %}}_Researching_ a new, unfamiliar tech stack{{% /fragment %}}
  * {{% fragment %}}_Checking_ whether you have the right syntax for a command{{% /fragment %}}
  * {{% fragment %}}_Orchestrating_ changes with complex dependency graphs{{% /fragment %}}

---

## Considerations

---

### Parameterisation

  * Script should ideally be able to be run locally
  * Mock/conditional logic for things that can't run locally e.g. EC2 metadata
  * Pass parameters (env vars, CLI flags) to set URLs, etc.
  * Scripts should not concern themselves with authentication
    * EC2 instances will have instance profiles
    * Running locally will use personal credentials
    * CI system will have commands for assume role

---

### Mocking

```python
def create_aem_volume(client):
    time.sleep(10)
    response = client.create_volume(name="foobar", size="100")
    return {"VolumeId": response["VolumeId"]}

@patch('time.sleep')
def test_can_create_aem_volume(mock_sleep):
    client = boto3.client('ec2')  # normal boto3 client
    stubber = Stubber(client)  # put a wrapper around the client
    stubber.add_response('create_volume', {"VolumeId": "a81d2cc238226c8fd333b3ec2b53ada772b386be"})  # mock the response for `create_volume` with a hardcoded value
    stubber.activate()  # activate it.
    response = create_aem_volume(client=client)  # pass the client to our function, now any API calls it makes will be mocked
    assert response = {"VolumeId": "a81d2cc238226c8fd333b3ec2b53ada772b386be"}
```

{{% fragment %}}TL;DR: any function using boto3 should accept clients as an argument to facilitate easy mocking{{% /fragment %}}

---

### Running Pipeline Steps Locally

  * Everything we do in the pipeline should be able to be run locally
  * Yes, we should be able to build and deploy even the jankiest monolith from our workstations
  * Avoiding functionality as Jenkins plugins helps with this
  * All things within reason; needing to run Jenkins locally may be overkill

---

## Signs you may need to change your workflow

  * Excessive amounts of commits in a PR
  * Commit messages not meaningful
  * Using the pipeline as a remote shell
  * Fighting self-healing/immutable infrastructure
  * Complex orchestration/coupling of infrastructure creation to fulfill dependency graph

---

# Conclusion

  * The CI system is not an efficient first pass for validating code
  * Appropriate feedback loop for appropriate changes:
    * Fast feedback loop for experimentation
    * Thorough testing once reasonably confident
  * Write things to be run locally

---

# Thanks
