# Day 42: Kustomize | CKA Course 2025

## Video reference for Day 42 is the following:


---
## ⭐ Support the Project  
If this **repository** helps you, give it a ⭐ to show your support and help others discover it! 

---

## Why Kustomize?

In real-world Kubernetes deployments, we often deal with multiple environments like **development**, **staging**, and **production**. While the application itself remains the same, environment-specific values such as image versions, replica counts, labels, and annotations tend to differ.

Let’s take a familiar example — the `nginx` deployment. In many organizations:

* **Dev** might use a newer version like `nginx:1.22` or even the `latest` build for testing purposes.
* **Staging** and **Prod** generally run a more stable and vetted version like `nginx:1.20`, often identical to ensure consistency.

This parallels how infrastructure environments are managed — you might test newer OS or runtime versions in dev before pushing them to prod.

---

### The Problem

Imagine you have an application named **app1** which runs in **three environments**: `dev`, `stage`, and `prod`. Each environment is made up of three logical tiers: **frontend**, **backend**, and **data**. Each tier typically involves several Kubernetes objects — such as:

* Deployments
* StatefulSets
* DaemonSets
* Services
* ConfigMaps
* PersistentVolumeClaims
* ServiceAccounts

That’s approximately **5–6 objects per tier**, and with 3 tiers per environment, we’re easily talking about **15 resources per environment**. Multiply this across 3 environments and you're now managing **\~45 Kubernetes objects** — and that’s just for a single application. A real-world production environment may have more objects, and **cloud-native applications often have multiple microservices per tier**, pushing this number much higher.

---

### What If You Want to:

* Add labels for observability or compliance
* Inject new annotations (e.g., for cost tracking or security posture)
* Change image versions for a patch release or rollback
* Modify naming conventions
* Adjust replica counts or resource limits

Making changes like these **manually across dozens of YAMLs** is time-consuming and error-prone. Even if your team gets it done, **manual edits invite human error** — and that's the last thing you want in production.

---

### What We Want

We want to avoid maintaining environment-specific copies of the same YAML with minor differences. Instead, we need:

* **A shared base** that defines common application structure
* **Environment-specific overlays** that customize only what's necessary

This promotes maintainability, consistency, and safety — and it's what **Kustomize** was designed to do.

---

### Real-World Perspective

> In production-grade Kubernetes environments, your cluster will typically host **multiple applications**, each composed of **multiple tiers** or **microservices**. Every tier — whether it's frontend, backend, database, caching, or messaging — will have its own Kubernetes manifests: **Deployments**, **Services**, **ConfigMaps**, **Secrets**, **PersistentVolumeClaims**, **ServiceAccounts**, and more.
>
> Now imagine you need to make a global change — like **adding a new label** for monitoring or compliance to all workloads, or **updating image versions** across services due to a security vulnerability. Managing this across dozens of YAML files manually is tedious and error-prone.
>
> You need a mechanism that makes such multi-file, multi-resource updates **structured**, **automated**, and **safe** — and this is where **Kustomize** shines.

---


## Enter Kustomize

The **Kubernetes community recognized** the challenges faced by platform engineers and DevOps teams when managing multiple environments, repeated YAMLs, and drift between manifests. Rather than relying on external templating tools, they introduced a **native, purpose-built solution** for Kubernetes manifest customization — and that’s where **Kustomize** enters.

**Kustomize** addresses this challenge by letting you define:

1. **Base Configuration** — shared across environments (what stays the same)
2. **Overlays** — environment-specific patches (what needs to change)

This ensures your YAMLs remain DRY (Don’t Repeat Yourself), manageable, and scalable — even as your applications grow in complexity.

> **Kustomize is not a templating tool.** Unlike Helm, there are no variables, functions, or templating syntax. You continue working with **plain YAML** as before. What changes is the introduction of **overlays** — declarative patches that adjust specific fields for environments like dev, staging, and prod. These overlays allow you to reuse and adapt a common base configuration without duplicating entire files or introducing templating complexity.

---



## What is Kustomize?

**Kustomize** is a declarative configuration management tool for Kubernetes that allows you to customize raw, template-free YAML files for multiple environments. Instead of using variables or a separate templating language (like in Helm), Kustomize applies **overlays** and **patches** to your base configuration — all in pure YAML.

Kustomize works natively with `kubectl` and supports:

* Applying **overrides** like replica count, image tags, labels, etc.
* Managing **variants of Kubernetes objects** without duplicating entire YAML files
* Keeping configurations **clean, modular, and DRY**

With Kustomize, your Kubernetes manifests are:

* Fully declarative
* Structured as **base + overlays**
* Compatible with existing tooling (`kubectl`, GitOps, CI/CD pipelines)

> Kustomize is ideal for teams that want to maintain multiple environment-specific configurations in a scalable, native way — without introducing a templating engine.

---

## Example: NGINX Deployment Across Dev, Staging, and Prod

Let’s say we want to deploy nginx across three environments with the following variations:

| Environment | Image Tag  | Replicas | Label        |
| ----------- | ---------- | -------- | ------------ |
| Dev         | nginx:1.22 | 2        | env: dev     |
| Staging     | nginx:1.20 | 3        | env: staging |
| Production  | nginx:1.20 | 5        | env: prod    |

### What We Do with Kustomize

* The **base** folder contains the generic YAML manifests: Deployment, Service, etc.
* Each **overlay** (dev, staging, prod) contains a small patch to modify specific fields like `replicas`, `image`, or `metadata.labels`.

---

## Recommended Folder Structure

```
kustomize-demo/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   └── patch-deployment.yaml
    ├── staging/
    │   ├── kustomization.yaml
    │   └── patch-deployment.yaml
    └── prod/
        ├── kustomization.yaml
        └── patch-deployment.yaml
```

* `base/`: Includes core deployment logic (same for all envs)
* `overlays/dev/`: Patch for image tag `1.22`, replicas `2`, and label `env=dev`
* `overlays/staging/`: Patch for image tag `1.20`, replicas `3`, and label `env=staging`
* `overlays/prod/`: Patch for image tag `1.20`, replicas `5`, and label `env=prod`

---

## How Kustomize Works (Flow Diagram)

```
        [ Base Manifests ]
                │
                ▼
        +------------------+
        |   Kustomization  |
        +------------------+
         │     │      │
         ▼     ▼      ▼
   [Dev Patch] [Staging Patch] [Prod Patch]
         │     │      │
         ▼     ▼      ▼
[Final Dev YAML] [Final Staging YAML] [Final Prod YAML]
```

Kustomize **combines the base and overlays** using rules like strategic merge or JSON patches. This results in finalized, environment-specific manifests — without duplicating entire YAMLs.

---

## Helm vs Kustomize

### Kustomize vs Helm

* **Templating vs Patching**: Helm uses Go templating for dynamic values; Kustomize uses declarative patches without templates.
* **Package Manager vs Overlay Tool**: Helm acts like a package manager with versioned charts; Kustomize focuses on customizing existing YAMLs.
* **Release Tracking**: Helm tracks deployments as releases; Kustomize does not manage or track releases.
* **Native Support**: Kustomize is built into `kubectl` (`kubectl apply -k`); Helm is a separate CLI tool.
* **Complexity vs Simplicity**: Helm is more feature-rich (hooks, rollbacks, dependencies); Kustomize is simpler and ideal for environment-specific config management.

---

### Why Introduce Kustomize First?

Kustomize is native to Kubernetes (via `kubectl kustomize`) and relies entirely on YAML — no new syntax or language to learn. It encourages clean separation between **base configuration** and **environment-specific overlays** without introducing logic or templating. It’s a great entry point before tackling the added complexity and features of Helm.

---

## Feature Comparison: Helm vs Kustomize

| Feature                      | Feature Description                                                            | Kustomize                  | Helm                           |
| ---------------------------- | ------------------------------------------------------------------------------ | -------------------------- | ------------------------------ |
| Native to Kubernetes         | Whether the tool is integrated into `kubectl` and part of Kubernetes ecosystem | Yes (built into `kubectl`) | No (requires separate binary)  |
| Template Language            | Supports templating syntax like Go templates                                   | No                         | Yes (Go templating syntax)     |
| Values/Variables Support     | Ability to pass and inject variable values                                     | No                         | Yes (`values.yaml`)            |
| Overlay Support              | Native mechanism for layering and environment-specific customization           | Yes (base and overlays)    | Indirect (via separate values) |
| Complexity                   | Learning curve and operational overhead                                        | Low                        | Medium to High                 |
| Package Management           | Ability to version, install, and upgrade packaged resources                    | No                         | Yes (Helm Charts)              |
| Logic/Conditionals/Loops     | Ability to use control structures inside manifests                             | No                         | Yes                            |
| Extensibility (hooks, tests) | Supports lifecycle hooks, testing, and advanced release workflows              | No                         | Yes                            |
| Reusability                  | Degree to which definitions can be modular and reused                          | Moderate                   | High (charts and subcharts)    |
| Use Case Fit                 | Best suited for what type of applications                                      | Simple to medium apps      | Medium to complex apps         |


---

**Summary:**
Use **Kustomize** when you want a straightforward way to manage environment-specific differences without introducing new syntax. Use **Helm** when your application requires richer templating, packaging, and reusability — especially if you're distributing or consuming applications via Helm charts.

Let me know if you’d like to follow this with a hands-on demo comparing the two.

---

## Installing Kustomize

Kustomize is bundled with `kubectl` (version ≥1.14), so you likely already have it. However, to ensure you're using the latest standalone version, you can upgrade it manually.

**Install the latest version:**

Visit the [official installation page](https://kubectl.docs.kubernetes.io/installation/kustomize/) and navigate to the **"binaries"** section. Or use this one-liner:

```bash
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
```

This script installs the latest stable version into your working directory.

---


## Understanding `kustomization.yaml`

In Kustomize, the `kustomization.yaml` file is the **core driver** of how your final Kubernetes configuration is assembled. It dictates **what gets included** in your manifests and **how it gets transformed**, acting as the blueprint that merges your common base with environment-specific overlays.

Importantly, while every directory in a Kustomize setup will contain a `kustomization.yaml` file, the **contents and intent** of this file differ depending on whether it resides in a **base** or an **overlay**.

 **Note:**
The file **must** be named `kustomization.yaml` (all lowercase).
Kustomize will **only recognize this exact filename**. Files like `Kustomization.yaml`, `kustomization.yml`, or `customization.yaml` will **not work**, and Kustomize will throw an error such as:
```
Error: no kustomization file found in the current directory
```
This strict naming convention is by design — it ensures predictable behavior across tools and CI/CD systems that rely on Kustomize.

---

### In the `base/` Directory

The `base/` directory represents the **common configuration shared across all environments** — essentially, what stays the same. The `kustomization.yaml` here serves as a reference list of the base Kubernetes objects.

* It typically uses the `resources:` field to list raw Kubernetes YAML files like Deployments, Services, ConfigMaps, etc.
* It does **not include any environment-specific customizations** — no replica counts, image versions, or label overrides.
* This keeps your base clean, reusable, and environment-agnostic.

Example:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deploy.yaml
  - svc.yaml
```

This setup ensures that every overlay consuming this base starts from the same consistent foundation.

---

### In the `overlays/` Directory

Each overlay represents a **specific environment** — such as `dev`, `stage`, or `prod`. The `kustomization.yaml` in these directories is used to **override or extend** the base configuration.

* Instead of pointing to raw YAMLs directly, overlays use `resources: ../../base` to reference the base configuration.
* You then apply environment-specific customizations — such as name prefixes/suffixes, namespace assignments, replica overrides, or image tag updates.
* Kustomize also supports different patching strategies here (`patches`, `patchesJson6902`, `patchesStrategicMerge`) to modify parts of the base manifests without copying them.

Example:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

nameSuffix: -dev

```

This makes overlays highly expressive — letting you inject environment-specific concerns **without duplicating** the base YAMLs.

---

### Why This Separation Matters

This separation between base and overlays ensures a clean, maintainable structure across environments:

* **Reusability**: Your base can be used across dev, staging, QA, and production without modification.
* **Clarity**: Overlays only express what’s different — helping teams reason about changes at a glance.
* **Maintainability**: Reduces duplication and drift between environments, especially when updates are needed across the board (e.g., changing labels, security fixes, etc).

By organizing configuration this way, Kustomize gives you a powerful, native, and declarative way to scale your Kubernetes manifests safely and efficiently.


---

### Is `base/` and `overlays/` Mandatory?

> **No, the directory names `base/` and `overlays/` are purely conventional.**

They are **not enforced** by Kustomize. What Kustomize actually cares about is the presence of a `kustomization.yaml` file in each directory, and how directories are **referenced via relative paths** inside that file.

#### So why do we use `base/` and `overlays/`?

They’re simply **well-adopted conventions** in the community that help structure your project clearly:

* `base/` typically contains the common resources shared across environments.
* `overlays/dev`, `overlays/stage`, `overlays/prod`, etc., contain environment-specific customizations.

You could just as easily use directories like:

```
├── shared-resources/
│   └── kustomization.yaml
└── env-configs/
    ├── dev/
    ├── prod/
    └── test/
```

As long as each directory includes a properly structured `kustomization.yaml` and references its relative resources correctly, Kustomize will work just fine.



---

## Demo 1: Kustomize Basics with Name Suffix Transformer

In this hands-on demonstration, we'll introduce Kustomize by using one of its simplest but most effective features — the **nameSuffix transformer**. This will help you understand the foundational concepts behind base configurations and environment-specific overlays.

### Step 1: Directory Setup

Since this is **Day 42** of the course, we'll organize all relevant files under a directory named `Day42`. Inside that, we’ll create the following structure:

```
Day42/
├── base/
│   ├── deploy.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/
    │   └── kustomization.yaml
    ├── stage/
    │   └── kustomization.yaml
    └── prod/
        └── kustomization.yaml
```

This layout separates common resources (base) from environment-specific modifications (overlays), which is the core design pattern of Kustomize.

---

### Step 2: Create the Base Configuration

In the `base/` directory, create a deployment manifest for a simple `nginx` app:

**`base/deploy.yaml`**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
```

Now, create a `kustomization.yaml` file in the same `base/` directory. This file tells Kustomize which resources in the directory are part of the base:

**`base/kustomization.yaml`**

```yaml
apiVersion: kustomize.config.k8s.io/v1
kind: Kustomization

resources:
  - deploy.yaml
```

Note: You can list multiple YAMLs here if needed. Kustomize will apply patches only to the files you reference in the overlays — all others will be used as-is.

---

### Step 3: Create Overlays for Environments

Under the `overlays/` directory, create three subdirectories: `dev/`, `stage/`, and `prod/`. Each of these will have its own `kustomization.yaml`.

Each overlay does two key things:

* **References the shared base** directory, which holds the common manifests.
* **Applies a `nameSuffix`** to distinguish environment-specific workloads, avoiding naming collisions when resources are deployed in the same cluster.

#### Example: `overlays/dev/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

nameSuffix: -dev
```

> **What does `../../base` mean?**
> This is a **relative path** from the location of the overlay. Since the dev overlay lives at `overlays/dev/`, `../../base` tells Kustomize to go up two directory levels and then down into the `base/` directory. This is how the overlay "points to" the shared base configuration to inherit its resources.

Repeat this for `stage` and `prod` as well, changing the suffix appropriately:

* `nameSuffix: -stage`
* `nameSuffix: -prod`

The `nameSuffix` transformer automatically appends the specified suffix to the names of all Kubernetes resources defined in the base (e.g., `nginx-deploy` becomes `nginx-deploy-dev`). This ensures clear differentiation of workloads across environments, especially when deploying to the same cluster or namespace.

---

### Step 4: View Generated Manifests

You can now view the final manifests generated by Kustomize without applying them:

```bash
# View base manifest without overlay
kubectl kustomize base

# View overlay for dev environment
kubectl kustomize overlays/dev

# Similarly, view for stage and prod
kubectl kustomize overlays/stage
kubectl kustomize overlays/prod
```

When you run the dev overlay, you’ll see the deployment name transformed:

```yaml
metadata:
  name: nginx-deploy-dev
```

But the rest of the manifest (like replicas, image, labels) remains the same — since we’ve not patched those yet.

---

### Step 5: Namespace Preparation and Applying Overlays

For better isolation and best practice, we’ll create separate namespaces:

```bash
kubectl create namespace dev
kubectl create namespace stage
kubectl create namespace prod
```

Now, apply each overlay using the `-k` flag (short for "kustomize"):

```bash
kubectl apply -k overlays/dev -n dev
kubectl apply -k overlays/stage -n stage
kubectl apply -k overlays/prod -n prod
```

This tells `kubectl` to build the customized manifest from the overlay directory and apply it to the specified namespace.

> Note: `-k` treats the provided path as a Kustomize directory. Do not use `-f` here, since that expects a file or raw YAML content.

---

### Summary of What We Learned

* **Base** holds reusable, environment-agnostic YAMLs.
* **Overlays** introduce environment-specific customizations.
* `nameSuffix` helps differentiate workloads across dev, stage, and prod.
* Kustomize reads `kustomization.yaml` files recursively to build final manifests.

---

### What’s Next?

Now that we’ve seen how the basic structure works, we’ll build on this foundation in **Demo 2** by introducing additional transformers — such as:

* Custom labels per environment
* Patching image versions
* Changing replica counts

These demonstrate how Kustomize allows flexible, maintainable Kubernetes deployments across multiple environments without duplication.


---

## Demo 2: Applying Additional Transformers in Kustomize

In this second demo, we’ll take the foundational structure from Demo 1 and apply **additional Kustomize transformers** to our configuration. This helps us better align our manifests with production-like requirements by customizing resource names, namespaces, annotations, labels, and container image versions — all without modifying the original YAMLs in the base.

---

### Objective

We'll enhance the `kustomization.yaml` file inside `overlays/dev/` to include the following transformers:

* Prefix and suffix for naming consistency
* Custom labels and annotations
* Image tag override
* Namespace assignment

---

### Step 1: Update `kustomization.yaml` in overlays/dev

Here’s the modified content for `Day42/overlays/dev/kustomization.yaml`:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

namePrefix: app1-
nameSuffix: -dev

namespace: dev

labels:
  - pairs:
      env: dev
    includeSelectors: true

commonAnnotations:
  branch: dev
  support: 800-800-700

images:
  - name: nginx
    newName: nginx
    newTag: "latest"

replicas:
  - name: nginx-deploy
    count: 2
```

---

### Step 2: Field-by-Field Explanation

* `resources: - ../../base`
  This tells Kustomize to load the base configuration from the relative path. The `../../base` path means: go two levels up and then into the `base/` directory. This ensures that any change to base manifests automatically reflects in the overlays.

* `namePrefix: app1-`
  Adds a prefix (`app1-`) to the names of all resources (e.g., `nginx-deploy` becomes `app1-nginx-deploy`).

* `nameSuffix: -dev`
  Adds a suffix (`-dev`) to all resource names to clearly indicate this is for the dev environment.

* `namespace: dev`
  Assigns all resources to the `dev` namespace. You do not need to manually define the namespace inside each manifest or specify `-n dev` while applying.

* `labels:`
  Adds custom labels to both metadata and selectors.

  ```yaml
  labels:
    - pairs:
        env: dev
      includeSelectors: true
  ```

  This adds `env=dev` to metadata **and** to selectors. Including selectors is important to ensure the label change is consistent between the Deployment selector and Pod template labels.

* `commonAnnotations:`
  Adds annotations to all resources (e.g., for tracking Git branch, support contact, etc.).

  ```yaml
  commonAnnotations:
    branch: dev
    support: 800-800-700
  ```

* `images:`
  Overrides the container image for a specific container. In this case, we're pinning `nginx` to the `latest` tag for development testing.

  ```yaml
  images:
    - name: nginx
      newName: nginx
      newTag: "latest"
  ```


* `replicas:`
  Allows you to declaratively set the number of pod replicas for a specific Deployment, using the built-in **`ReplicaCountTransformer`**.

  ```yaml
  replicas:
    - name: nginx-deploy
      count: 2
  ```

  This tells Kustomize to locate the resource named `nginx-deploy` (from the base) and override its `spec.replicas` value to `2`. This is especially useful when you want different environments to scale differently (e.g., dev with 2 replicas, prod with 5), and prefer not to maintain separate patch files for each.

  This feature is native to Kustomize and is processed like any other transformer — clean, declarative, and integrated directly in `kustomization.yaml`.

---

### Step 3: Replicate for Stage and Prod

Make similar changes in the `kustomization.yaml` files inside `overlays/stage/` and `overlays/prod/`. Adjust the `env`, `namespace`, image tag, and name suffix appropriately:

* `env: stage`, `namespace: stage`, `nameSuffix: -stage`, `newTag: "1.20"`
* `env: prod`, `namespace: prod`, `nameSuffix: -prod`, `newTag: "1.20"`

---

### Step 4: Verify the Output

You can verify the rendered manifests using:

```bash
kubectl kustomize overlays/dev
```

Sample output (truncated for readability):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    branch: dev
    support: 800-800
  labels:
    app: nginx
    env: dev
  name: app1-nginx-deploy-dev
  namespace: dev
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
      env: dev
  template:
    metadata:
      annotations:
        branch: dev
        support: 800-800
      labels:
        app: nginx
        env: dev
    spec:
      containers:
      - image: nginx:latest
        name: nginx
        ports:
        - containerPort: 80
```

---

### Step 5: Apply the Kustomized Manifest

Since we've already created the `dev` namespace, we can apply the manifest directly using:

```bash
kubectl apply -k overlays/dev
```

You should see:

```
deployment.apps/app1-nginx-deploy-dev created
```

> No need to specify the namespace on the CLI, as it’s already defined in the Kustomize overlay.

---

### Notes & Best Practices

* The use of `labels` (instead of `commonLabels`) is recommended in recent Kustomize versions, as `commonLabels` is deprecated.
* These transformers allow declarative and centralized control of configuration across environments.
* For deeply nested values (e.g., replicas, resource limits), use **patches**, not transformers. THis is coming up next.

---

### Documentation

* [Kustomize Built-in Transformers](https://kubectl.docs.kubernetes.io/references/kustomize/builtins/)
* [Kubernetes Kustomization Concepts](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/)

---

## Transformers and Patches in Kustomize

In Kustomize, both **transformers** and **patches** play vital roles in customizing Kubernetes manifests — but they serve **different purposes** and should be used accordingly.

---

### ✅ What Are Transformers?

**Transformers** are built-in features that apply **broad, reusable modifications** across multiple Kubernetes resources. These are declared directly in `kustomization.yaml`, and are ideal for high-level changes that apply to all or many objects in a configuration.

Transformers make your configuration DRY, declarative, and easy to scale across environments.

#### Common Transformers and Their Purpose

| Transformer                 | Description                                                              |
| --------------------------- | ------------------------------------------------------------------------ |
| `namePrefix` / `nameSuffix` | Adds a prefix/suffix to all resource names                               |
| `namespace`                 | Assigns resources to a specific namespace                                |
| `labels`                    | Adds labels to metadata and optionally to selectors                      |
| `commonAnnotations`         | Adds annotations across all resources                                    |
| `images`                    | Changes image name or tag inside container specs                         |
| `replicas`                  | Sets the number of replicas per deployment (via ReplicaCountTransformer) |

These transformers are applied declaratively and **do not require separate patch files**.

> You can find the full list of supported transformers in the [official Kustomize documentation](https://kubectl.docs.kubernetes.io/references/kustomize/builtins/).

---

### What Are Patches?

**Patches** are used to make **precise, field-level modifications** to a Kubernetes object — especially for **deeply nested fields** or configurations not supported by transformers.

Patches are helpful when:

* The field you want to change is inside nested structures (e.g., container specs)
* The modification is highly specific (e.g., change a single port, env var, or resource limit)
* You want fine-grained control over a specific object

---

## Understanding Patches in Kustomize

Kustomize supports two main ways to modify deeply nested fields or fine-tune resource configurations:

### Why Use Patches?

While **transformers** offer broad, reusable modifications (like adding labels, changing image tags, setting replica counts), they **cannot target arbitrary fields deep inside complex Kubernetes manifests**.

This includes fields like:

* `containerPort`
* `env` variables
* `resources.requests` and `resources.limits`
* `volumeMounts`, `nodeSelector`, etc.

To handle such cases, Kustomize offers **patches**, which let you surgically override specific portions of a manifest.

---

## Categories of Patches in Kustomize

| Patch Type               | Format     | Merge Strategy      | When to Use                                                   |
| ------------------------ | ---------- | ------------------- | ------------------------------------------------------------- |
| `patchesStrategicMerge`  | YAML       | Strategic Merge     | Best for typical workloads like `Deployment`, `Service`, etc. |
| `patchesJson6902`        | JSON patch | RFC 6902 operations | When strategic merge fails or you need precise edits          |
| `patches` (generic form) | YAML/JSON  | With `target` field | For complex use cases and fine-grained matching               |

Let’s break them down:

---

## Patching in Kustomize: From Strategic Merge to Inline Flexibility

In Kustomize, **patches** allow you to declaratively modify specific parts of Kubernetes manifests without duplicating full YAML files. There are three major patching approaches supported:

1. `patches` (generic form) – modern and inline
2. `patchesJson6902` – file-based with JSON Patch
3. `patchesStrategicMerge` – older, now deprecated

From **Demo 3 onward**, we build on the same consistent layout:

```
├── base
│   ├── deploy.yaml
│   ├── kustomization.yaml
│   └── svc.yaml
└── overlays
    └── dev
        └── kustomization.yaml
```

* We’ll demonstrate patching for the **dev** environment, but the same structure and logic can be applied for `stage` and `prod`.
* In real-world implementations, the folder structure is often more complex, reflecting individual microservices (e.g., `cart/`, `user/`, `payment/`), with each microservice directory containing its own `dev`, `stage`, and `prod` overlays.
* What you see here is a simplified structure for learning. Production usage varies based on team practices and application architecture.

---

## 1. `patches` (Generic Form) — Preferred

This is the most flexible and recommended patching method in modern Kustomize. It allows you to define both the target resource and the patch operations inline in `kustomization.yaml`.

### Why Use This Method

* Inline patching reduces file sprawl
* Works well with GitOps and automation pipelines
* Uses RFC 6902 JSON Patch, which is expressive and concise

### JSON Patch Operations (RFC 6902)

| Operation | Description                                             |
| --------- | ------------------------------------------------------- |
| `add`     | Adds a new field or array entry                         |
| `replace` | Replaces the value at the path (field must exist)       |
| `remove`  | Deletes a field or list element                         |
| `copy`    | Copies a value from one path to another                 |
| `move`    | Moves a value between paths                             |
| `test`    | Validates a field before applying changes (rarely used) |

In practice, `add`, `replace`, and `remove` cover almost all use cases.

---

## Demo 3: `patches` (Generic Form)

In this demo, we will use the preferred `patches` method to perform inline JSON patch operations directly inside `kustomization.yaml`.

We'll build on top of the following base structure:

```text
├── base
│   ├── deploy.yaml
│   ├── svc.yaml
│   └── kustomization.yaml
└── overlays
    └── dev
        └── kustomization.yaml
```

---

### svc.yaml in Base

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

This is a basic NodePort service without a fixed `nodePort` value. If deployed as-is, Kubernetes will allocate a random port from the 30000–32767 range.

---

### overlays/dev/kustomization.yaml

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# Reference to shared base
resources:
  - ../../base

# Optional: Apply name prefix/suffix
nameSuffix: -dev
namePrefix: app1-

# Add common labels and annotations
labels:
  - pairs:
      env: dev
    includeSelectors: true

commonAnnotations:
  branch: dev
  support: 800-800

# Override container image
images:
  - name: nginx
    newName: nginx
    newTag: "latest"

# Target namespace
namespace: dev

# Scale to 2 replicas
replicas:
  - name: nginx-deploy
    count: 2

# Inline patches using RFC 6902 (JSON Patch)
patches:
  # Patch for Deployment
  - target:
      group: apps
      version: v1
      kind: Deployment
      name: nginx-deploy
    patch: |-
      - op: add
        path: /spec/template/spec/containers/0/env
        value:
          - name: LOG_LEVEL
            value: debug
      - op: replace
        path: /spec/template/spec/containers/0/resources
        value:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "256Mi"
      - op: remove
        path: /metadata/labels/unused

  # Patch for Service
  - target:
      version: v1
      kind: Service
      name: nginx-svc
    patch: |-
      - op: replace
        path: /spec/ports/0/nodePort
        value: 31000
```

---

### What We're Doing

This demo illustrates several real-world patching needs:

* **Environment Variable Addition**: We inject `LOG_LEVEL=debug` into the first container. This is common in development and debugging workflows.
* **Resource Limits Setup**: The `resources` block defines CPU/memory limits and requests. Although we're using `replace`, this field doesn’t exist in the base, so ideally we should use `add`.
* **Metadata Cleanup**: We remove a `metadata.labels.unused` key that might exist in the base.
* **Hardcoding NodePort**: Instead of letting Kubernetes assign a random port, we explicitly set the service’s `nodePort` to `31000`. This is useful when external dependencies (e.g., firewall rules or reverse proxies) rely on stable port numbers.

---

### Note on Replace vs Add

The patch uses `replace` for the `resources` block in the container spec. However, this block **does not exist** in the base configuration. While `replace` still works in this case (because Kustomize permits replacing non-existent paths in some cases), this is **not guaranteed across all resource types**. It is **best practice to use `add`** when you're introducing a new field to ensure safe behavior.

Always double-check whether the path you're targeting exists in the base. If it does not, default to `add` instead of `replace` to avoid unexpected runtime errors or patch application failures.

---

## 2. `patchesJson6902` — File-Based Patching

The `patchesJson6902` method allows us to define JSON Patch operations in **separate files**, which is useful when:

* The patch logic is large or complex.
* You want to reuse or share the patch across overlays.
* You need to track it independently in version control.

Unlike the generic `patches:` form, where the patch is defined inline, `patchesJson6902` separates the concern into an external file and references it from `kustomization.yaml`.

---

## Demo 4: `patchesJson6902`

The `patchesJson6902` method allows you to define [RFC 6902](https://datatracker.ietf.org/doc/html/rfc6902) compliant JSON patch operations in a **separate YAML or JSON file**. This is particularly useful for complex, structured patches that should be version-controlled and reused across overlays.

---

### Project Structure

```text
├── base
│   ├── deploy.yaml
│   ├── svc.yaml
│   └── kustomization.yaml
└── overlays
    └── dev
        ├── patch-deploy.yaml
        └── kustomization.yaml
```

---

### overlays/dev/kustomization.yaml

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# Reference to shared base
resources:
  - ../../base

# Optional: Apply name prefix/suffix
nameSuffix: -dev
namePrefix: app1-

# Apply environment-based labels
labels:
  - pairs:
      env: dev
    includeSelectors: true

# Set common annotations for tracking
commonAnnotations:
  branch: dev
  support: 800-800

# Override container image globally
images:
  - name: nginx
    newName: nginx
    newTag: "latest"

# Define target namespace
namespace: dev

# Scale deployment to two replicas
replicas:
  - name: nginx-deploy
    count: 2

# Apply JSON 6902 patch
patchesJson6902:
  - target:
      group: apps
      version: v1
      kind: Deployment
      name: nginx-deploy
    path: patch-deploy.yaml
```

---

### overlays/dev/patch-deploy.yaml

```yaml
- op: add
  path: /spec/template/spec/containers/0/env
  value:
    - name: LOG_LEVEL
      value: debug
- op: replace
  path: /spec/template/spec/containers/0/image
  value: nginx:1.22
```

---

### What We're Doing

This patch performs two JSON operations on the `nginx-deploy` Deployment:

1. **Adds** a new environment variable `LOG_LEVEL=debug` to the first container. This is typically used to toggle debug mode in development.
2. **Replaces** the image from `nginx:latest` (defined in the base overlay via `images`) to `nginx:1.22`.

Although the base overlay already upgrades the NGINX image to `latest`, we override it here at the patch level with `1.22`. This shows how image overrides and patches can interact, with the patch taking precedence.

> **Note on Image Tag Selection**: It's a common misconception that dev environments always run the `latest` image. While that's often true during rapid development, teams may intentionally use older versions in dev to:
>
> * Reproduce bugs from production
> * Perform regression testing
> * Compare behavior across releases
> * Validate backward compatibility

This example shows the dev team forcing `nginx:1.22` instead of `latest` to test a specific scenario. Your patching strategy should be based on intent, not assumptions.

---

### Why Use `patchesJson6902`

* **Decoupled Patch Logic**: You keep your patch operations outside the core `kustomization.yaml`, making the overlay easier to read and maintain.
* **Versionable and Auditable**: You can track changes to patch logic independently in Git, making it easier to audit and review.
* **Reusable**: A single patch file can be reused across overlays (e.g., dev, stage, prod) or modularized for multi-team GitOps setups.
* **Better Structure for Large Teams**: This format is ideal in production-like environments where multiple people work on patch logic.

Use `patchesJson6902` when you need clear separation, better auditability, or long-term maintainability for your Kubernetes manifest changes.


---

## 3. `patchesStrategicMerge` (Deprecated)

This older approach uses partial Kubernetes manifest structure to override values. It was very common before JSON patching became mainstream but is now deprecated.

### Why Learn It

* Still found in production
* Simple to write
* You must know how to convert it to modern equivalents

---

Here is the complete and revised **Demo 5: `patchesStrategicMerge` (Deprecated)**, aligned with your shared base YAML structure, annotations, prefix/suffix, label injection, image updates, and namespace/replica overrides:

---

## Demo 5: `patchesStrategicMerge` (Deprecated)

### Background

`patchesStrategicMerge` was the original patching mechanism in Kustomize. It allows you to define patches using regular Kubernetes manifest structure. While still supported, it has been **officially deprecated**, and users are encouraged to migrate to the `patches` or `patchesJson6902` format.

That said, you’ll still encounter this format in many real-world repositories—especially older or legacy environments. Understanding how it works and how to migrate from it is essential.

---

### Project Structure

```text
├── base
│   ├── deploy.yaml
│   ├── svc.yaml
│   └── kustomization.yaml
└── overlays
    └── dev
        ├── patch-deploy.yaml
        └── kustomization.yaml
```

---

### overlays/dev/kustomization.yaml

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# Reference to shared base
resources:
  - ../../base

# Optional: Apply name prefix/suffix
nameSuffix: -dev
namePrefix: app1-

# Apply environment-based labels
labels:
  - pairs:
      env: dev
    includeSelectors: true

# Override container image globally
images:
  - name: nginx
    newName: nginx
    newTag: "latest"

# Set common annotations
commonAnnotations:
  branch: dev
  support: 800-800

# Target namespace
namespace: dev

# Scale deployment
replicas:
  - name: nginx-deploy
    count: 2

# Apply deprecated patch
patchesStrategicMerge:
  - patch-deploy.yaml
```

---

### overlays/dev/patch-deploy.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  template:
    spec:
      containers:
        - name: nginx
          env:
            - name: LOG_LEVEL
              value: debug
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
```

---

### What We're Doing

This patch:

* **Adds** a new environment variable `LOG_LEVEL=debug` to the `nginx` container.
* **Defines** explicit CPU and memory `requests` and `limits` for the container.

The patch file is written in standard Kubernetes manifest format, using only the fields we want to override or inject. This strategy is intuitive but lacks the flexibility of the more modern patching formats.

---

### Why This Is Deprecated

Strategic merge relies on predefined merge keys and schema awareness, which becomes brittle and limited with custom resources or advanced use cases. As a result, Kustomize has officially deprecated this method in favor of more flexible JSON-based patching via `patches` or `patchesJson6902`.

You will still find `patchesStrategicMerge` in many production environments, so it's important to:

* Know how it works
* Recognize its limitations
* Understand how to migrate away from it

---

### How to Migrate Using `kustomize edit fix`

To convert this deprecated patch to the modern syntax, run the following inside the `overlays/dev/` directory:

```bash
kustomize edit fix
```

This will:

* Convert `patchesStrategicMerge` to `patches` or `patchesJson6902`
* Preserve patch intent while using the recommended syntax
* Update `kustomization.yaml` accordingly

Use this command in version-controlled environments to simplify migration and ensure compliance with current Kustomize standards.

---

## Final Comparison

| Scenario or Use Case                          | Recommended Patching Method          | Reference |
| --------------------------------------------- | ------------------------------------ | --------- |
| Inline patching with maximum flexibility      | `patches` (generic form)             | Demo 3    |
| Declarative, file-based, and reusable patches | `patchesJson6902`                    | Demo 4    |
| Simpler syntax, legacy or existing codebases  | `patchesStrategicMerge` (deprecated) | Demo 5    |

---

### Notes:

* Use **`patches`** when patch logic is small, composable, or used in pipelines (e.g., GitOps).
* Use **`patchesJson6902`** when managing patches as reusable files across multiple overlays or environments.
* Use **`patchesStrategicMerge`** only if you’re maintaining legacy repositories or doing quick local experimentation — but migrate when possible.

Let me know if you’d like to add columns for Pros/Cons, CRD compatibility, or GitOps suitability.
