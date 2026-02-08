## Kustomize Patches

### 1. What are patches in Kustomize

In Kustomize, **patches** are a core mechanism used to modify Kubernetes resources **after** they are defined in base configurations.

Instead of copying and rewriting entire YAML files, patches allow you to:
- Override specific fields
- Add new configuration
- Remove existing configuration

All without duplicating the original resource definition.

Patches are essential when working with:
- Multiple environments like dev, staging, and prod
- Small environment-specific differences
- Large shared base configurations

---

### 2. Why patches are needed

In real-world Kubernetes setups:

- Most configuration is shared across environments
- Only a few values differ, such as:
  - replica counts
  - image tags
  - environment variables
  - labels or annotations

Without patches, you would be forced to duplicate YAML files for each environment.  
Patches solve this by applying **focused changes** on top of existing resources.

---

### 3. Types of patches in Kustomize

Kustomize supports two primary patch mechanisms:

1. Strategic Merge Patch  
2. JSON Patch (RFC 6902)

Each patch type exists to solve a different kind of problem.

---

### 4. Strategic Merge Patch

Strategic Merge Patch is the **most commonly used** patch type in Kustomize.

It works by merging changes into an existing Kubernetes resource using Kubernetes API merge rules.

Key characteristics:

- Only the specified fields are modified
- Existing fields are preserved
- Missing fields are added
- Lists are merged instead of blindly replaced

This makes strategic merge patches ideal for most environment-specific customizations.

---

### 5. How strategic merge patch works

When Kustomize applies a strategic merge patch:

- If a field already exists, it is updated
- If a field does not exist, it is added
- If the field is a list, items are merged using identifiers like `name`

This behavior follows Kubernetes-native merge semantics.

---

### 6. Base Deployment resource

Example base Deployment defined in `base/deployment.yaml`:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-app
      labels:
        app: my-app
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: my-app
      template:
        metadata:
          labels:
            app: my-app
        spec:
          containers:
            - name: my-app
              image: nginx:latest
              ports:
                - containerPort: 80

This Deployment represents the shared configuration used across environments.

---

### 7. Creating a strategic merge patch for production

Now suppose we want production to:

- Run more replicas
- Use a different image version

We create a patch file in `prod/patch-replicas.yaml`:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-app
    spec:
      replicas: 5
      template:
        spec:
          containers:
            - name: my-app
              image: nginx:1.18.0

Only the fields that need to change are included.

---

### 8. Applying the strategic merge patch

In the production overlay `prod/kustomization.yaml`:

    resources:
      - ../../base

    patchesStrategicMerge:
      - patch-replicas.yaml

This tells Kustomize to:
- Load the base resources
- Apply the patch on top of them

---

### 9. Result after applying strategic merge patch

After running `kustomize build`, the resulting Deployment becomes:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-app
      labels:
        app: my-app
    spec:
      replicas: 5
      selector:
        matchLabels:
          app: my-app
      template:
        metadata:
          labels:
            app: my-app
        spec:
          containers:
            - name: my-app
              image: nginx:1.18.0
              ports:
                - containerPort: 80

Only the intended fields are modified.  
Everything else remains unchanged.

---

### 10. When to use strategic merge patches

Use strategic merge patches when:

- You want to override configuration cleanly
- You are modifying common Kubernetes fields
- You want Kubernetes-aware merge behavior
- You are working with Deployments, Services, StatefulSets, etc.

This should be your **default patching choice**.

---

### 11. JSON Patch

JSON Patch provides a **more fine-grained** and **explicit** way to modify resources.

It follows the RFC 6902 standard and allows you to:
- Add fields
- Remove fields
- Replace values
- Modify specific array positions

JSON Patch is useful when strategic merge patch is not precise enough.

---

### 12. How JSON Patch works

A JSON Patch consists of a list of operations.

Each operation contains:

- op: the operation type (add, remove, replace)
- path: the JSON path to the field
- value: the new value (for add and replace)

Operations are applied in order.

---

### 13. JSON Patch example scenario

Suppose in production we want to:

- Add an environment variable to the container
- Remove a label from metadata

This requires precise control, making JSON Patch a good choice.

---

### 14. JSON Patch file example

Create a JSON patch file `prod/patch-env-variable.json`:

    [
      {
        "op": "add",
        "path": "/spec/template/spec/containers/0/env",
        "value": [
          {
            "name": "ENV",
            "value": "production"
          }
        ]
      },
      {
        "op": "remove",
        "path": "/metadata/labels/app"
      }
    ]

This patch:
- Adds an ENV environment variable
- Removes the `app` label from metadata

---

### 15. Applying JSON Patch in kustomization.yaml

In `prod/kustomization.yaml`:

    resources:
      - ../../base

    patchesJson6902:
      - target:
          kind: Deployment
          name: my-app
        path: patch-env-variable.json

The target ensures the patch applies only to the intended resource.

---

### 16. Result after applying JSON Patch

After running `kustomize build`, the resulting Deployment includes:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-app
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: my-app
      template:
        metadata:
          labels: {}
        spec:
          containers:
            - name: my-app
              image: nginx:latest
              ports:
                - containerPort: 80
              env:
                - name: ENV
                  value: production

The JSON Patch operations are applied exactly as defined.

---

### 17. Strategic merge vs JSON patch

Strategic Merge Patch:
- Kubernetes-aware
- Safer defaults
- Easier to read
- Best for most cases

JSON Patch:
- Very precise
- More verbose
- Array-index based
- Best for advanced use cases

Both are important, but they serve different needs.
