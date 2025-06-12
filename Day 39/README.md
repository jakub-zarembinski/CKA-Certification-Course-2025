# Day 39: Custom Resources (CR) and Custom Resource Definitions (CRD) | CKA Course 2025

## Video reference for Day 39 is the following:


---
## ⭐ Support the Project  
If this **repository** helps you, give it a ⭐ to show your support and help others discover it! 

---

### Prerequisites:

To fully understand this concept, it’s highly recommended to revisit the **Kubernetes API section from Day 35**:

* [Day 35 GitHub Notes](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main/Day%2035)
* [Day 35 YouTube Lecture](https://www.youtube.com/watch?v=DkKc3RCW2BE&ab_channel=CloudWithVarJosh)

Watching the API deep dive alone is sufficient to understand how resources are surfaced through endpoints.

---

## What is a Kubernetes Resource?

Anything you create or manage inside a Kubernetes cluster is considered a **resource**. Throughout this course, you’ve already worked extensively with them — from pods and deployments to services and persistent volume claims.

Each of these resources is an object exposed by the Kubernetes API. These resources are organized under API **groups**, and each group contains multiple **versions** and **kinds** of resources.

To explore the complete list of built-in resources in your cluster, you can use one of the most informative commands in your toolbox:

```bash
kubectl api-resources
```

If you've been following this course from the beginning, you already know how much insight this command provides. It shows not just resource names and short names, but also their associated API group, namespaced scope, and supported verbs.

---

## What is a Custom Resource?

A **custom resource** is any resource that does not exist in Kubernetes by default but is introduced by the user or an external system to extend the Kubernetes API.

Kubernetes exposes resources via its API endpoints. These include:

* `/api` — for the core group (e.g., Pods, Services, ConfigMaps)
* `/apis/<group>` — for all named groups (e.g., `apps/v1` for Deployments)

A custom resource becomes part of the `/apis` path under a new or existing API group. It lets you define your own object kind, schema, and expected behavior.

To fully understand this concept, it’s highly recommended to revisit the **Kubernetes API section from Day 35**:

* [Day 35 GitHub Notes](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main/Day%2035)
* [Day 35 YouTube Lecture](https://www.youtube.com/watch?v=DkKc3RCW2BE&ab_channel=CloudWithVarJosh)

Watching the API deep dive alone is sufficient to understand how resources are surfaced through endpoints.

---

## Key Properties of Custom Resources

* **Created by you or an external system:**
  Custom Resources are user-defined objects created by cluster users or external systems/applications, not built-in Kubernetes components.

* **Exist only under named API groups (`/apis`):**
  CRs live under custom API groups accessed via `/apis/{group}`; they cannot be part of the core API group (`/api`).

* **Extend the Kubernetes API without modifying Kubernetes code:**
  CRDs allow you to add new resource types to Kubernetes dynamically, without changing or rebuilding the Kubernetes API server.

* **Fully compatible with `kubectl`:**
  Once a CRD is created, you can manage your custom resources using standard `kubectl` commands (get, describe, create, delete, etc.), similar to built-in resources.

* **Declarative when used with controllers:**
  Custom Resources typically represent desired state; when paired with a controller, the controller ensures the actual cluster state matches this desired state, following Kubernetes’ declarative model.

---

## What is a CustomResourceDefinition (CRD)?

A **CustomResourceDefinition (CRD)** is how you tell Kubernetes about the existence of your new resource type. It is itself a Kubernetes resource — a manifest that defines:

* The API group and version
* The singular and plural names of the resource
* The resource kind
* Optional OpenAPI validation schema

Once applied, it registers a new resource type in your Kubernetes API server.

You can now define and create objects of this new custom kind — these are your **Custom Resources (CRs)**.

> Kubernetes has an extensible API, which means you can expand its capabilities by introducing new types of resources and controllers beyond the built-in ones like Pods, Deployments, and Services. A **Custom Resource (CR)** is one such extension — a user-defined object added to the cluster. The **CustomResourceDefinition (CRD)** acts as the glue that registers this new resource type with the Kubernetes API, enabling full `kubectl` compatibility.


---

## Do Built-in Resources Have Definitions Like CRDs?

While **Custom Resources** require us to explicitly create a `CustomResourceDefinition` (CRD), **built-in resources** like `Deployment`, `Pod`, `Service`, etc., also have something *similar* — but it is **already defined by Kubernetes itself** in the **core or built-in APIs**.

---

### Equivalent of CRD for Built-in Resources

For built-in resources:

* There **is no separate YAML file** that defines their structure like a CRD.
* Instead, their "definitions" are **built into the Kubernetes API server** and published via the API machinery.

These definitions include:

* Group/Version (like `apps/v1` for Deployment)
* Kind (`Deployment`)
* OpenAPI schemas (what fields are allowed in `spec`, `metadata`, etc.)

---

### Where Is This Resource Definition Stored?

You can inspect the schema and structure of built-in resources in several ways:

#### 1. **Kubernetes OpenAPI Schema**

Kubernetes exposes its OpenAPI definitions (just like CRDs):

```bash
kubectl get --raw /openapi/v2 | jq .
```

This contains the full schema for every resource, including fields, types, descriptions.

#### 2. **kubectl explain**

This is the most user-friendly way to explore built-in resource structure:

```bash
kubectl explain deployment.spec
kubectl explain pod.spec.containers
```

This is essentially your built-in “resource definition” viewer — same in spirit as a CRD.

#### 3. **Kubernetes API Reference Docs**

Online docs like [Kubernetes API Reference](https://kubernetes.io/docs/reference/kubernetes-api/) show the official schema and fields for all built-in kinds.

---

## Understanding Controllers and Custom Controllers in Kubernetes

When we discussed the **Kubernetes architecture** earlier in the course, we talked about the **`kube-controller-manager`** — which acts as an **umbrella for multiple built-in controllers** running in the cluster.

Kubernetes has many default resource types (like Deployments, ReplicaSets, Jobs, and so on), and most of these resources have an associated **controller**. These controllers constantly **observe the current state** of the cluster and compare it against the **desired state** defined by users through YAML manifests. If there's a difference, the controller takes corrective action to bring the current state in line with the desired state.

> For example, the **Deployment controller** ensures the specified number of pod replicas are running at all times.

---



## Custom Controllers and the Control Loop Pattern

When you define a **Custom Resource Definition (CRD)**, it allows you to extend the Kubernetes API with new object types. You can create, retrieve, and delete these custom resources using `kubectl`, just like built-in ones. However, these resources are **passive by default** — Kubernetes will store them, but it won’t take any action on them unless explicitly told to.

To **act on a CRD** (e.g., spin up Pods based on a custom spec), you need to implement a **Custom Controller** — a background process that watches for changes in your custom resource and reconciles the cluster state accordingly.

---

### What Is a Controller's Job?

Every controller in Kubernetes — built-in or custom — follows the **control loop pattern**:

> **"Observe desired state → Compare with current state → Take action to reconcile"**

This logic forms the heartbeat of Kubernetes automation and declarative management.

---

### What Is a Control Loop?

A **control loop** is a continuous cycle where a controller:

1. **Watches** the current state of the cluster (usually via a shared informer and cache)
2. **Compares** it with the desired state (defined in a resource’s `.spec`)
3. **Takes corrective action** to bring the actual state in line with what is desired

This loop is triggered by events (e.g., changes to a resource) and often backed by periodic rechecks to ensure eventual consistency.

---

### Real-World Analogy: Thermostat

Think of a thermostat in your house:

* You set the desired temperature: **22°C**
* The thermostat reads the current temperature: **18°C**
* It turns on the heater to bring it up to 22°C

This is exactly how Kubernetes controllers function — observing, comparing, and correcting.

---

### Example: Deployment Controller

If you create a deployment like this:

```yaml
spec:
  replicas: 3
```

... and one pod crashes, the **Deployment controller** notices that only 2 pods are running (current state), compares it to the desired state (3 replicas), and immediately creates another pod.

---

### Custom Controllers: Bringing Logic to CRDs

If you define a CRD (e.g., `Widget`, `AppConfig`, or `BackupPolicy`), you might want logic that enforces a spec — say, starting a Pod or taking a snapshot.

In that case, a **custom controller**:

* Watches your CR objects (using informers)
* Compares `.spec` with actual system state (often `.status`)
* Applies changes (e.g., creating Jobs, ConfigMaps, Deployments, etc.)

Even at a high level, most custom controllers resemble this loop:

```go
for {
    // 1. Fetch the latest state of the custom resource
    // 2. Compare desired and current states
    // 3. Take action if needed
}
```

---

### Tools for Writing Controllers

Most controllers are written in **Go**, using one of the following frameworks:

* [**Kubebuilder**](https://github.com/kubernetes-sigs/kubebuilder) – scaffolds full controller projects with CRD support
* [**controller-runtime**](https://github.com/kubernetes-sigs/controller-runtime) – lower-level library that Kubebuilder uses internally
* [Operator SDK](https://sdk.operatorframework.io/) – often used in conjunction with OLM (Operator Lifecycle Manager)

---

### CKA Relevance

> While writing controllers isn't required for the CKA exam, understanding the **control loop pattern** helps demystify how Kubernetes automates everything — and gives you the vocabulary to explain what controllers actually do.

---


### Not All Resources Have Controllers

Some resources are just metadata or configuration blobs — and don’t require active management.

Examples:

* `ConfigMap`, `Secret`, `Pod` (standalone), `PersistentVolume` **do not** have controllers.
* `Deployment`, `StatefulSet`, `DaemonSet`, `HPA`, `Job`, `CronJob`, `ReplicaSet` **do**.

---

## Rule of Thumb

> **If a resource defines a desired state and can drift from it, Kubernetes usually assigns a controller to reconcile it.**
> **If a resource is static, metadata-driven, or only evaluated at creation time, no controller is needed.**

---

## Demo: Creating a Realistic Custom Resource — `BackupPolicy`

This example is more aligned with **real production needs**. We'll define a `BackupPolicy` CRD that enables teams to declare backup behavior for their apps — such as schedules, retention, and volume to back up — without needing to write scripting or manage jobs manually.

---

### Why Not Just Use a CronJob?

You *can* use Kubernetes `CronJob`, but it has limitations:

| Feature                         | CronJob                  | BackupPolicy CRD (with Controller)    |
| ------------------------------- | ------------------------ | ------------------------------------- |
| Schedule execution              | ✅ Built-in               | ✅ Handled by custom controller        |
| PVC awareness                   | ❌ Manual volumeMounts    | ✅ Field in CR, resolved by controller |
| Retention policies              | ❌ Not supported          | ✅ Declarative (`retentionDays`)       |
| Backup validation               | ❌ Needs external scripts | ✅ Logic in controller                 |
| Application-specific hooks      | ❌ Needs customization    | ✅ Encapsulated in Go logic            |
| Declarative backup config store | ❌ Procedural job YAMLs   | ✅ CR acts as single config unit       |

So, a `BackupPolicy` CRD abstracts this complexity and lets app teams describe what they want — and a custom controller enforces it.

---

## Step 1: Define the CRD — `backup-policy-crd.yaml`

```yaml
# This defines a new custom resource type in the Kubernetes API
apiVersion: apiextensions.k8s.io/v1  # The API version for CRDs; must be this for modern Kubernetes
kind: CustomResourceDefinition       # We're defining a new custom resource type

metadata:
  name: backuppolicies.ops.cloudwithvarjosh  # Full name must be <plural>.<group>; it's how Kubernetes uniquely identifies the CRD

spec:
  group: ops.cloudwithvarjosh         # API group under which this CRD will be exposed (used in apiVersion when creating instances)

  names:
    plural: backuppolicies            # Plural name used in kubectl (e.g., `kubectl get backuppolicies`)
    singular: backuppolicy            # Singular name (used for clarity, e.g., in messages)
    kind: BackupPolicy                # PascalCase kind — used as the `kind` field in resource manifests
    shortNames:
      - bp                            # Optional: short alias for CLI use (e.g., `kubectl get bp`)

  scope: Namespaced                   # Resource will exist inside namespaces; each namespace can have its own set of policies

  versions:
    - name: v1                        # Version of the custom resource
      served: true                    # This version will be served by the Kubernetes API
      storage: true                   # This version will be used to store objects in etcd

      schema:
        openAPIV3Schema:              # Schema used to validate objects created with this CRD
          type: object                # Root of the object must be an object

          properties:
            spec:                     # The spec field holds user-defined input (desired state)
              type: object
              properties:

                schedule:
                  type: string
                  description: >      # Cron-formatted schedule for the backup job
                    Cron format (e.g., "0 1 * * *") — defines when the backup should run.
                    This is similar to how Kubernetes CronJobs are scheduled.

                retentionDays:
                  type: integer
                  description: >      # Number of days to retain old backups before deletion
                    Specifies how long backup data should be preserved.
                    Useful for compliance, cost optimization, or restore planning.

                targetPVC:
                  type: string
                  description: >      # Name of the PersistentVolumeClaim to be backed up
                    This field identifies which PVC (storage volume) should be backed up.
                    Useful for targeting database or application data volumes.

```

---

## Step 2: Apply the CRD

```bash
kubectl apply -f backup-policy-crd.yaml
```

Verify creation:

```bash
kubectl get crd backuppolicies.ops.cloudwithvarjosh
```

Check API resources:

```bash
kubectl api-resources | grep backup
```

Expected output:

```
backuppolicies   bp   ops.cloudwithvarjosh/v1   true   BackupPolicy
```

---

## Step 3: Create a Custom Resource — `mysql-backup.yaml`

Now, create a CR for your specific workload. In this case, it defines a backup policy for a MySQL PVC.

```yaml
apiVersion: ops.cloudwithvarjosh/v1   # Must match the group and version defined in the CRD
kind: BackupPolicy                    # Must match the 'kind' field in the CRD

metadata:
  name: mysql-backup                  # Name of this BackupPolicy object
  namespace: default                  # Since scope is Namespaced, this object will exist in the 'default' namespace

spec:
  schedule: "0 1 * * *"               # Backup should run daily at 1:00 AM (Cron format: min hour dom mon dow)
  retentionDays: 7                    # Retain backups for 7 days; older ones can be deleted
  targetPVC: mysql-data-pvc           # The name of the PersistentVolumeClaim (PVC) to back up
```

**What this represents:**

You’re declaring:

> “At 1:00 AM every day, take a backup of the PVC named `mysql-data-pvc`, and keep those backups for 7 days.”

But remember — this declaration **won’t cause any backups to happen** *unless* there’s a **controller** that watches `BackupPolicy` resources and initiates actions accordingly.

Apply it:

```bash
kubectl apply -f mysql-backup.yaml
```

---

## Step 4: Interact With the BackupPolicy Resource

```bash
kubectl get backuppolicies
```

Output:

```
NAME            AGE
mysql-backup    10s
```

Get details:

```bash
kubectl describe backuppolicy mysql-backup
```

Fetch YAML:

```bash
kubectl get backuppolicy mysql-backup -o yaml
```

---

## Step 5: Clean Up

```bash
kubectl delete -f mysql-backup.yaml
kubectl delete -f backup-policy-crd.yaml
```

---

## Recap: What’s Happening and Why It’s Useful

This CRD doesn’t **do** anything by itself — just like a `Widget`, it only stores config.

But in production:

* A **custom controller** would **watch `BackupPolicy` CRs**
* It would dynamically **create CronJobs or one-time Jobs**
* It could:

  * Mount the correct PVC
  * Use annotations or labels to tag backups
  * Upload them to S3
  * Enforce retention and cleanup
  * Alert on failures

By defining a CRD + CR:

✔ Teams write **only the intent** (schedule, retention, PVC)
✔ Platform engineers encode all the logic in a controller
✔ Makes backups **repeatable, policy-driven, and declarative**

---

## 🧠 Real Production Scenario

This kind of approach is used in:

* **Internal platform teams** creating Kubernetes abstractions
* **Backup-as-a-Service operators** like [Velero](https://velero.io/)
* **GitOps pipelines** to automate disaster recovery workflows
* **Policy engines** that verify every workload has a backup plan

--- 