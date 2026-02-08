## Resources & Controllers

### 1. Introduction

To understand how Kubernetes really works under the hood, you must understand **Resources** and **Controllers**.  
This mental model explains *why* Kubernetes behaves the way it does and *how* tools like kubectl and Kustomize fit into the system.

Kubernetes is not a scripting system.  
It is a **state-driven system**.

This blog explains that system step by step.

---

### 2. Kubernetes API high-level model

The Kubernetes API can be mentally divided into two major parts:

-> Resource Types  
-> Controllers  

These two pieces work together to run everything in a Kubernetes cluster.

- Resources describe **what you want**
- Controllers make **what you want actually happen**

---

### 3. What are Resources

Resources are **objects declared as JSON or YAML** and written to a Kubernetes cluster.

When you run a command like:

kubectl apply -f deployment.yaml

You are sending a **Resource definition** to the Kubernetes API server.

Once stored, that Resource becomes part of the cluster state.

---

### 4. Kubernetes objects as Resources

Instances of Kubernetes objects are called **Resources**.

Examples include:

- Deployments
- Services
- Namespaces
- ConfigMaps
- Secrets

Each time you create one of these, you are creating a Resource inside the cluster.

---

### 5. Workloads

Resources that run containers are referred to as **Workloads**.

Workloads are responsible for actually running application code.

Common workload types include:

- Deployments
- StatefulSets
- Jobs
- CronJobs
- DaemonSets

Each workload type exists for a specific use case, but they all follow the same Resource model.

---

### 6. Resource Config

Users work with Resource APIs by declaring them in files.

These files are called **Resource Config**.

Important properties of Resource Config:

- Declarative
- Describes desired state
- Does not contain execution logic

You do not tell Kubernetes *how* to do something.  
You tell Kubernetes *what* the final state should look like.

---

### 7. Applying Resource Config

Resource Config is applied to the cluster using tools such as kubectl.

This apply operation supports:

- Create
- Update
- Delete

All through a declarative model.

After the Resource is stored in the API server, **controllers take over**.

---

### 8. Unique identity of Resources

Every Resource in Kubernetes is uniquely identified by four fields:

- apiVersion  
  Defines the API group and version

- kind  
  Defines the type of object

- metadata.namespace  
  Defines the namespace of the instance

- metadata.name  
  Defines the name of the instance

Together, these four values uniquely identify a Resource in the cluster.

---

### 9. Structure of a Resource

Every Kubernetes Resource follows the same structural pattern.

It consists of the following components.

---

### 10. TypeMeta

TypeMeta describes **what kind of Resource this is**.

It includes:

- apiVersion
- kind

This tells Kubernetes *which API and controller logic* applies to the object.

---

### 11. ObjectMeta

ObjectMeta contains identifying and descriptive information.

This includes:

- Resource name
- Namespace
- Labels
- Annotations
- Other metadata

This data is used for organization, selection, and automation.

---

### 12. Spec

Spec defines the **desired state** of the Resource.

This is the intent provided by the user.

Examples of Spec data:

- Number of replicas
- Container image
- CPU and memory limits
- Ports and environment variables

Spec answers the question:  
“What do I want the cluster to look like?”

---

### 13. Status

Status represents the **observed state** of the Resource.

This field is:

- Written by the cluster
- Updated by controllers
- Read-only for users

Status answers the question:  
“What is actually happening right now?”

Important rule:

Resource Config written by users **omits the Status field**.

---

### 14. Introduction to Controllers

Controllers are responsible for **actuating Resources**.

They continuously:

- Watch the Kubernetes API
- Observe desired state changes
- Observe system changes
- Take action to reconcile differences

Controllers are the engine that makes Kubernetes work.

---

### 15. What controllers observe

Controllers react to two main types of changes:

- Changes to Resources  
  Create, update, or delete operations

- Changes to the system  
  Pod crashes, node failures, scaling events

Controllers respond automatically without user intervention.

---

### 16. How controllers fulfill intent

Controllers make changes to the cluster to fulfill intent defined by:

- Resource Config provided by users
- Automation systems like Autoscalers

They do not run once.  
They run continuously.

---

### 17. Controller example: Deployment

When a user creates a Deployment:

1. The Deployment Resource is stored in the API
2. The Deployment Controller notices it
3. The controller checks for the expected ReplicaSet
4. If the ReplicaSet does not exist, the controller creates it
5. The ReplicaSet then manages Pods

Each controller focuses on its own responsibility.

---

### 18. Asynchronous actuation

Kubernetes works asynchronously.

This means:

“Kubernetes takes your instructions, stores them, and then in the background, a controller starts working to make them happen. You don’t wait — it just happens eventually.”

There is no blocking execution like a script.

---

### 19. Why asynchronous behavior matters

This model allows Kubernetes to:

- Recover from failures automatically
- Continuously correct drift
- Scale systems reliably
- Handle large distributed environments

Instead of doing something once, Kubernetes keeps checking forever.

---

### 20. Desired state reconciliation

You provide Kubernetes with a **desired state**, similar to a wish list.

Kubernetes does not execute instructions step by step.

Instead:

- Controllers constantly compare desired state with actual state
- Differences are fixed over time
- The system converges toward your intent

This reconciliation loop is the heart of Kubernetes.
