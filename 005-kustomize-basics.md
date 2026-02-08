## Kustomize Basics

### 1. Introduction to Kustomize

When working with Kubernetes, it is very common to run the **same application** in multiple environments such as **development**, **staging**, and **production**.

Even though the application is the same, each environment usually behaves a little differently.  
These differences are often small, but important.

---

### 2. The problem scenario

Let’s assume we have a simple **Nginx Deployment**.

We want to deploy:
- 1 replica in the DEV environment
- 2–3 replicas in the STAGE environment
- 5–10 replicas in the PROD environment

Everything else in the deployment stays the same.

---

### 3. A simple and working solution

One of the simplest ways to handle this is to create **separate directories for each environment** and copy the Kubernetes configuration into each one.

Example directory structure:

    dev
    │── nginx-deply.yaml
    
    stg
    │── nginx-deply.yaml
    
    prod
    │── nginx-deply.yaml

Each directory contains its own version of the deployment file.

---

### 4. Applying environment-specific configurations

To deploy these configurations, we can run:

kubectl apply -f dev/  
Result: 1 pod is created

kubectl apply -f stg/  
Result: 2 pods are created

kubectl apply -f prod/  
Result: 3 pods are created

This approach works correctly and Kubernetes behaves exactly as expected.

---

### 5. Why this approach is not scalable

Although this solution works, it has serious limitations.

As your Kubernetes setup grows:

- You add more resources such as Services, ConfigMaps, Secrets
- You must copy and modify these files for every environment
- The same configuration is duplicated many times
- Keeping everything in sync becomes difficult
- Small changes require editing multiple files

As the number of environments and resources increases, this approach becomes hard to manage.

---

### 6. Introducing Kustomize

Kustomize solves this exact problem.

Instead of copying YAML files for each environment, Kustomize lets you:

- Keep common configuration in one place
- Apply environment-specific changes on top
- Avoid duplication
- Scale cleanly as your setup grows

---

### 7. Core Kustomize concepts

Kustomize is built around two key concepts:

- Base configuration
- Overlays

These two concepts work together to produce the final Kubernetes configuration.

---

### 8. Base configuration

The **base configuration** contains everything that is identical across all environments.

This includes:
- Deployments
- Services
- Default values
- Shared resources

Anything you know will be the same in DEV, STAGE, and PROD belongs in the base.

The base also defines **default values** that can be overridden later.

---

### 9. Base example

If the base configuration contains a Deployment file:

- The deployment.yaml file is identical across all environments
- It contains default values

For example:

    spec:
      replicas: 1

This value represents a default that overlays can override.

---

### 10. Overlays

An **overlay** customizes the base configuration for a specific environment.

Each environment gets its own overlay.

Overlays allow you to:
- Change only what is different
- Override values from the base
- Keep environment-specific logic isolated

---

### 11. Overlay examples

Base configuration:

spec:
  replicas: 1

DEV overlay:

spec:
  replicas: 1

STAGE overlay:

spec:
  replicas: 2

PROD overlay:

spec:
  replicas: 5

Only the replica count changes, while everything else stays the same.

---

### 12. Kustomize directory structure

A common Kustomize directory structure looks like this:

    k8s
    ├── base
    │   ├── kustomization.yaml
    │   ├── nginx-deply.yaml
    │   ├── service.yaml
    │   └── redis-deply.yaml
    └── overlays
        ├── dev
        │   ├── kustomization.yaml
        │   └── configmap.yaml
        ├── stg
        │   ├── kustomization.yaml
        │   └── configmap.yaml
        └── prod
            ├── kustomization.yaml
            └── configmap.yaml

---

### 13. How Kustomize works

The Kustomize workflow is simple:

- You define a base configuration
- You define overlays for each environment
- Kustomize combines them together

Conceptually, it looks like this:

BASE + OVERLAYS → FINAL Kubernetes manifest

---

### 14. Applying the final configuration

Kustomize builds the final YAML and hands it to Kubernetes.

You can:
- Generate the final YAML
- Apply it directly to the cluster

This ensures each environment gets the correct configuration without duplication.

---

### 15. Tooling and simplicity

Important points about Kustomize:

- Kustomize comes built in with kubectl
- No additional installation is required for basic usage
- Installing the standalone Kustomize CLI gives access to newer versions
- Kustomize does not require learning a templating language
- All configuration files are plain YAML
- Every file can be validated and processed normally
