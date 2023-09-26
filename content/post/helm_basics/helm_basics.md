---
title: "Helm basics"
description: "A cheat sheet to get up to speed with Helm and its commands"
date: 2023-05-30T12:54:12-07:00
categories: ["k8s", "helm"]
tags: ["grasping essentials"]
draft: false
---

This post will cover the definition of Helm's building blocks and basic commands.

## Why Helm?

When we intend to deploy an application to a K8s cluster, it's important to know that Kubernetes doesn't inherently understand our application's requirements (and it shouldn't).

What I mean by that is Kubernetes doesn't recognize that PVC "x" is associated with Deployment "y," which, in turn, relies on Service "z", and without these components, the application will not run as intended. All what Kubernetes knows is to manage its resources and thrive on keeping cluster's objects alive.

So we needed a way to "bundle" our Kubernetes application for better release management. We wanted to put the pieces together. Out API app needs a Deployment that requires a Service and we need a storage for that. So we all these parts should deployed, removed, updated, versioned, and shared together, becasue these parts represent a specific application.

Helm is designed to know about our application. That's why it's called a package manager because it looks at our application's K8s manifests as a group (package). It allows us to look at our Kubernetes application as "an application" rather than a collection of Kubernetes objects.

## Helm Chart

A Helm Chart is a **packaged**, **deployable unit** that comprises **all configurations** needed for deploying an application to a Kubernetes cluster.

Let's break this statement down:

- Packaged: A Helm chart is a package for Kubernetes applications, similar to how a Debian apt package bundles software for easy installation on Linux.

- Deployable unit: A Helm chart can be deployed to a K8s cluster, similar to the other K8s objects. It has all

- Includes all configuration files: Within a chart, you'll add all the required K8s manifests (such as service, deployment, replicaset, etc.), that your application needs. Making it easy to deploy and manage our application within a Kubernetes cluster.

By combining these three characteristics, you can deliver your application to any Kubernetes cluster of your choice easily. Not only that, but you can also share your chart (i.e. your applications) so others can deploy it to their cluster too.

## Helm Release

A Helm "release" is a running instance of a Helm chart. When you install a Helm chart, it generates a release with a unique name and version. You can install a chart multiple times, each will have its unique name, and optinally, you can use different variables and parameters.

For example, when you install an Nginx chart, the *installed instance* of the chart is called a "release". You can have one or more Nginx releases deployed from the same chart; each installation has its unique name.

## Helm Repository

A "repository" in Helm refers to a storage location where Helm charts are stored. Like any artifact repository, it can be accessed and shared. It can be a remote or local.You can interact with Helm repositories using the `helm repo <command>` to add, update, or manage repositories.

## Helm Hub

A "hub" refers to a centralized **platform** that hosts a collection of repositores such as the default one Artifact Hub ([artifacthub.io](https://artifacthub.io/)). These hubs provide a user-friendly interface for discovering and exploring Helm repositories.

### Helm Hub vs. Repo

 Think of Helm repositories like *storage* places where Helm charts are stored and Helm hub is like a platform that offers a *search* service to find Helm charts from different repositories. So basically, Helm Hub makes it easy to find and interact with Helm charts, while Helm repositories store the actual chart files.

## Commands cheat sheet

```sh
# Helm works similarly to Linux package managers:
# 1. You add, update, or delete a repository,
# 2. Install, and delete Helm charts.
#
# By default, Helm uses https://artifacthub.io for charts.  
#
# Helm deploys K8s resources, this means that Helm is aware of K8s namespaces.
# Helm will always use the default namespace of the currently activated context 
# in your K8s cluster. 
# Similar to kubectl, use the '-n' flag to interact with K8s namespaces 
# while working with Helm commands.
#
# Search the default Helm hub
helm search hub <chartName>

# Add a repo to your local helm setup
helm repo add bitnami https://charts.bitnami.com/bitnami

# Refresh helm packages list
helm repo update

# Show added repos
helm repo list

# Search for a chart in the previously added repos.
# The `CHART VERSION` shows the chart's version, and
# the `APP VERSION` column shows the application's version (e.g. Nginx's version).
# This command will list the latest version available in the repository.
helm search repo <chartName>

# List all `APP VERSION` for a specific package
helm search repo --versions <chartName>

# Install a chart:
# You can install the same chart multiple times with different names.
# Chart names must be unique. A running instance of a chart is called `release`.
# Sticking to a naming scheme for your releases is always a good idea.
helm install [specify-release-name] [chart-name]
helm install my-release-1 bitnami/nginx
helm install my-release-2 bitnami/nginx -n dev-namespace

# List installed packages
helm list
helm ls
helm ls -a # list `all` including failing deployments
helm ls --all-namespaces

# Uninstall a chart
helm uninstall my-release-1

# Download (+ unpack) the chart but don't install it 
helm pull --untar bitnami/wordpress

# Once downloaded, you can install the chart
helm install [release-4] ./<chart>
helm install my-release ./wordpress

# Install a specific version of a chart (this is `CHART VERSION`)
helm install [release] [repo/chart] --version [version]
helm install my-release bitnami/nginx --version 9.9.4

# Upgrade a release to the latest version available in the repo.
# use `--version` followed by chart version number to upgrade to 
# a certain version.
helm upgrade [release] [chart]
helm upgrade my-release bitnami/nginx

# You can 'downgrade' using the same command but with an older `--version` number
helm upgrade my-release bitnami/nginx --version 9.9.0

# The upgrade command also takes variables updating flags. 
# Here, we are changing the default service type of Nginx 
# from LoadBalancer to a NodePort and the default value of 
# the port from 80 to 8181
helm upgrade my-release --set service.type=NodePort --set service.ports.http=8181 bitnami/nginx 

# To retain the previously modified values when upgrading a release
helm upgrade my-release --version 9.9.5 --reuse-values bitnami/nginx

# Show details of a chart without installing it
helm show chart bitnami/nginx

# Helm `get` to pull extended info about a release
# you can get release's -> 'manifest', 'values', 'notes', 'hooks', 'all'
helm get manifest [releaseName]
helm get values [releaseName]
helm get all [releaseName]

# Get status of a release (deplyed, failed ...etc)
helm status [release]

# List all available values in a chart without downloading it.
# if you're looking for specific flag, pipe the output to grep i.e. ' | grep '
helm show values [repo/chart]
helm show values bitnami/apache | grep replicaCount

# Then to update a default variable, use the '--set' flag
helm install apache-release bitnami/apache --set replicaCount=2

# Control the output using -o [options]
helm search hub nginx-ingress -o yaml
```
