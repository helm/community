# User Profiles

The purpose of this document is to aid in the development of Helm features by evaluating Helm features against those who will be using them.

When considering requirements and implementation solutions it's useful look at who uses Helm to have a better understanding of how something should work. The Helm user profiles describe the different types of users and contributors to Helm along with the order of priority they have relative to each other. The ordering is because nothing can share the exact same priority and we want to convey that.

## Preface

* What's described here are user profiles as opposed to [personas](https://en.wikipedia.org/wiki/Persona#In_user_experience_design). Personas are example actors rather than general categories.
* Kubernetes Cluster Operators are out of scope for this document. A cluster operator is one who manages the operation of a Kubernetes cluster that applications can run in.

## Profiles

Profiles describe a type of role a user may perform. A real person may perform more than one role and have more than one profile apply to them. How this mapping works between profiles and real people can vary between companies and other organizations. To handle this variation we focus on the user profiles rather than how they may map to people in these different organizations.

### 1. Application Operator

Application operators take an application and operate it within a Kubernetes cluster. For example, the operation of WordPress and MySQL. This is not to be confused with the role of a Kubernetes cluster operator (see the preface above for more detail).

### 2. Application Distributor

Distributors are people who package an application for someone else to operate. Examples of this would be those who maintainer [Community Charts](https://github.com/kubernetes/charts) such as WordPress and Datadog.

### 3. Application Developer

An application developer writes the software for an application. They are typically not concerned with where the application is running as applications can often be run more than one environment (e.g., many applications can be run in a virtual machine or a container). Examples of this include the developers of WordPress and MySQL.

### 4. Supporting Tool Developer

Supporting tool developers build tools adjacent to Helm such as a linter, Helm plugin, or even kubectl. These are developers building complementary things that can be used along with Helm.

### 5. Helm Developer

Helm developers are those who develop Helm itself. That includes core maintainers along with anyone else who fixes a bug or updates docs.

Generally speaking, the developers of Helm consider the end users above themselves when looking at requirements and implementation strategies.

## Profiles Not In Scope

Some user profiles are not considered in scope for Helm. That does not mean a real person who multiple profiles apply to is not considered a supported user. Rather, the out of scope profiles apply to roles that are not typically supported.

## Cluster Operator

A cluster operator stands up and operates a Kubernetes cluster. This includes elements such as the control plane, nodes, and elements in the stack below these. It does not include applications running on Kubernetes as those are handled by the _Application Operator_ profile.