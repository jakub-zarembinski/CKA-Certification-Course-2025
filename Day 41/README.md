# Day 41: Pod Security – Security Context and Linux Capabilities | CKA Course 2025

## Video reference for Day 41 is the following:

---
## ⭐ Support the Project  
If this **repository** helps you, give it a ⭐ to show your support and help others discover it! 

---
### Why Pod Security?

Kubernetes lets you run containers easily, but it does not secure them by default. If you spin up a basic pod, it often runs as the `root` user inside the container, which can be dangerous.

Let’s try it out:

```bash
kubectl run debugpod --image=busybox -it --restart=Never -- sh
```

Then run:

```bash
id
```

Output:

```
uid=0(root) gid=0(root) groups=0(root),10(wheel)
```

It means:

* **`uid=0(root)`**: The user ID is 0, which corresponds to the `root` user.
* **`gid=0(root)`**: The primary group ID is also 0, representing the `root` group.
* **`groups=0(root),10(wheel)`**:

  * The user belongs to both the `root` group (ID 0) and the `wheel` group (ID 10).
  * The `wheel` group traditionally grants users the ability to execute `sudo` or switch to root.

> In summary: UID 0 is what makes a user `root`. It's not the username but the UID that determines privilege. The rest (username or group names) are just labels mapped from files like `/etc/passwd` or `/etc/group`.


**But, what is the risk with using the `root` user?**

By default, this pod runs as the **root user** inside the container — which poses several security risks. Running containers as root is **not recommended** because:

* **Privilege Escalation**: If the container is compromised, an attacker could attempt to escalate privileges and break out of the container.
* **Access to Host Resources**: In less strictly isolated environments (e.g., without user namespaces or Pod Security Standards), a root container may gain elevated access to the host system.
* **Accidental Misuse**: Applications running as root can unintentionally modify critical files or perform actions that would otherwise be restricted.
* **Non-compliance**: Many regulatory and security standards require workloads to run with the least privilege possible.

---

**Important Note**

By default, a pod may run as the **root user** inside the container — which can pose serious security risks. However, **not all pods run as root**, even if no explicit security settings are applied in the pod manifest. This is because:

* **Some container images are designed to run as non-root** by default. Their Dockerfiles may include instructions like `USER 1000`, which ensures the container process does not start with UID 0.
* **Security-conscious base images** (e.g., `distroless`, `bitnami`) often avoid running processes as root by design.

Still, relying solely on the image to enforce non-root execution is **not sufficient** for securing Kubernetes workloads.

As DevOps and Infrastructure engineers, we must:

* **Explicitly define security context fields** (like `runAsUser`, `runAsNonRoot`, etc.) in the **pod manifest** to ensure pods do not run as root — regardless of how the container image was built.
* **Enforce policies** using tools like Pod Security Standards, OPA Gatekeeper, or Kyverno to validate that workloads adhere to these constraints.

This ensures consistency, defense in depth, and aligns with the **principle of least privilege**, reducing the potential impact of a container compromise.


---

## Pod Security

Hardening Kubernetes workloads begins with **Pod Security** — ensuring that containers run with the least privilege required. Pod security helps prevent containers from escalating privileges or accessing sensitive host resources, especially in shared or multi-tenant clusters.

Kubernetes provides two primary mechanisms to control pod-level privileges:

1. **Security Context** (Pod and Container level)
2. **Linux Capabilities** (Container level)

These together form the foundation for enforcing the principle of least privilege, reducing the attack surface, and achieving compliance and runtime security in production environments.

---

## 1. Security Context

A **Security Context** defines privilege and access control settings for a Pod or individual containers. It allows you to configure:

* The user and group IDs containers should run as (`runAsUser`, `runAsGroup`)
* Whether containers must run as non-root users (`runAsNonRoot`)
* Volume file ownership through filesystem group (`fsGroup`)
* Privilege escalation restrictions (`allowPrivilegeEscalation`)
* Filesystem mutability (`readOnlyRootFilesystem`)

Security context settings can be applied at:

* **Pod level** — affects all containers in the Pod
* **Container level** — overrides Pod-level settings for that specific container

---

## Security Context in Kubernetes: Field-by-Field Overview

Kubernetes Security Context allows fine-grained control over pod and container behavior with respect to user identity, filesystem access, and privileges. Below is a concise breakdown organized by where each setting applies.

---

### Pod-Level Fields (`spec.securityContext`)

* **`runAsUser`**: Sets the UID for all containers in the Pod. Example: `1000`. No username mapping is required inside the container.

  > Recommended: Use a non-zero UID (e.g., `1000`) to avoid running as root.

* **`runAsGroup`**: Sets the primary GID for container processes. Useful for managing file permissions and shared access.

  > Recommended: Set an appropriate non-root GID to match mounted volumes or app requirements.

* **`fsGroup`**: Assigns a supplemental GID to all processes, granting group ownership to mounted volumes. Files created inside those volumes will belong to this group.

  > Recommended: Set to allow non-root users access to shared mounted volumes.

* **`runAsNonRoot`**: Enforces non-root execution. If omitted and the image defaults to UID 0 (root), the pod fails to start. Acts as a safeguard against unintentional root access.

  > Recommended: Always set to `true` to enforce non-root user execution.

---

### Container-Level Fields (`spec.containers[].securityContext`)

* **`allowPrivilegeEscalation`**: Prevents processes in the container from gaining additional privileges (e.g., via `setuid` binaries).
  In Linux, **privilege escalation** means a process can gain more powerful permissions than it started with — even if it didn’t start as root. This can happen through special mechanisms like **setuid binaries** (binaries that run with the file owner’s permissions).
  Suppose there's a binary owned by `root` with the `setuid` bit set. Even if a **non-root** user runs it, it will execute with **root privileges**.
  Without protection, your container could elevate its privileges even if it starts with a safe UID.

  > Recommended: Set to `false` unless the application explicitly needs privilege escalation.

* **`privileged`**: Grants full host access to the container — including kernel capabilities, device access, and other unrestricted system-level controls. Equivalent to “no isolation.”

  > Recommended: Avoid setting to `true` unless necessary for privileged workloads (e.g., low-level networking or storage operations).

* **`readOnlyRootFilesystem`**: Mounts the container’s root filesystem as read-only. Prevents any write operations to system directories like `/`, `/etc`, `/var`, helping protect the container from tampering or accidental changes.

  > Recommended: Set to `true` for stateless containers or apps that don’t require root filesystem writes.



Each of these settings is part of the broader security design in Kubernetes and should be tuned according to your application’s privilege needs and security posture. Where overlap exists (e.g., `runAsUser` defined at both Pod and Container level), **container-level settings take precedence**.

---

### Scope of Security Context Fields: Pod vs. Container

In Kubernetes, the `securityContext` allows you to fine-tune privileges and access control at both the Pod and Container levels. However, not all fields are applicable in both scopes, and it’s crucial to understand which attributes can be set where—especially when crafting security policies or debugging unexpected behavior. The table below offers a clear breakdown of field applicability, helping you write minimal and secure pod specs with precision.



| Field                      | Pod Level | Container Level | Notes                                                                 |
|---------------------------|-----------|------------------|-----------------------------------------------------------------------|
| `runAsUser`               | ✅         | ✅                | Container overrides Pod                                               |
| `runAsGroup`              | ✅         | ✅                | Container overrides Pod                                               |
| `runAsNonRoot`            | ✅         | ✅                | Container overrides Pod                                               |
| `fsGroup`                 | ✅         | ❌                | Pod-level only; affects volume ownership                              |
| `allowPrivilegeEscalation`| ❌         | ✅                | Container-level only                                                  |
| `privileged`              | ❌         | ✅                | Container-level only                                                  |
| `readOnlyRootFilesystem`  | ❌         | ✅                | Container-level only                                                  |

✅ = Supported  
❌ = Not supported

#### Key Clarifications:
- **Pod-level `securityContext`** is defined under `spec.securityContext` and applies to all containers unless overridden.
- **Container-level `securityContext`** is defined under `spec.containers[].securityContext` and takes precedence.
- Fields like `allowPrivilegeEscalation`, `privileged`, `capabilities`, and `readOnlyRootFilesystem` are **not valid** at the Pod level—they must be set per container.
- `fsGroup` is strictly Pod-level and is used to set group ownership on mounted volumes.


---

## Demo: Security Context

In this demo, we will apply **security context settings** at both the **Pod** and **Container** levels and explore how they influence the behavior of the containerized application. This is a hands-on way to understand how various security-related configurations affect runtime identity, access, and capabilities.

### Step 1: Deploy a Pod with Security Contexts Defined

We will create a pod named `secure-pod` with the following security context configurations:

* **Pod-level Security Context** (applies to all containers unless overridden):

  * `runAsUser`: Specifies the UID that the container process runs as.
  * `runAsGroup`: Specifies the primary GID the container runs as.
  * `fsGroup`: Specifies the GID that owns mounted volumes.
  * `runAsNonRoot`: Ensures the container will not start if it tries to run as UID 0 (root).

* **Container-level Security Context**:

  * `allowPrivilegeEscalation: false`: Prevents the container from gaining additional privileges.
  * `readOnlyRootFilesystem: true`: Mounts the container’s root filesystem as read-only.

Here is the complete YAML manifest:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
    runAsNonRoot: true
  containers:
    - name: app
      image: busybox
      command: ["sh", "-c", "sleep 3600"]
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
```

Apply this manifest using:

```bash
kubectl apply -f secure-pod.yaml
```

---

### Step 2: Verification

Once the pod is running, exec into it:

```bash
kubectl exec -it secure-pod -- sh
```

Run the `id` command:

```bash
~ $ id
uid=1000 gid=3000 groups=2000,3000
```

#### Explanation:

* `uid=1000`: This comes from `runAsUser: 1000` in the pod’s security context. It means the main process is running as user ID 1000.
* `gid=3000`: This is from `runAsGroup: 3000`, the primary group ID assigned to the process.
* `groups=2000,3000`:

  * `2000` is from `fsGroup`, which grants access to mounted volumes.
  * `3000` is from `runAsGroup`, assigned as the primary GID.

> Note: The `fsGroup` setting changes the group ownership of mounted volumes, enabling read/write access as needed for shared volumes or storage mounts.

#### Identity & Root User Clarification:

* The UID `0` is reserved for the **root** user in Linux.
* In our case, we explicitly disallowed root by using `runAsNonRoot: true`, ensuring the pod fails to start if a root UID is attempted.

---

### Step 3: Test Read-Only Root Filesystem

Try creating a directory:

```bash
~ $ mkdir test_dir
mkdir: can't create directory 'test_dir': Read-only file system
```

This error is expected because we set `readOnlyRootFilesystem: true`. It makes the root filesystem (`/`) immutable, including directories like `/etc`, `/var`, `/tmp`, etc.

This improves security by preventing runtime changes to binaries or config files inside the container.

---

### Step 4: Test Privilege Escalation

Try escalating privileges using common methods:

```bash
~ $ sudo su
sh: sudo: not found

~ $ su -
su: must be suid to work properly
```

These attempts fail because:

* The image does not include `sudo` or `su` by default.
* Even if `su` was available, `allowPrivilegeEscalation: false` prevents privilege gain via setuid binaries.

---

### Step 5: Understanding `whoami`

Run:

```bash
~ $ whoami
whoami: unknown uid 1000
```

Explanation:

* The container is running as UID 1000, but there’s no username associated with this UID in `/etc/passwd` inside the busybox image.
* The `whoami` command tries to resolve UID → username using `/etc/passwd`, and fails.

This is **not an error** — Linux security is based on **UIDs and GIDs**, not usernames. Username is optional and only needed for human readability.

---


## 2. Linux Capabilities

In Linux, **root** is an all-powerful user — but in reality, root privileges are made up of many distinct low-level permissions called **capabilities**. Kubernetes allows you to selectively manage these capabilities at the **container level**, giving you fine-grained control over what a container can and cannot do.

> Capabilities are configured via the `securityContext.capabilities` field and are only supported **at the container level** — not at the pod level.

By default, containers inherit a reduced set of Linux kernel capabilities compared to a full root user. However, they still retain some sensitive capabilities (like `NET_RAW` or `SYS_ADMIN`) unless explicitly dropped.

With Kubernetes, you can use the `capabilities` field inside the `securityContext` to:

* **Drop unnecessary kernel privileges** (`drop: ["ALL"]`)
* **Add only what's needed** (`add: ["NET_BIND_SERVICE"]`)

This gives fine-grained control over what your containerized process can do — beyond what basic UID/GID settings offer.

---

### What Are Linux Capabilities?

Instead of giving full root access to a container, Linux capabilities allow you to **drop unnecessary privileges** and optionally **add only the ones required**.

This aligns with the **principle of least privilege**, helping reduce the attack surface and prevent containers from performing dangerous operations.

---

### Common Linux Capabilities

| Capability     | Description                                                                                     |
| -------------- | ----------------------------------------------------------------------------------------------- |
| `NET_ADMIN`    | Modify network interfaces, routing tables, and firewall rules                                   |
| `SYS_ADMIN`    | Very broad; includes mounting filesystems, changing kernel params, system configuration changes |
| `SYS_TIME`     | Set system clock, modify hardware clock, manage time sync services (e.g., NTP)                  |
| `CHOWN`        | Change file ownership using `chown`                                                             |
| `SETUID`       | Change user ID of a running process                                                             |
| `DAC_OVERRIDE` | Override standard file permission checks (Discretionary Access Control)                         |

> Base images may include some of these by default. **Explicitly dropping** unneeded capabilities is key to reducing the container’s privilege footprint.

---

## Demo: Linux Capabilities

In this hands-on demo, we'll create a pod that **drops all Linux capabilities** and **adds back only a few required ones**, then verify:

* Operations that succeed because of the explicitly added capabilities
* Operations that fail because all others were dropped

This approach aligns with the **least privilege principle**.

---

### Step 1: Deploy a Pod With Select Capabilities

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cap-pod
spec:
  containers:
    - name: app
      image: busybox
      command: ["sh", "-c", "sleep 3600"]
      securityContext:
        allowPrivilegeEscalation: false
        capabilities:
          drop:
            - ALL
          add:
            - NET_ADMIN
            - SYS_TIME
```

Apply the manifest:

```bash
kubectl apply -f cap-pod.yaml
```

---

### Step 2: Exec Into the Pod

```bash
kubectl exec -it cap-pod -- sh
```

---

### Step 3: Verify Added Capabilities Work

**1. Test `SYS_TIME`:**

```bash
date -s "2025-06-01 12:00:00"
```

If `CAP_SYS_TIME` is honored by the runtime, the system time inside the container should update.
*Note: It may not work on all environments (e.g., kind, OpenShift, hardened GKE clusters).*

**2. Test `NET_ADMIN`:**

```bash
ip link set eth0 down
```

Expected behavior: the interface should go down (in permissive environments).

---

### Step 4: Verify Dropped Capabilities Fail

Because we dropped **all other capabilities**, attempts to perform privileged operations that require other capabilities should fail.

**1. Test `CHOWN`:**

```bash
touch myfile && chown 1000:1000 myfile
```

Expected error:

```
chown: myfile: Operation not permitted
```

---

### Key Takeaway

With this single pod, we demonstrate that **only explicitly added capabilities** are available to the container. Everything else is denied — proving the enforcement of **tight privilege boundaries** via:

```yaml
capabilities:
  drop:
    - ALL
  add:
    - NET_ADMIN
    - SYS_TIME
```

Let me know if you'd like a version that includes `CAP_CHOWN` or `CAP_SYS_ADMIN` as a bonus test.
