## Introduction to Kustomize

### 1. Introduction

When you start learning Kubernetes, the very first tools you hear about are **kubectl** and **Kustomize**.  
Before we go deep into Kustomize itself, it is important to clearly understand **what kubectl is**, **why it exists**, and **how Kustomize fits on top of it**.

---

### 2. What is kubectl

`kubectl` is the **command-line interface (CLI)** for Kubernetes.

In very simple words:

- Kubernetes is a system that runs and manages containers
- `kubectl` is the tool you use to **talk to Kubernetes**

Without `kubectl`, you cannot practically interact with a Kubernetes cluster.

---

### 3. What does kubectl do

`kubectl` allows you to interact with Kubernetes clusters.

Using `kubectl`, you can:

- Connect to a Kubernetes cluster
- Send instructions to the cluster
- Ask the cluster about its current state

Think of `kubectl` as a **remote control** for Kubernetes.

---

### 4. How kubectl is used

`kubectl` is used to **deploy and manage applications** on Kubernetes.

Some common things you do with `kubectl`:

- Deploy applications using YAML files
- Create, update, and delete Kubernetes resources
- Inspect running applications
- Debug problems inside the cluster

At a high level, the workflow looks like this:

1. You write Kubernetes configuration files in YAML
2. You use `kubectl` to apply those files
3. Kubernetes reads them and creates resources accordingly

---

### 5. The problem with plain YAML files

Kubernetes relies heavily on YAML configuration files.

Over time, you will notice some problems:

- You often deploy the **same app** to multiple environments
- Example environments: development, staging, production
- Most of the YAML stays the same
- Only small values change, like:
  - Image tags
  - Replicas count
  - Environment variables
  - Resource limits

Without a proper system, you end up with:

- Copied YAML files
- Huge duplication
- Difficult maintenance
- High chance of mistakes

This is where **Kustomize** comes in.

---

### 6. What is Kustomize

Kustomize is a tool that helps **customize Kubernetes configuration files** in a **template-free way**.

Important point:

- Kustomize works **directly with plain YAML**
- No placeholders
- No templating language
- No complex syntax

You start with **standard Kubernetes YAML**, and Kustomize modifies it safely.

---

### 7. Template-free customization

Unlike tools that use templates, Kustomize does **not** require you to change your original YAML files.

This means:

- Your base YAML files remain valid Kubernetes manifests
- You can still apply them directly using `kubectl apply`
- Customization happens on top of them

This is one of the biggest strengths of Kustomize.

---

### 8. Generators in Kustomize

Kustomize provides several **handy methods** to make customization easier.

One important concept is **generators**.

Generators allow you to:

- Generate ConfigMaps
- Generate Secrets
- Automatically inject them into your configuration

Instead of manually writing large YAML files, Kustomize can generate them for you based on simple inputs.

This reduces repetition and human error.

---

### 9. Patches in Kustomize

Kustomize uses **patches** to introduce environment-specific changes.

Key idea:

- You start with a **standard base configuration**
- You apply **small patches** for each environment
- The base file remains untouched

For example:

- Base configuration defines a Deployment
- Development patch changes replicas to 1
- Production patch changes replicas to 5

The original YAML never changes.

---

### 10. Environment-specific customization

Kustomize makes it easy to manage multiple environments.

You can have:

- One base configuration
- Multiple overlays
  - Development
  - Staging
  - Production

Each overlay only contains **what is different** for that environment.

This keeps your configuration:

- Clean
- Readable
- Easy to maintain

---

### 11. Core philosophy of Kustomize

The core idea behind Kustomize is very simple:

- Customize raw, template-free YAML files
- Support multiple purposes and environments
- Leave original YAML untouched and reusable

Because of this philosophy:

- Your base YAML stays clean
- Your changes are explicit
- Your configuration remains predictable
