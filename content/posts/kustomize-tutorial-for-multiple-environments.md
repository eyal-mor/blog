---
title: "Kustomize Tutorial"
draft: true
type: post
tags: [development, k8s, kubernetes, kustomize, kubectl -k]
series: [kubernetes management]
---

## Introduction

This is a first post in a series to look for my choice of tooling to manage Deployments in a K8S (Kubernetes) cluster.

In this post, we will look at how to use Kustomize to create a development + production deployment.

## What is Kustomize

If you want to join YAML files together for multiple environments, and like using straight up kubectl, look no further than [Kustomize](https://kustomize.io/).

Kustomize's power comes from its simplicity. It doesn't manage state, doesn't handle type safety, or even powerful templating.

It is purely a data structure merger. Want to merge 2 objects? Merge lists? Override lists? Merge nested data structures? Kustomize probably has a solution for you.

One of the major benefits when reading/learning about Kustomize, is that it's part of the K8S SIGs (Special Interest Group). Freebie benefit right of the bat!

## How to use it - High Level Overview

A simple interface is presented to the user:

```yaml
# Tell Kubectl this is an API object to be handled by kustomize, of version /1beta1
apiVersion: kustomize.config.k8s.io/v1beta1
# The kind of the API Object, references which schema to use when validating.
kind: Kustomization

# A list of base files to pull in.
# Can also be a directory full of YAML files
bases:
  - ./base
# A list of YAML files to patch.
patchesStrategicMerge:
  - ... # We don't talk about Hugo yet
```

