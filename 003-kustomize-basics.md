## Kustomize Basics

### 1. Using Kustomize

After understanding what Kustomize is and how it integrates with kubectl, the next step is to **actually use it**.

The entry point to using Kustomize is the `kustomization.yaml` file.

---

### 2. Creating a kustomization file

In a directory containing Kubernetes YAML resource files such as Deployments and Services, you create a file named `kustomization.yaml`.

This file tells Kustomize:

- Which resources to include
- What customizations to apply on top of those resources

Example directory structure:

~/someApp
├── deployment.yaml
├── service.yaml
└── kustomization.yaml

---

### 3. Role of each file

Each file in this directory has a specific responsibility.

deployment.yaml  
Describes how your application runs.  
It defines things like:
- Container image
- Number of replicas
- Ports
- Environment variables

service.yaml  
Defines how your application is exposed.  
It allows other services or users to connect to your app.

kustomization.yaml  
Acts as the control file for Kustomize.  
It references the other YAML files and defines customizations such as:
- Common labels
- Name prefixes
- Image overrides

---

### 4. Why Kustomize is useful in this setup

Kustomize allows you to customize Kubernetes configurations **without modifying the original YAML files**.

This is especially useful when:

- You copied configuration from another repository
- The original source continues to evolve
- You want to pull upstream changes without conflicts

Your customizations live separately and are layered on top.

---

### 5. Building configuration with Kustomize

To generate the final Kubernetes YAML, you run:

kustomize build ~/someApp

This command:

- Reads `kustomization.yaml`
- Loads referenced resource files
- Applies all customizations
- Outputs a single, combined YAML

At this stage, **nothing is applied to the cluster**.

---

### 6. Applying the generated configuration

To deploy the application, you pipe the output to kubectl:

kustomize build ~/someApp | kubectl apply -f -

This does two things:

1. Builds the final YAML using Kustomize
2. Applies it to the Kubernetes cluster using kubectl

This is the standard Kustomize deployment flow.

---

### 7. Managing multiple environments with overlays

Once basic usage is clear, the next step is handling **multiple environments**.

Typical environments include:
- Development
- Staging
- Production

Instead of duplicating YAML files, Kustomize uses **bases and overlays**.

---

### 8. Base and overlay directory structure

Example structure:

    ~/someApp
    ├── base
    │ ├── deployment.yaml
    │ ├── service.yaml
    │ └── kustomization.yaml
    └── overlays
    ├── development
    │ ├── cpu_count.yaml
    │ ├── replica_count.yaml
    │ └── kustomization.yaml
    └── production
    ├── cpu_count.yaml
    ├── replica_count.yaml
    └── kustomization.yaml

---

### 9. What is a base

The base directory contains:

- Standard Kubernetes manifests
- Environment-agnostic configuration
- Often maintained by another team or upstream repository

The base should be reusable and unchanged across environments.

---

### 10. What is an overlay

An overlay is another `kustomization.yaml` that:

- References the base
- Applies patches on top of it
- Changes only what is environment-specific

Examples of overlay changes:
- Replica count
- CPU limits
- Memory limits
- Environment variables

---

### 11. Why this structure works well with git

This layout makes version control easier:

- The base can be pulled from upstream without conflicts
- Overlays remain small and focused
- No duplication of large YAML files
- No need for git submodules

Each environment evolves independently while sharing a common base.

---

### 12. Building and applying an overlay

To generate production configuration:

kustomize build ~/someApp/overlays/production

To apply it to the cluster:

kustomize build ~/someApp/overlays/production | kubectl apply -f -

Only the production-specific changes are applied on top of the base.
