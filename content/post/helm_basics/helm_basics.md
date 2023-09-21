---
title: "Helm basics"
description: "A cheat sheet to get up to speed with Helm and its commands"
date: 2023-05-30T12:54:12-07:00
categories: ["k8s", "helm"]
tags: ["grasping essentials"]
draft: false
---

Helm, often referred to as the package manager for Kubernetes, provides a streamlined and efficient way to deploy, upgrade, and manage application deployments on your Kubernetes clusters. This post will go over the definition of Helm's building blocks and basic commands.

## Why Helm?

It's important to know that Kubernetes doesn't know anything about our application's requirements. It doesn't know that PVC "x" is part of Deployment "y" which needs Service "z", and without these parts the application will not work. All what Kubernetes knows is to manage its resources and thrive on keeping its components in the cluster alive. Helm, on the other hand, is designed to know about our application. That's why it's called a package manager because it looks at our application's manifests as a group (package). It allows us to look at our Kubernetes application as "an application" rather than a collection of Kubernetes objects.

## Helm Chart

A Helm Chart is a packaged, deployable unit that encapsulates all the necessary components and configurations required to deploy an application on Kubernetes. For example, an Nginx Helm chart includes all the necessary Kubernetes manifests (`service`, `deployment`, `replicaset`, ... etc), required to easily deploy and manage Nginx to your K8s cluster.

## Helm Release

A "release" refers to a specific instance of a deployed application in a Kubernetes cluster. When you install a Helm chart, it generates a release with a unique name and version. The release tracks the deployed application's lifecycle, including information such as the chart version, configuration values, deployed resources, and any other metadata associated with the deployment. Simply put, when you install an Nginx chart, the installed instance of the chart is called a "release". You could have one or more Nginx releases deployed from the same chart; each has its unique name and attributes.

## Helm Repository

A "repository" in Helm refers to a storage location where Helm charts are stored, accessed, and shared. It can be a remote or local location containing packaged charts that can be fetched and installed using Helm commands. You can interact with Helm repositories using the `helm repo <command>` to add, update, or manage repositories.

## Helm Hub

A "hub" refers to a centralized platform or service that hosts a collection of repositores. These hubs, such as the default one Artifact Hub ([artifacthub.io](https://artifacthub.io/)), provide a user-friendly interface for discovering and exploring Helm charts. The hub offers a way for you to discover charts stored in various repositories managed by different individuals and organizations. They serve as curated repositories that offer a wide range of repositories that houses Helm charts for users to search, browse, and install.

While the distinction between a "hub" and a "repository" can vary based on context, it's worth noting that the terms are sometimes used interchangeably, and the functionalities they provide may overlap.

## Commands cheat sheet

```sh
# Helm works similarly to Linux package managers:
# 1. You add, update, or delete a repository,
# 2. Install, and delete Helm charts.
#
# By default Helm uses https://artifacthub.io for charts.  
#
# Helm deploys K8s resource; this means that Helm is aware of K8s namespaces.
# Helm will always use the default namespace of the currently activated context 
# in your K8s cluster. 
# Similar to kubectl, use the '-n' flag to interact with k8s namespaces 
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
