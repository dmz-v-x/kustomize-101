## Overlay Walkthruogh

### 1. Full overlay walkthrough: dev, staging, and production

In this blog, we will put **everything together**.

So far, you have learned:
- What Kustomize is
- Base and overlay concepts
- kustomization.yaml fields
- Build and apply workflow
- Transformers
- Patches and their edge cases

Now it is time to see **a complete, real-world overlay workflow** from start to finish.

This blog walks through:
- One application
- One base configuration
- Three overlays: dev, staging, prod
- How changes flow from base to overlays
- How Kustomize builds final manifests per environment

---

### 2. Scenario and goal

We will deploy a simple Nginx application.

Requirements:
- Same application across all environments
- Different replica counts per environment
- Different image tags per environment
- No duplication of YAML files

Environment requirements:

- dev: 1 replica, nginx:latest
- staging: 2 replicas, nginx:1.21
- prod: 5 replicas, nginx:1.18

---

### 3. Project directory structure

Here is the complete directory layout:

    k8s/
    ├── base/
    │   ├── deployment.yaml
    │   ├── service.yaml
    │   └── kustomization.yaml
    └── overlays/
        ├── dev/
        │   ├── kustomization.yaml
        │   └── patch.yaml
        ├── staging/
        │   ├── kustomization.yaml
        │   └── patch.yaml
        └── prod/
            ├── kustomization.yaml
            └── patch.yaml

Each directory with a kustomization.yaml is a build entry point.

---

### 4. Base deployment configuration

The base Deployment defines **shared defaults**.

File: base/deployment.yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-app
      labels:
        app: nginx-app
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: nginx-app
      template:
        metadata:
          labels:
            app: nginx-app
        spec:
          containers:
            - name: nginx
              image: nginx:latest
              ports:
                - containerPort: 80

This file contains no environment-specific values.

---

### 5. Base service configuration

File: base/service.yaml

    apiVersion: v1
    kind: Service
    metadata:
      name: nginx-service
    spec:
      selector:
        app: nginx-app
      ports:
        - port: 80
          targetPort: 80
      type: ClusterIP

---

### 6. Base kustomization.yaml

File: base/kustomization.yaml

    resources:
      - deployment.yaml
      - service.yaml

The base kustomization only lists shared resources.

---

### 7. Dev overlay requirements

Dev environment goals:
- Minimal replicas
- Fast iteration
- No heavy resource usage

We will:
- Keep replicas at 1
- Use nginx:latest
- Add a name prefix for clarity

---

### 8. Dev overlay patch

File: overlays/dev/patch.yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-app
    spec:
      replicas: 1
      template:
        spec:
          containers:
            - name: nginx
              image: nginx:latest

---

### 9. Dev overlay kustomization.yaml

File: overlays/dev/kustomization.yaml

    resources:
      - ../../base

    namePrefix: dev-

    patchesStrategicMerge:
      - patch.yaml

---

### 10. Building the dev configuration

To generate the dev manifests:

    kustomize build overlays/dev

This produces:
- dev-nginx-app Deployment
- dev-nginx-service Service
- 1 replica
- nginx:latest image

---

### 11. Staging overlay requirements

Staging environment goals:
- Closer to production
- Moderate scale
- Stable image version

We will:
- Set replicas to 2
- Use nginx:1.21
- Add a name prefix

---

### 12. Staging overlay patch

File: overlays/staging/patch.yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-app
    spec:
      replicas: 2
      template:
        spec:
          containers:
            - name: nginx
              image: nginx:1.21

---

### 13. Staging overlay kustomization.yaml

File: overlays/staging/kustomization.yaml

    resources:
      - ../../base

    namePrefix: staging-

    patchesStrategicMerge:
      - patch.yaml

---

### 14. Building the staging configuration

To generate the staging manifests:

    kustomize build overlays/staging

This produces:
- staging-nginx-app Deployment
- 2 replicas
- nginx:1.21 image

---

### 15. Prod overlay requirements

Production environment goals:
- High availability
- Stable and tested image
- Higher replica count

We will:
- Set replicas to 5
- Use nginx:1.18
- Add a name prefix

---

### 16. Prod overlay patch

File: overlays/prod/patch.yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-app
    spec:
      replicas: 5
      template:
        spec:
          containers:
            - name: nginx
              image: nginx:1.18

---

### 17. Prod overlay kustomization.yaml

File: overlays/prod/kustomization.yaml

    resources:
      - ../../base

    namePrefix: prod-

    patchesStrategicMerge:
      - patch.yaml

---

### 18. Building the production configuration

To generate the production manifests:

    kustomize build overlays/prod

This produces:
- prod-nginx-app Deployment
- 5 replicas
- nginx:1.18 image

---

### 19. Applying an overlay to the cluster

To deploy any environment, you choose the overlay directory.

Example for production:

    kustomize build overlays/prod | kubectl apply -f -

This applies only the production configuration.

---

### 20. How changes propagate

If you update base/deployment.yaml:
- All overlays automatically inherit the change
- No patch updates required

If you update an overlay patch:
- Only that environment is affected

This separation is the key benefit of Kustomize.

---

### 21. Why this workflow scales

This approach:
- Avoids YAML duplication
- Keeps environment differences explicit
- Makes reviews easier
- Reduces configuration drift
- Fits perfectly with Git-based workflows

