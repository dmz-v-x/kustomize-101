## Kustomize Transformers

### 1. Introduction to Kustomize Transformers

In Kustomize, **transformers** are the mechanisms that modify Kubernetes manifests during the build process.

They take the original Kubernetes YAML files and transform them based on rules defined in `kustomization.yaml`.

Important idea:

- You do not write transformer code directly
- You enable transformers by using specific fields in `kustomization.yaml`
- Kustomize applies these transformations automatically during `kustomize build`

Transformers allow you to customize resources without editing the original YAML files.

---

### 2. What transformers do in Kustomize

Transformers are responsible for:

- Renaming resources
- Adding labels and annotations
- Modifying existing fields
- Updating container images
- Injecting generated ConfigMaps and Secrets

They operate **after resources are loaded** and **before final YAML is produced**.

---

### 3. Types of transformers in Kustomize

Commonly used transformer categories include:

- Name prefix and suffix transformers
- Label and annotation transformers
- Patch transformers
- ConfigMap and Secret generators
- Image transformers

Each of these is controlled by fields inside `kustomization.yaml`.

---

### 4. NamePrefix and NameSuffix transformers

Name transformers modify the names of Kubernetes resources such as:

- Deployments
- Services
- ConfigMaps
- Secrets

This is useful for:
- Environment separation
- Avoiding naming conflicts
- Clear resource identification

---

### 5. Using namePrefix

You can add a prefix to all resource names using `namePrefix`.

Example kustomization.yaml:

    namePrefix: dev-

    resources:
      - deployment.yaml
      - service.yaml

---

### 6. Original Deployment resource

Example deployment.yaml:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-app
    spec:
      replicas: 3
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

---

### 7. Result after applying namePrefix

After running `kustomize build`, the generated resource becomes:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: dev-my-app
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: dev-my-app
      template:
        metadata:
          labels:
            app: dev-my-app
        spec:
          containers:
            - name: dev-my-app
              image: nginx:latest
              ports:
                - containerPort: 80

Kustomize automatically updates:
- Resource names
- Selectors
- Labels

This prevents broken references.

---

### 8. Label and annotation transformers

Kustomize allows you to add labels and annotations to all resources using:

- `commonLabels`
- `commonAnnotations`

These transformers apply metadata uniformly across all resources.

---

### 9. Using commonLabels and commonAnnotations

Example kustomization.yaml:

    commonLabels:
      app: my-app
      environment: production

    commonAnnotations:
      createdBy: kustomize
      version: v1.0

    resources:
      - deployment.yaml
      - service.yaml

---

### 10. Resulting resource metadata

After transformation, resources will include:

- Labels for selection and grouping
- Annotations for metadata and tooling

Labels are typically used by Kubernetes and selectors.  
Annotations are used by humans and external systems.

---

### 11. Patch transformers

Patch transformers modify existing resources without rewriting them entirely.

They allow you to:
- Override specific fields
- Customize behavior per environment
- Avoid YAML duplication

The most common patch type is **strategic merge patch**.

---

### 12. Using patchesStrategicMerge

Example kustomization.yaml:

    patchesStrategicMerge:
      - patch-replicas.yaml

    resources:
      - deployment.yaml

---

### 13. Patch file example

Example patch-replicas.yaml:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-app
    spec:
      replicas: 5

This patch overrides only the replicas field.

---

### 14. Result after applying the patch

The generated Deployment will now contain:

    spec:
      replicas: 5

All other fields remain unchanged.

This makes patch transformers ideal for environment-specific overrides.

---

### 15. ConfigMap and Secret generators

Generators create new Kubernetes resources instead of modifying existing ones.

Kustomize provides:
- `configMapGenerator`
- `secretGenerator`

These are commonly used alongside transformers.

---

### 16. ConfigMap generator example

Example kustomization.yaml:

    configMapGenerator:
      - name: my-app-config
        literals:
          - ENV=production
          - DEBUG=false

    resources:
      - deployment.yaml

Kustomize generates a ConfigMap automatically.

---

### 17. Generated ConfigMap result

The generated ConfigMap looks like:

    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: my-app-config
    data:
      ENV: production
      DEBUG: "false"

By default, Kustomize appends a hash suffix to track changes.

---

### 18. Secret generator example

Example kustomization.yaml:

    secretGenerator:
      - name: my-app-secret
        literals:
          - SECRET_KEY=supersecretkey
          - DB_PASSWORD=secretpassword

This generates a Kubernetes Secret.

---

### 19. Generated Secret result

The generated Secret will look like:

    apiVersion: v1
    kind: Secret
    metadata:
      name: my-app-secret
    type: Opaque
    data:
      SECRET_KEY: c3VwZXJzZWNyZXRrZXk=
      DB_PASSWORD: c2VjcmV0cGFzc3dvcmQ=

Values are base64-encoded automatically.

---

### 20. Image transformers

Image transformers allow you to update container images without editing YAML files.

They are commonly used to:
- Change image tags
- Switch image registries
- Deploy different versions per environment

---

### 21. Using the images field

Example kustomization.yaml:

    images:
      - name: nginx
        newName: nginx
        newTag: 1.18.0

This updates all references to the nginx image.

---

### 22. Image transformation result

Before transformation:

    image: nginx:latest

After transformation:

    image: nginx:1.18.0

This works across all resources that reference the image.

---

### 23. Why transformers are powerful

Transformers allow you to:

- Keep base YAML clean and reusable
- Apply environment-specific changes safely
- Avoid copy-paste configurations
- Maintain predictable, declarative behavior

They are a core reason why Kustomize scales well.
