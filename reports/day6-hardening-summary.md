# Day 6 - Kubernetes Workload Hardening

## Objective

Replace the intentionally insecure container configuration with a hardened Kubernetes workload and verify that the security controls are working correctly.

---

## Hardened Pod

The hardened workload is defined in:

```text
manifests/hardened-pod.yaml
```

The hardened container uses the unprivileged Nginx image:

```text
nginxinc/nginx-unprivileged:alpine
```

---

## Security Controls Applied

### 1. Non-Root Execution

The container was configured to run using a non-root user:

```yaml
runAsNonRoot: true
runAsUser: 101
runAsGroup: 101
```

The configuration was verified using:

```bash
kubectl exec hardened-pod -- id
```

The output confirmed that the container was running with a non-zero UID rather than `uid=0(root)`.

### Security Benefit

Running containers as non-root reduces the privileges available to a compromised application or process.

---

## 2. Privilege Escalation Disabled

The following control was configured:

```yaml
allowPrivilegeEscalation: false
```

### Security Benefit

This prevents processes inside the container from gaining additional privileges through mechanisms such as setuid binaries.

---

## 3. Linux Capabilities Dropped

All default Linux capabilities were removed:

```yaml
capabilities:
  drop:
    - ALL
```

### Security Benefit

Dropping unnecessary Linux capabilities reduces the actions available to processes running inside the container.

---

## 4. Read-Only Root Filesystem

The container root filesystem was configured as read-only:

```yaml
readOnlyRootFilesystem: true
```

### Initial Issue

During the first deployment attempt, the hardened Nginx container failed because Nginx attempted to create temporary files inside `/tmp`.

The error included:

```text
mkdir() "/tmp/proxy_temp" failed (30: Read-only file system)
```

### Resolution

The read-only root filesystem control was kept enabled.

Instead of disabling the security control, a temporary writable volume was mounted only at `/tmp`:

```yaml
volumeMounts:
  - name: nginx-tmp
    mountPath: /tmp

volumes:
  - name: nginx-tmp
    emptyDir: {}
```

After this change, the hardened pod successfully entered the `Running` state.

### Security Benefit

The main container filesystem remains read-only while Nginx receives only the temporary writable storage required for normal operation.

---

## 5. Resource Requests and Limits

CPU and memory controls were configured:

```yaml
resources:
  requests:
    cpu: "50m"
    memory: "64Mi"
  limits:
    cpu: "100m"
    memory: "128Mi"
```

### Security Benefit

Resource limits reduce the risk that one container can consume excessive CPU or memory and affect other workloads.

---

## Verification

The hardened pod status was checked using:

```bash
kubectl get pod hardened-pod
```

Result:

```text
Running
```

The container identity was checked using:

```bash
kubectl exec hardened-pod -- id
```

The result confirmed that the workload was running as a non-root user.

The main security controls were inspected using:

```bash
kubectl get pod hardened-pod -o yaml | grep -E "runAsNonRoot|runAsUser|runAsGroup|allowPrivilegeEscalation|readOnlyRootFilesystem|drop:|cpu:|memory:"
```

---

## Evidence

![Hardened Pod Non Root](../screenshots/day6-hardened-nonroot.png)

![Hardened Security Context](../screenshots/day6-hardened-security-context.png)

---

## Before and After

| Security Control | Insecure Pod | Hardened Pod |
|---|---|---|
| User | Root (`uid=0`) | Non-root (`uid=101`) |
| Privileged mode | Enabled | Not enabled |
| Privilege escalation | Not restricted | Disabled |
| Linux capabilities | Default capabilities | All dropped |
| Root filesystem | Writable | Read-only |
| CPU limits | None | Configured |
| Memory limits | None | Configured |
| Writable filesystem | Broad | `/tmp` only where required |

---

## Result

The hardened Kubernetes workload was successfully deployed.

The workload now runs as a non-root user, blocks privilege escalation, drops Linux capabilities, uses a read-only root filesystem, and applies CPU and memory resource limits.

An initial compatibility issue with the read-only filesystem was resolved by providing Nginx with a dedicated temporary `/tmp` volume while preserving the security control.

The hardened configuration provides a significantly more restricted environment than the intentionally insecure pod.
