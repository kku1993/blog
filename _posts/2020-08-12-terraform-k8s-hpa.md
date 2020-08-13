---
layout: post
title:  "Kubernetes Horizontal Pod Autoscaler in Terraform"
date:   2020-08-12
categories: tech, terraform, kubernetes
---

A conflict arises when using `kubernetes_horizontal_pod_autoscaler` in Terraform
along with a replication controller or deplyoment. This post shows how to
resolve this using Terraform `lifecycle`.

# Problem

Let's say we have a deployment in kubernetes that we want to scale using a
horizontal pod autoscaler. Your terraform config probably looks something like
this:

```
resource "kubernetes_deployment" "my_deployment" {
  metadata {
    name      = "MyDeployment
  }

  spec {
    replicas = 1  // default value
  }
}

resource "kubernetes_horizontal_pod_autoscaler" "my_hpa" {
  metadata {
    name      = "MyHPA"
  }

  spec {
    max_replicas = 16
    min_replicas = 5

    target_cpu_utilization_percentage = 80

    scale_target_ref {
      api_version = "apps/v1"
      kind        = "Deployment"
      name        = "MyDeployment"
    }
  }
}
```

When you run `terraform plan`, you'll notice that there is a diff. In
particular, the deployment believes that there should only be 1 replica, but the
kubernetes cluster is actually running 5 thanks to the autoscaler. This diff
will continue to show up as long as the autoscaler and deployment both try to
control the number of replicas.

# Solution

Use [ignore_changes](https://www.terraform.io/docs/configuration/resources.html#ignore_changes){:target="_blank"}
to let Terraform know that the number of replicas is controlled by the
autoscaler, and the deployment can safely ignore changes in replica count.

Continuing the example above, we would modify our Terraform config to:

```
resource "kubernetes_deployment" "my_deployment" {
  metadata {
    name      = "MyDeployment
  }

  lifecycle {
    ignore_changes = [
      # Number of replicas is controlled by
      # kubernetes_horizontal_pod_autoscaler, ignore the setting in this
      # deployment template.
      spec[0].replicas,
    ]
  }

  spec {
    replicas = 1  // default value
  }
}

resource "kubernetes_horizontal_pod_autoscaler" "my_hpa" {
  metadata {
    name      = "MyHPA"
  }

  spec {
    max_replicas = 16
    min_replicas = 5

    target_cpu_utilization_percentage = 80

    scale_target_ref {
      api_version = "apps/v1"
      kind        = "Deployment"
      name        = "MyDeployment"
    }
  }
}
```
