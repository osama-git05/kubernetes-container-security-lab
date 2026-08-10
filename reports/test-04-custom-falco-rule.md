# Test 04 - Custom Falco Rule Detection

## Objective

Create and validate a custom Falco rule capable of detecting file-write activity below the `/etc` directory.

This test was performed because the same activity was not detected by the default Falco rules during the original security testing.

---

## Original Test Result

The following command had previously been executed:

```bash
kubectl exec deployment/nginx-lab -- sh -c 'touch /etc/test_file_for_falco_rule'
```

### Original Falco Result

**Not Detected**

The default Falco rules did not generate a warning for this activity.

---

## Custom Rule

A custom Falco rule named:

```text
Write below etc
```

was created in:

```text
falco/falco-custom-rules.yaml
```

The rule monitors file-write operations below:

```text
/etc
```

and records useful context including:

- File path
- User
- Process
- Command
- Container
- Container image
- Kubernetes pod
- Kubernetes namespace

---

## Custom Rule Deployment

The custom rule was added to the existing Falco Helm deployment using:

```bash
helm upgrade falco falcosecurity/falco \
  --namespace falco \
  --reuse-values \
  -f falco/falco-custom-rules.yaml
```

Falco was then verified as running successfully.

The rule file passed Falco schema validation:

```text
/etc/falco/rules.d/custom-rules.yaml | schema validation: ok
```

---

## Retest

The original test file was removed:

```bash
kubectl exec deployment/nginx-lab -- rm -f /etc/test_file_for_falco_rule
```

The same file-write activity was then repeated:

```bash
kubectl exec deployment/nginx-lab -- touch /etc/test_file_for_falco_rule
```

Falco logs were checked using:

```bash
kubectl logs -n falco \
  -l app.kubernetes.io/name=falco \
  -c falco \
  --since=1m | grep "File below /etc"
```

---

## Detection Result

**Detected**

Falco generated the following warning:

```text
Warning File below /etc opened for writing |
file=/etc/test_file_for_falco_rule
user=root
process=touch
command=touch /etc/test_file_for_falco_rule
container=nginx
image=docker.io/library/nginx
namespace=default
```

Falco also identified the associated Kubernetes pod and container information.

---

## Before and After

| Stage | Activity | Result |
|---|---|---|
| Default Falco rules | Write file below `/etc` | Not Detected |
| Custom Falco rule | Write file below `/etc` | Detected |

---

## Security Significance

This test demonstrates that runtime detection coverage can be extended when default rules do not cover a specific behavior.

Instead of relying only on built-in detections, a custom rule was created to monitor behavior relevant to the security requirements of the environment.

This demonstrates basic detection-engineering skills using Falco.

---

## Evidence

![Custom Falco Rule Detection](../screenshots/custom-falco-rule-detected.png)

---

## Result

The custom Falco rule was successfully deployed, validated, and tested.

The same activity that previously generated no warning was successfully detected after the custom rule was introduced.
