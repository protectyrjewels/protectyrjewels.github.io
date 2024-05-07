---
title: Declarative Helm deployments using Helmfile
date: 2024-05-07 18:59:00 +01:00
tags: [github actions, helm, helmfile]
description: Code recipe for using Helmfile and Github Actions together
---

If you are looking to transition to adopt a more declarative deployment style for your Helm charts, a useful tool to know is [helmfile](https://github.com/helmfile/helmfile) (or [helmsman](https://github.com/Praqma/helmsman) as an alternative).


Helmfile uses a single .yaml file, called a helmfile, containing all the configurations of the Helm releases. It supports local and remote Helm charts, multiple environments (which can be selected with the -e option), deploying to multiple clusters, hooks, and can work together with kustomize.

Here's a simple `helmfile.yaml` for a single local chart release:

```yaml
environments:
  dev:
    values:
      - ./values/dev.yaml
  prod:
    values:
      - ./values/prod.yaml

---

helmDefaults:
  wait: true
  waitForJobs: true
  atomic: true
  timeout: 90

releases:
- name: micro-service
  namespace: default
  chart: ./microservice # local chart
  values:
    - image:
        tag: { { .Values.micro_service.image.tag } }
```

The dev.yaml and prod.yaml are just normal yaml files:

```yaml
micro_service:
  image:
    tag: "latest"
```

Here's how I'm using it in my Github Actions CD pipeline:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v4

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4.0.2
      with:
        aws-access-key-id: ${ { secrets.AWS_ACCESS_KEY_ID } }
        aws-secret-access-key: ${ { secrets.AWS_SECRET_ACCESS_KEY } }
        aws-region: us-west-1

    - name: Install dependencies (kubectl, helm, helmfile)
      uses: mamezou-tech/setup-helmfile@v2.0.0

    - name: Validate Helmfile
      run: helmfile -e dev -f deploy/helmfile.yaml lint

    - name: Update Kubeconfig
      run: |
        aws eks update-kubeconfig --region us-west-1 --name super-cluster

    - name: Deploy with Helmfile
      run: |
        helmfile -e dev -f deploy/helmfile.yaml --log-level debug apply
```

The last command `apply` will try to synchronize the Kubernetes cluster state with the state declared in the helmfile. 

If you want to debug your deployments before applying them, you can use the `template` command.

Easy-peasy.
