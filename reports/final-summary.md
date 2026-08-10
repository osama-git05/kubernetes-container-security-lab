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
---

# Final Project Security Summary

## Overall Result

The Kubernetes Container Security and Runtime Detection Lab was completed successfully.

The project demonstrated the complete defensive workflow:

**Deploy → Identify → Test → Detect → Harden → Retest → Document**

An intentionally insecure Kubernetes workload was analyzed, runtime monitoring was implemented using Falco, security controls were applied to create a hardened workload, and Falco detection coverage was extended using a custom rule.

---

## Insecure vs Hardened Configuration

| Security Control | Insecure Configuration | Hardened Configuration |
|---|---|---|
| User | Root (`uid=0`) | Non-root (`uid=101`) |
| Privileged mode | Enabled | Not enabled |
| Privilege escalation | Not restricted | `allowPrivilegeEscalation: false` |
| Linux capabilities | Default capabilities | All capabilities dropped |
| Root filesystem | Writable | Read-only |
| Temporary storage | General writable filesystem | Dedicated writable `/tmp` using `emptyDir` |
| CPU controls | None | Requests and limits configured |
| Memory controls | None | Requests and limits configured |
| NetworkPolicy | None initially | Default-deny ingress and egress policy |
| Runtime monitoring | None initially | Falco enabled |

---

## Falco Runtime Detection Results

### Sensitive File Access

Falco successfully detected access to:

```text
/etc/shadow
```

The alert identified information including:

- File path
- User
- Process
- Command
- Container
- Container image
- Kubernetes pod
- Namespace

This confirmed that Falco runtime monitoring was functioning correctly.

---

## Controlled Security Test Results

| Test | Initial Result |
|---|---|
| Interactive shell inside Nginx | Not detected by default rules |
| Write file below `/etc` | Not detected by default rules |
| Privileged pod deployment | Not detected by default rules |
| Read `/etc/shadow` | Detected |

The tests demonstrated that runtime monitoring tools do not automatically detect every activity. Detection depends on the enabled rules and the behavior being monitored.

---

# Custom Falco Detection Engineering

The original `/etc` file-write test did not generate an alert using the default Falco rules.

A custom rule was therefore created:

```text
Write below etc
```

The custom rule monitors write operations targeting files below:

```text
/etc
```

The rule was deployed through the Falco Helm configuration and successfully passed schema validation:

```text
/etc/falco/rules.d/custom-rules.yaml | schema validation: ok
```

The original activity was then repeated:

```bash
kubectl exec deployment/nginx-lab -- touch /etc/test_file_for_falco_rule
```

Falco successfully generated:

```text
Warning File below /etc opened for writing
```

with information including:

```text
file=/etc/test_file_for_falco_rule
user=root
process=touch
command=touch /etc/test_file_for_falco_rule
container=nginx
image=docker.io/library/nginx
namespace=default
```

---

## Detection Improvement

| Detection Stage | `/etc` Write Result |
|---|---|
| Default Falco rules | Not Detected |
| Custom Falco rule | Detected |

This demonstrates basic detection-engineering capability: identifying a gap in existing monitoring, creating a custom rule, deploying it, validating it, and retesting the original activity.

---

# Hardening Result

The hardened Kubernetes workload successfully implemented:

- Non-root execution
- Disabled privilege escalation
- Dropped Linux capabilities
- Read-only root filesystem
- Controlled writable `/tmp` storage
- CPU requests and limits
- Memory requests and limits
- Default-deny NetworkPolicy

During implementation, the read-only filesystem initially prevented Nginx from creating temporary files.

Rather than disabling the security control, an `emptyDir` volume was mounted at `/tmp`.

This allowed the application to function while preserving the read-only root filesystem.

---

# Final Outcome

The project demonstrated practical experience with:

- Kubernetes administration
- Docker and container concepts
- Kubernetes security contexts
- Container hardening
- Falco runtime security
- Custom Falco rule development
- Runtime detection
- Detection engineering
- NetworkPolicy
- Linux security controls
- Security testing
- Troubleshooting
- Git and GitHub
- Technical documentation

The final environment provides clear evidence of both vulnerable and hardened Kubernetes configurations and demonstrates how runtime security monitoring can be improved when default detection coverage is insufficient.

---

# Supporting Evidence

Detailed reports:

```text
reports/test-01-container-shell.md
reports/test-02-file-write-etc.md
reports/test-03-privileged-pod.md
reports/test-04-custom-falco-rule.md
reports/day6-hardening-summary.md
```

Falco configuration:

```text
falco/falco-custom-rules.yaml
falco/notes-and-alerts.md
```

Key custom detection evidence:

```text
screenshots/custom-falco-rule-detected.png
```
