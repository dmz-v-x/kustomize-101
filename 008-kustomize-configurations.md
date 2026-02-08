## Kustomize Configurations

### 1. Build and apply customized configurations

Once you have created your kustomization.yaml file and the required Kubernetes resource files such as deployment.yaml and service.yaml, the next step is to **build** and **apply** your customized configuration.

This step connects everything you have learned so far:
- kustomization.yaml
- resources
- patches
- generators

At this stage, Kustomize is used to **generate final Kubernetes manifests**, and kubectl is used to **apply them to the cluster**.

---

### 2. Building the Kustomize configuration

To build the Kubernetes manifests defined by your kustomization.yaml file, run the following command from the directory where kustomization.yaml exists.

    kustomize build .

The dot (`.`) represents the current directory.

What this command does:

- Reads the kustomization.yaml file
- Loads all referenced resources
- Applies patches, prefixes, labels, generators, and other customizations
- Outputs a single combined Kubernetes YAML

Important point:

- This command does NOT apply anything to the cluster
- It only generates the final YAML

This makes it safe to run and inspect at any time.

---

### 3. Inspecting the generated output

Since `kustomize build` prints YAML to standard output, you can:

- View it directly in the terminal
- Redirect it to a file for inspection

Example:

    kustomize build . > final.yaml

This allows you to:
- Review exactly what will be applied
- Debug configuration issues
- Understand how Kustomize transforms your resources

---

### 4. Applying the generated configuration

Once you are satisfied with the generated YAML, you can apply it to your Kubernetes cluster.

    kustomize build . | kubectl apply -f -

Explanation:

- `kustomize build .` generates the final YAML
- `kubectl apply -f -` applies YAML from standard input
- The `-` tells kubectl to read from the pipe

This is the most common and recommended workflow.

---

### 5. Declarative update behavior

If you later make changes to:
- deployment.yaml
- service.yaml
- kustomization.yaml
- patches
- generators

You do NOT need a new command.

Simply run the same command again:

    kustomize build . | kubectl apply -f -

Kubernetes will:
- Compare desired state with current state
- Apply only the necessary changes
- Leave unchanged resources untouched

This is the declarative model in action.

---

### 6. Using kubectl with built-in Kustomize

Modern versions of kubectl include Kustomize support.

You can generate manifests using:

    kubectl kustomize .

This command behaves the same as:

    kustomize build .

It is useful if:
- You want to rely only on kubectl
- You do not have the standalone kustomize binary installed

---

### 7. Applying directly using kubectl

kubectl also supports applying Kustomize configurations directly.

    kubectl apply -k .

What happens internally:

- kubectl runs Kustomize
- It builds the final YAML
- It applies the resources to the cluster

This is a shortcut that combines build and apply into a single step.

---

### 8. Debugging with dry-run

Before applying changes to a real cluster, it is often useful to perform a dry run.

    kustomize build . | kubectl apply -f - --dry-run=client

What this does:

- Builds the final YAML
- Simulates applying it
- Does NOT modify the cluster

This allows you to:
- Validate resource definitions
- Catch mistakes early
- Safely test changes

---

### 9. Why build and apply is important

Understanding this workflow is critical because:

- Kustomize never talks to the cluster directly
- kubectl is always responsible for applying changes
- You can inspect and debug output at every step
- The same commands work across all environments

This separation keeps deployments predictable and safe.
