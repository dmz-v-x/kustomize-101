## Kustomize Directories

### 1. Managing directories in Kustomize

As Kubernetes projects grow, managing all configuration files in a single directory quickly becomes unmanageable.

Kustomize solves this problem by allowing you to organize configuration into **directories**, each controlled by its own kustomization.yaml file.  
This directory-based design is one of Kustomize’s most powerful features.

It enables:
- Clean separation of concerns
- Reuse of shared configuration
- Environment-specific customization without duplication

---

### 2. How Kustomize uses directories

Kustomize works by **recursively reading directories** that contain a kustomization.yaml file.

Each directory that has a kustomization.yaml becomes a **configuration unit**.

You do not point Kustomize at individual YAML files.  
You point it at a directory, and Kustomize figures out everything from there.

---

### 3. The base and overlay model

Kustomize directory management is built around two main ideas:

- Base directories
- Overlay directories

This model allows you to share common configuration while customizing behavior per environment.

---

### 4. Base directory: shared resources

The **base directory** contains Kubernetes resources that are common across all environments.

Typical contents of a base directory include:
- Deployments
- Services
- Default configuration
- Shared labels and metadata

The base directory always contains its own kustomization.yaml file.

Example base directory structure:

    base/
    ├── deployment.yaml
    ├── service.yaml
    └── kustomization.yaml

The kustomization.yaml in the base directory lists only the shared resources.

---

### 5. Purpose of the base directory

The base directory represents:
- Reusable configuration
- Environment-agnostic defaults
- A stable foundation that overlays can build on

The base should not contain environment-specific values like replica counts for production or staging.

---

### 6. Overlay directories: environment-specific customization

Overlay directories customize the base configuration for a specific environment.

Each environment gets its own overlay directory, such as:
- dev
- staging
- prod

Each overlay directory:
- Has its own kustomization.yaml
- References the base directory
- Applies patches and overrides

---

### 7. Overlay directory structure

A typical overlay directory contains:
- kustomization.yaml
- Patch files
- Environment-specific ConfigMaps or Secrets

Example overlay directory structure:

    overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   ├── patch-dev.yaml
    │   └── config-map-dev.yaml
    ├── staging/
    │   ├── kustomization.yaml
    │   ├── patch-staging.yaml
    │   └── config-map-staging.yaml
    └── prod/
        ├── kustomization.yaml
        ├── patch-prod.yaml
        └── config-map-prod.yaml

---

### 8. How overlays reference the base

Inside an overlay’s kustomization.yaml, the base directory is referenced using the resources field.

Example overlay kustomization.yaml:

    resources:
      - ../../base

This tells Kustomize:
- Start with everything defined in the base
- Then apply the overlay’s customizations on top

---

### 9. What belongs in overlays

Overlays typically contain:
- Replica count changes
- Resource limits
- Image tags
- Environment-specific configuration
- Feature toggles

Only differences from the base should live in overlays.

---

### 10. Choosing an environment to deploy

In Kustomize, **each overlay directory is an entry point**.

You do not deploy all environments at once.  
You choose the environment by selecting the overlay directory.

For example, to build the dev configuration:

    kustomize build overlays/dev

To apply it to the cluster:

    kustomize build overlays/dev | kubectl apply -f -

---

### 11. Why overlays are entry points

This design ensures:
- Clear separation between environments
- No accidental deployment of multiple environments
- Simple and predictable workflows
- Clean integration with CI/CD pipelines

Each environment is built and deployed independently.

---

### 12. Why this directory model scales

This directory-based approach scales well because:
- Shared configuration lives in one place
- Environment differences stay small and focused
- Updates to the base automatically flow to all environments
- Git history remains clean and readable

This is why Kustomize is widely used in production systems.
