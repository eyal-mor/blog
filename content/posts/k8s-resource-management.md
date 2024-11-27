---
title: "Helm vs Kustomize vs Terraform vs Tanka vs Kapitan vs Dagger"
date: 2022-05-22T18:05:47+03:00
draft: false
type: post
categories: [development, k8s, kubernetes]
tags: [hugo, content, static site generator]
---

> This post is WIP.

## Preface

K8S has become more than a way to run pods. It had become a ubiquitous API layer for cloud resource management.

Mature, well tested controllers provide life cycle hooks for all CRUD operations of a resource (or multiple resources in some cases)

Let's keep in mind, K8S is a REST api on steroids. It allows extending the core API with CRDs and custom controllers by enforcing a common pattern on all APIs that are deployed within it.

If you have worked with any cloud provider, you have always interacted with a REST API.

My proficiency is with AWS, but the concept still stands for Other Cloud Providers.

Through the UI (console) the web browser generates API requests that turns into Cloud Resources. Through the CLI, same thing with less RAM and less JavaScript (most of the times).

A common pattern emerged is to use a State Management tool to simplify our REST API calls, dependency management, and most importantly, state of our resources.

This movement is now called GitOps, but it had many names & shapes before.

(Put Example Here)

Cloud Native Management
With all that theory out of the way, let's talk practice.

### Helm

The Helm project has been used from almost the beginning of the K8S project.

The current recommended (and secure) version is Helm 3.

Helm uses Golang's templating features to provide variable substitution.

State management is achieved by Helm adding metadata tags on each API resource it creates, and queries the K8S API to maintain state.

Helm is very popular and is considered a standard tool in many Cloud Native scenarios.

### Kustomize

If you want to join YAML files together for multiple environments, and like using straight up kubectl, look no further than Kustomize.

Kustomize's power comes from its simplicity. It doesn't manage state, doesn't handle type safety, or even powerful templating.

It is purely a data structure merger. Want to merge 2 objects? Merge lists? Override lists? Merge nested data structures? Kustomize probably has a solution for you.

### Terraform / OpenTofu

At a high level, Terraform is a templating language and state management solution for the Cloud.
It's power comes from its providers.

Unlike the tools listed here, Terraform was created to make manage the dependencies of Cloud Resources.

This means automatically resolving which resources depends on the creation of other resources.

One thing to note when using Terraform is that you can use multiple providers to support a vast amount of use-cases.

### Tanka

From Grafana we get this cool project.

Instead of YAML/JSON manipulation, Tanka takes the approach of using a templating language - [Jsonnet](https://jsonnet.org/).

Jsonnet unlike YAML is a programming language with a runtime.

That means we can leverage our development experience from other languages, and apply it to a new language.

# Comparison

| Tool      | Language | Type Safety |       Templating       | State Management |                                                                Stars                                                                |
| :-------- | :------: | :---------: | :--------------------: | :--------------: | :---------------------------------------------------------------------------------------------------------------------------------: |
| Helm      |   YAML   |  False'ish  | Go Templating Language |       True       |                 [![GitHub stars](https://img.shields.io/github/stars/helm/helm.svg)](https://github.com/helm/helm/)                 |
| Kustomize |   YAML   |    False    |     JSON Merge RFC     |      False       | [![GitHub stars](https://img.shields.io/github/stars/kubernetes-sigs/kustomize.svg)](https://github.com/kubernetes-sigs/kustomize/) |
| Terraform |   HCL    |  True'ish   |          HCL           |       True       |         [![GitHub stars](https://img.shields.io/github/stars/opentofu/opentofu.svg)](https://github.com/opentofu/opentofu/)         |
| Tanka     | Jsonnet  |    True     |        Jsonnet         |    False'ish     |             [![GitHub stars](https://img.shields.io/github/stars/grafana/tanka.svg)](https://github.com/grafana/tanka/)             |

## Conclusions

When working with K8S, I personally strive to use the most popular, most cost effective and the easiest tool to google my issues with.

This would make Helm or OpenTofu the winner, however, the bugs due to the unsafe usage of YAML + Golang as a templating language may cause grief.
For my personal projects I lean towards OpenTofu, as HCL outputs JSON, which doesn't have any of the pitfalls of Golang templating wrapping around YAML.

Additionally, OpenTofu is versatile enough to handle Helm as well through a provider.

With that said, the best recommendation is to use what you already know to achieve your goals, in this case, deploying Pods.
