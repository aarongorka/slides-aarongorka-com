---
title: "Introduction to Cookiecutter"
outputs: ["Reveal"]
featuredImage: "/intro_to_cookiecutter_preview.png"
reveal_hugo:
  custom_theme: "reveal-hugo/themes/robot-lung.css"
  margin: 0.2
---

# Introduction to

![](/cookiecutter_medium.png)

---

## Cookiecutter Overview

  * Project templating tool
  * Language agnostic
  * Uses Jinja templating
  * Installs with Python
  * FOSS: https://github.com/cookiecutter/cookiecutter

---

## The Problem

---

### A Hypothetical Situation
  * Starting a new client
  * Need to deploy some projects
  * Modules to do the hard work already exist
  * Still need some **implementation** repositories

How do I do this? Copy & paste some previous project?
{{< frag c="¯\_(ツ)_/¯" >}}

---

### What This Results In
  * No clear process for deploying modules
  * Quite often end up reinventing the wheel
  * Copying over baggage from previous clients
  * Changes may not be simple search & replace
  * No way to easily share fixes/enhancements

---

### A Solution
  * {{< frag c="boilerplate | scaffolding" >}}
  * {{< frag c="clone template -> generate project" >}}

---

Some implementaitons:

  * Yeoman: https://yeoman.io/
  * JHipster: https://www.jhipster.tech/
  * Maven archetypes: https://maven.apache.org/archetypes/
  * Create React App: https://github.com/facebook/create-react-app
  * {{% fragment %}}Cookiecutter: https://github.com/cookiecutter/cookiecutter{{% /fragment %}}

---

## Why Cookiecutter?

  * Language agnostic, unopinionated
  * Interactive prompts for values
  * _Simple_ Jinja templating
  * Doesn't require Nodejs

---

## Installation

```
pip3 install --user cookiecutter
```

---

## Usage

```
$ cookiecutter git@gitlab-ssh.runcmd.cmdsolutions.com.au:cmd-terraform/cookiecutter-tf-aws-eks.git --checkout init
```

... then fill out the prompts

---

## Demo

`<demo goes here>`

---

Cookiecutter relies on adoption to be useful

---

## Creating Your Own Cookiecutter

---

## Cookiecutter Example

```shell
cookiecutter-tf-aws-eks $ tree
.
├── CODEOWNERS
├── README.md
├── cookiecutter.json
└── {{ cookiecutter.project_slug }}
    ├── backend.tf
    ├── docker-compose.yml
    ├── envvars.yml
    ├── locals.tf
    ├── main.tf
    ├── Makefile
    ├── provider.tf
    └── README.md
```

---

`backend.tf`:
```hcl
terraform {
  backend "s3" {
    bucket  = "{{ cookiecutter.client_name }}-terraform-backend"
    key     = "{{ cookiecutter.client_name }}-tf-aws-eks"
    region  = "ap-southeast-2"
    profile = "{{ cookiecutter.client_name }}-tfbackend"
    # https://github.com/terraform-providers/terraform-provider-aws/issues/5018#issuecomment-465332353
    skip_metadata_api_check = true
  }
}
```

---

`cookiecutter.json`:
```json
{
  "client_name_pretty": "Name of the client the implementation is being created for, prettified e.g. 'Mantel Group'",
  "client_name": "{{ cookiecutter.client_name_pretty.lower().replace(' ', '-') }}",
  "project_name": "{{ cookiecutter.client_name_pretty }} TF AWS EKS",
  "project_slug": "{{ cookiecutter.project_name.lower().replace(' ', '-') }}",
  "project_short_description": "{{ cookiecutter.client_name_pretty }} Terraform AWS EKS Implementation",
  "aws_account_id_sandpit": "AWS Account ID of your sandpit account for assume role",
  "aws_account_id_nonprod": "AWS Account ID of your nonprod account for assume role",
  "aws_account_id_prod": "AWS Account ID of your prod account for assume role",
  "aws_account_id_tfbackend": "AWS Account ID of your tfbackend account for backend assume role"
}
```

---

# Advanced Usage

---

## Retroactive Updates
First commit should be the cookie without any edits to allow future updates

```text
5. Both application changes and     <---- merge branch --+   4. Updated cookie
   and features from cookiecutter
                                                                     ^
       ^                                                             |
       |                                                             |
       |                                                             |
       |                                                             |
       +                                                             |
                                                             Rerun cookiecutter
2. application development, custom changes                           |
                                                                     |
       ^                                                             |
       |                                                             |
       |                                                             |
       +                                                             +

1. root commit   +--------- branch ---------------------->   3. detached commit
```

---

## CI/CD on Cookiecutters

`Makefile`:
```Makefile
clone:
	docker-compose run --rm cookiecutter --no-input --overwrite-if-exists . project_name='Python Test Project'
	$(MAKE) -C python-test-project .env

recursive:
	$(MAKE) -C python-test-project deps build styleTest run #deploy smokeTest remove
```

Source: https://github.com/aarongorka/serverless-python-boilerplate/blob/master/Makefile

---

# Fin
