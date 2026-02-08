## Installing Kustomize

### 1. Kustomize bundled with kubectl

An important thing to understand is that **Kustomize already comes bundled with kubectl**.

Starting from **kubectl version 1.14**, Kustomize is built directly into the Kubernetes CLI.  
Because of this, for most day-to-day use cases, **you do not need to install Kustomize separately**.

This design makes Kustomize feel like a native part of Kubernetes rather than an external tool.

---

### 2. What bundled actually means

When we say Kustomize is bundled with kubectl, it means:

- kubectl already understands `kustomization.yaml`
- kubectl can build Kustomize configurations internally
- kubectl can apply Kustomize output directly to the cluster

You are still using Kustomize concepts, but you are invoking them through kubectl.

---

### 3. Applying configurations using kubectl and Kustomize

With bundled Kustomize, you can directly apply configurations using this command:

    kubectl apply -k <directory>

What happens internally:

1. kubectl detects a `kustomization.yaml` file
2. Kustomize builds the final YAML
3. kubectl applies the generated YAML to the cluster

You do not see the generated YAML unless you explicitly ask for it.

---

### 4. Viewing generated YAML without applying

Sometimes you want to **see what Kustomize generates** before applying it.

You can do this using:

    kubectl kustomize <directory>

This command:

- Runs Kustomize internally
- Prints the final YAML to the terminal
- Does not apply anything to the cluster

This is very useful for debugging and learning.

---

### 5. When you do not need a separate installation

You do **not** need to install Kustomize separately when:

- You are using kubectl version 1.14 or later
- You are doing basic to intermediate Kustomize usage
- You are comfortable with the Kustomize version shipped with kubectl

For most beginners and many production setups, this is completely sufficient.

---

### 6. When a standalone Kustomize installation is needed

A separate standalone Kustomize binary is required only in specific cases:

- You need a **newer Kustomize version** than what kubectl ships
- You want access to **latest features or bug fixes**
- You want better control over versioning
- You want to build YAML without kubectl installed

In these cases, installing standalone Kustomize makes sense.

---

### 7. Built-in vs standalone comparison

Built-in Kustomize via kubectl:

- No extra installation
- Simple workflow
- Tied to kubectl version

Standalone Kustomize binary:

- Independent versioning
- More control
- Better for advanced workflows and CI pipelines

Both use the same `kustomization.yaml` structure.

---

### 8. Verifying bundled Kustomize support

You can verify whether bundled Kustomize is available by running:

    kubectl kustomize --help

If Kustomize support exists, you will see help output describing the command.

This confirms:

- kubectl recognizes Kustomize
- You can work with `kustomization.yaml` files

---

### 9. Common beginner confusion

A common misunderstanding is thinking:

- kubectl and Kustomize are always separate tools

In reality:

- Kustomize is a standalone tool
- kubectl embeds Kustomize internally
- You choose how you want to use it

Understanding this early prevents confusion later.

