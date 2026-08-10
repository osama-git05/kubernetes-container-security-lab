# Kubernetes Container Security Lab - Security Findings

## Day 3 - Insecure Pod Findings

### Objective

Deploy an intentionally insecure Kubernetes pod and identify common container security weaknesses before applying hardening controls.

The pod was configured to demonstrate several insecure practices that may increase the impact of a container compromise.

---

## Finding 1 - Container Running as Root

The user identity inside the container was checked using:

```bash
kubectl exec insecure-pod -- id
```

The command returned:

```text
uid=0(root)
```

This confirms that the container is running as the root user.

### Security Risk

Running containers as root increases the potential impact of a compromise. If an attacker gains control of the application or container process, they may have elevated privileges within the container.

### Evidence

![Insecure Pod Running as Root](../screenshots/day3-insecure-root.png)

---

## Finding 2 - Privileged Container

The `insecure-pod.yaml` manifest contains the following security configuration:

```yaml
securityContext:
  privileged: true
```

This confirms that privileged mode is enabled for the container.

### Security Risk

Privileged containers receive significantly greater access to the underlying system and weaken normal container isolation.

A compromised privileged container may therefore present a greater risk to the Kubernetes node than a standard restricted container.

### Evidence

![Privileged Security Context](../screenshots/day3-privileged-context.png)

---

## Finding 3 - No Resource Limits

The insecure pod does not define any CPU or memory resource requests or limits.

There is no `resources` configuration in the container specification.

### Security Risk

Without resource restrictions, a container may consume excessive CPU or memory.

This can reduce the resources available to other workloads and may negatively affect cluster stability.

---

## Finding 4 - No NetworkPolicy

The cluster NetworkPolicy configuration was checked using:

```bash
kubectl get networkpolicy
```

At this stage of the project, no NetworkPolicy had been configured.

### Security Risk

Without a NetworkPolicy, this project does not currently restrict pod network traffic.

This provides broader network communication than may be required for the workload.

---

## Insecure Pod Configuration

The intentionally insecure workload is defined in:

```text
manifests/insecure-pod.yaml
```

The pod uses an Ubuntu container and explicitly enables privileged mode.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: insecure-pod
  labels:
    app: insecure-pod
spec:
  containers:
    - name: insecure-container
      image: ubuntu:24.04
      command: ["sleep", "3600"]
      securityContext:
        privileged: true
```

---

## Security Findings Summary

| Security Control | Insecure Configuration | Security Concern |
|---|---|---|
| Container user | Root (`uid=0`) | Increased privileges if compromised |
| Privileged mode | Enabled | Weakened container isolation |
| Resource limits | None | Potential excessive CPU or memory consumption |
| NetworkPolicy | None | Pod network traffic is not restricted by the project |

---

## Day 3 Result

The intentionally insecure Kubernetes pod was successfully deployed and inspected.

Four security weaknesses were identified:

- The container runs as the root user.
- Privileged mode is enabled.
- CPU and memory limits are not configured.
- No Kubernetes NetworkPolicy has been applied.

These findings establish the insecure baseline for the project.

Later stages of the lab will introduce a hardened workload using controls such as non-root execution, disabled privilege escalation, dropped Linux capabilities, a read-only root filesystem, resource limits, and a NetworkPolicy.

The insecure and hardened configurations will then be compared to demonstrate the effect of Kubernetes container security controls.
