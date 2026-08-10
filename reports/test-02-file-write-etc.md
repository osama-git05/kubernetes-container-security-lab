# Test 2 - File Written Below /etc

## Objective

Determine whether a file can be created below the `/etc` directory inside the Nginx container and whether the activity is detected by the installed default Falco rules.

## Command

```bash
kubectl exec deployment/nginx-lab -- sh -c 'touch /etc/test_file_for_falco_rule'
```

Falco logs were then checked using:

```bash
kubectl logs -n falco -l app.kubernetes.io/name=falco -c falco --since=5m | grep Warning
```

## Expected Result

The file should be successfully created inside `/etc`.

A Falco warning may be generated depending on the rules enabled in the installed Falco version.

## Actual Result

The command completed successfully and the file was created inside the Nginx container.

No corresponding warning was returned when the Falco logs were checked.

## Falco Detection

**Not detected**

No default Falco warning was generated for the `/etc` file-write activity in this installation.

This result is recorded as a detection coverage limitation rather than a failed security test.

## Security Risk

Unexpected modification of files below `/etc` may indicate configuration tampering or unauthorized changes inside a container.

An attacker with command execution access could potentially modify configuration files or other system-related files.

## Recommended Mitigation

- Run containers with a read-only root filesystem where possible.
- Run containers as non-root users.
- Restrict Kubernetes `exec` permissions using RBAC.
- Use custom Falco rules where additional file-write monitoring is required.
- Minimize unnecessary write permissions inside production containers.

## Test Result

**Successful test / No Falco alert generated**

The file-write action succeeded, but the installed default Falco rules did not generate a warning for this activity.

## Evidence

![File Written Below /etc](../screenshots/day5-test2-file-write-etc.png)
