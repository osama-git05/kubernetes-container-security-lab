# Test 3 - Privileged Pod Deployment

## Objective

Determine whether an intentionally privileged Kubernetes pod can be deployed successfully and whether the activity is detected by the installed default Falco rules.

## Commands

The existing insecure pod was removed:

```bash
kubectl delete pod insecure-pod --ignore-not-found
```

The privileged pod was then redeployed using:

```bash
kubectl apply -f insecure-pod.yaml
```

Its status was verified using:

```bash
kubectl get pod insecure-pod
```

## Expected Result

The privileged pod should be created successfully and enter the `Running` state.

A Falco warning may be generated depending on the default rules enabled in the installed Falco version.

## Actual Result

The privileged pod was successfully recreated and entered the `Running` state.

Falco logs were reviewed using:

```bash
kubectl logs -n falco -l app.kubernetes.io/name=falco -c falco --since=5m | grep Warning
```

No corresponding warning was generated.

## Falco Detection

**Not detected**

The installed default Falco rules did not generate a warning for the privileged pod deployment.

This is recorded as a configuration finding and detection coverage limitation.

## Security Risk

Privileged containers weaken normal container isolation and provide significantly greater access to the underlying host environment.

If a privileged container is compromised, the potential impact may be significantly greater than for a restricted container.

## Recommended Mitigation

- Avoid privileged containers unless there is a strict operational requirement.
- Use non-root execution where possible.
- Disable privilege escalation.
- Drop unnecessary Linux capabilities.
- Apply Pod Security Admission or similar policy controls.
- Use admission policies to prevent unauthorized privileged workloads.
- Add custom Falco rules if privileged workload creation should generate a runtime alert.

## Test Result

**Successful test / No Falco alert generated**

The privileged pod was successfully deployed, but no default Falco warning was generated for this activity.

## Evidence

![Privileged Pod Test](../screenshots/day5-test3-privileged-pod.png)
