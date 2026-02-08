## Strategic Merge Patches

### 1. Strategic Merge Patches: deeper understanding and edge cases

In the previous blog, you learned **what patches are** and how to use **strategic merge patches** at a high level.

Now it is time to go one level deeper.

This blog focuses on **how strategic merge patches actually work under the hood**, the rules they follow, and the **common edge cases and failures** that confuse people in real projects.

This topic is critical because:
- Strategic merge patches can silently fail
- Small mistakes can cause patches to not apply at all
- Debugging becomes hard without understanding the rules

---

### 2. How Kustomize matches a patch to a resource

For a strategic merge patch to apply, **Kustomize must be able to match the patch to exactly one resource**.

Matching is done using these fields:

- apiVersion
- kind
- metadata.name
- metadata.namespace (if present)

If **any of these do not match**, the patch will not be applied.

Important rule:

If Kustomize cannot find a matching resource, it does NOT throw an error.  
The patch is simply ignored.

This is the number one source of confusion.

---

### 3. Example of correct resource matching

Base Deployment:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-app
      namespace: default
    spec:
      replicas: 2

Patch file:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-app
      namespace: default
    spec:
      replicas: 5

Because all identifying fields match, the patch is applied correctly.

---

### 4. Common matching failure cases

Strategic merge patches fail silently when:

- metadata.name does not match exactly
- apiVersion is incorrect
- kind is incorrect
- namespace is missing or different
- The resource exists in a different overlay than expected

Example of a failing patch:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: myapp   # does not match "my-app"
    spec:
      replicas: 5

This patch will NOT apply.

---

### 5. How lists are handled in strategic merge patches

Strategic merge patches are **Kubernetes-aware**.

This means lists are not always replaced.  
Instead, they are **merged using a merge key**.

For many Kubernetes resources, the merge key is `name`.

---

### 6. Container list merging behavior

Containers are defined as a list.

Base Deployment:

    spec:
      template:
        spec:
          containers:
            - name: my-app
              image: nginx:latest
              ports:
                - containerPort: 80

Patch file:

    spec:
      template:
        spec:
          containers:
            - name: my-app
              image: nginx:1.18.0

Because the container name matches, only the image field is updated.

Result:

    containers:
      - name: my-app
        image: nginx:1.18.0
        ports:
          - containerPort: 80

---

### 7. What happens when the merge key is missing

If the merge key is missing or incorrect, Kustomize cannot merge the list item.

Example patch with missing container name:

    spec:
      template:
        spec:
          containers:
            - image: nginx:1.18.0

This causes the entire containers list to be replaced or ignored, depending on the resource type.

This is a common and dangerous mistake.

---

### 8. Lists that are replaced instead of merged

Not all lists support strategic merging.

Some lists are replaced entirely.

Examples include:
- env
- args
- command

Base:

    env:
      - name: ENV
        value: dev

Patch:

    env:
      - name: ENV
        value: prod

Result:

    env:
      - name: ENV
        value: prod

This behavior is expected but often surprising.

---

### 9. Deleting fields with strategic merge patches

Strategic merge patches can also delete fields.

This is done using the `$patch` directive.

Example:

    metadata:
      labels:
        app: my-app
        env: prod

Patch to delete a label:

    metadata:
      labels:
        env: null

The `env` label will be removed.

---

### 10. Deleting list items with `$patch: delete`

To delete an item from a list:

    spec:
      template:
        spec:
          containers:
            - name: sidecar
              $patch: delete

This removes the container named `sidecar`.

---

### 11. Order of patch application

When multiple strategic merge patches are defined:

- Patches are applied in the order they are listed
- Later patches can override earlier patches

Example:

    patchesStrategicMerge:
      - patch-replicas.yaml
      - patch-image.yaml

Understanding patch order is important when multiple overrides exist.

---

### 12. Why patches silently fail

Strategic merge patches fail silently because:

- Kustomize assumes missing matches are intentional
- This allows flexible composition
- But it shifts responsibility to the user

This is why inspecting build output is mandatory.

---

### 13. How to debug patch issues

Best debugging practices:

- Always run `kustomize build` before applying
- Inspect the generated YAML carefully
- Confirm resource names after namePrefix or nameSuffix
- Check namespaces
- Verify container names exactly

Never assume a patch worked without checking output.

---

### 14. When to prefer JSON patches instead

Use JSON Patch when:

- You need to delete a specific array index
- You need absolute precision
- Strategic merge behavior is unclear
- You want explicit add/remove operations

Strategic merge is safer by default, but JSON Patch is more explicit.
