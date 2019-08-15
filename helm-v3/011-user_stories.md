# Helm v3 User Stories

This document lists [user stories](https://en.wikipedia.org/wiki/User_story) for the development of Helm's 3rd major version. The user stories here are not a complete set for Helm but those that build upon Helm 2. The stories are grouped by role as defined in the [User profiles](../user-profiles.md).

## 1. Application Operator

1. As an application operator, I want to be able to pull charts from within the cluster so that I can manage multiple clusters without exposing details to the CI system
1. As an application operator, I want to be able to pull charts, so that my dev team can bootstrap local envs easily (e.g. w/Minikube or Docker)
1. In order to have a secure system as an application operator, I want Helm to install/upgrade/delete based on the user’s credentials
1. In order to have a secure system as an application operator, I want to be able to see who ran a particular install or other operation
1. In order to have a secure system as an application operator, I want out-of-the-box “easy security”
1. In order to have a secure system as an application operator, I want RBACs done for me
1. In order to have a secure system as an application operator, I want to audit what charts have been deployed
1. In order to have a secure system as an application operator, I want a pull model so that I can manage multiple private clusters from one authoritative source
1. In order to have a secure system as an application operator, I want to enable RBAC on an in-cluster controller so that I can prevent external changes from happening
1. In order to easily get started as an application operator, I want to easily install and use Helm

## 2. Application Distributor

1. As an application distributor, I want to develop charts using a template engine of my choice (e.g., Jinja, KSonnet, JSonnet)
1. In order to have reusability as an application distributor, I want to use common libraries to build charts
1. As an application distributor, I want to be able to override chart templates (e.g., those in dependency charts)
1. As an application distributor, I want to intercept rendered YAML/JSON and do last-mile modifications (e.g. insert or remove)
1. As an application distributor, I want lifecycle hooks for pre- and post-render operations
1. As an application distributor, I want to be able to do simple variable substitution in values.yaml (and reference in --set)

## 3. Application Developer

1. As a application developer, I want to automatically pull charts so that my local env looks like production

## 4. Supporting Tool Developer

1. As a supporting tool developer, I want to interact with Helm without using gRPC.
1. As a supporting tool developer, I want to write plugins that intercept hooks
1. As a supporting tool developer, I want to be able to write alternate back-ends (for deploying)

## 5. Helm Developer

1. TBD
