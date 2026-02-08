## Kustomize File & It's components

### 1. Introduction to kustomization.yaml

In Kustomize, kustomization.yaml is the central control file.

It does not describe a Kubernetes resource itself.  
Instead, it tells Kustomize:

- Which Kubernetes resources to include
- How those resources should be customized
- How the final manifests should be generated

You can think of kustomization.yaml as the instruction file that Kustomize follows to build final Kubernetes YAML.

---

### 2. What kustomization.yaml controls

A kustomization.yaml file controls how Kubernetes manifests are assembled and customized.

Using this file, you can:

- Select which resource YAML files are part of your application
- Apply naming conventions
- Set namespaces
- Add labels and annotations
- Modify existing resources using patches
- Generate ConfigMaps and Secrets dynamically

Some fields are required, while most are optional.

---

### 3. resources (Required)

The resources field is the most important and foundational part of kustomization.yaml.

Purpose  
Specifies the Kubernetes resource YAML files that Kustomize should manage.

Type  
A list of file paths, directories, or URLs.

Example with local files

    resources:
      - deployment.yaml
      - service.yaml

Kustomize will load these files and include them in the final output.

---

### 4. Including directories and remote resources

Resources can also be grouped in directories or referenced from remote URLs.

    resources:
      - ./base
      - https://raw.githubusercontent.com/kubernetes/kubernetes/master/examples/nginx-deployment.yaml

This allows:

- Reuse of base configurations
- Consumption of upstream manifests
- Cleaner and more scalable project structure

---

### 5. namePrefix (Optional)

Purpose  
Adds a prefix to the names of all resources.

Type  
String

Example

    namePrefix: prod-

A resource named nginx-deployment becomes prod-nginx-deployment.

This is commonly used to avoid name collisions and clearly identify environment-specific resources.

---

### 6. nameSuffix (Optional)

Purpose  
Adds a suffix to the names of all resources.

Type  
String

Example

    nameSuffix: -v1

A resource named nginx-deployment becomes nginx-deployment-v1.

---

### 7. namespace (Optional)

Purpose  
Assigns a namespace to all resources managed by this kustomization.

Type  
String

Example

    namespace: my-namespace

This applies the namespace uniformly without modifying individual resource files.

---

### 8. commonLabels (Optional)

Purpose  
Adds labels to all resources defined in the kustomization.

Type  
Key-value map

Example

    commonLabels:
      app: my-app
      environment: production

These labels are applied to every resource and are commonly used for selection, monitoring, and organization.

---

### 9. commonAnnotations (Optional)

Purpose  
Adds annotations to all resources defined in the kustomization.

Type  
Key-value map

Example

    commonAnnotations:
      team: devops
      version: v1.2.0

Annotations are used for metadata and are not intended for selection.

---

### 10. patchesStrategicMerge (Optional)

Purpose  
Modifies existing resources by merging changes into them.

This is the most commonly used and beginner-friendly patching mechanism.

Type  
List of patch files

Example

    patchesStrategicMerge:
      - patch-deployment.yaml

Only the fields specified in the patch file are modified.  
All other fields remain unchanged.

---

### 11. patchesJson6902 (Optional)

Purpose  
Applies patches using the JSON 6902 standard.

This method provides very precise control over resource modification.

Type  
List of patch definitions

Example

    patchesJson6902:
      - target:
          version: apps/v1
          kind: Deployment
          name: nginx-deployment
        patch: |-
          - op: replace
            path: /spec/replicas
            value: 3

This explicitly replaces the replicas field of the targeted Deployment.

---

### 12. configMapGenerator (Optional)

Purpose  
Generates ConfigMaps dynamically from literals or files.

This avoids manually writing large ConfigMap YAML files.

Type  
List of generators

Example

    configMapGenerator:
      - name: app-config
        literals:
          - DATABASE_URL=postgres://localhost:5432
        files:
          - ./config.json

Kustomize generates a ConfigMap named app-config using the provided data.

---

### 13. secretGenerator (Optional)

Purpose  
Generates Secrets dynamically from literals or files.

Type  
List of generators

Example

    secretGenerator:
      - name: app-secret
        literals:
          - PASSWORD=my-secret-password
        files:
          - ./secret-file.txt

This creates a Kubernetes Secret using the provided values.

---

### 14. Full example kustomization.yaml

    apiVersion: kustomize.config.k8s.io/v1beta1
    kind: Kustomization

    resources:
      - deployment.yaml
      - service.yaml

    namePrefix: prod-
    namespace: production

    commonLabels:
      app: my-app
      environment: production

    patchesStrategicMerge:
      - patch-deployment.yaml

    configMapGenerator:
      - name: app-config
        literals:
          - DATABASE_URL=postgres://localhost:5432

    secretGenerator:
      - name: app-secret
        literals:
          - PASSWORD=my-secret-password

This configuration:

- Loads resource files
- Applies naming and namespace rules
- Adds labels
- Applies patches
- Generates ConfigMaps and Secrets



With this foundation, the next step is to go deeper into patches and overlays with real-world examples.
