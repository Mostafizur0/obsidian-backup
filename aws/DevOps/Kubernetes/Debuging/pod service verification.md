[[Debug Error]]

# How to Verify a Pod is Exposed to a Service Using kubectl

This tutorial walks you through how to ensure a Kubernetes Pod is successfully receiving traffic from a Service using `kubectl`. 

Kubernetes connects Services to Pods automatically using **label selectors**. If the labels match and the Pod is healthy, the control plane adds the Pod's IP to the Service's routing list. 

---

## Step 1: Check the Service Endpoints
The fastest way to verify connection is to see if the Service has registered any backend Pod IPs.

```bash
kubectl get endpoints <service-name>
```

### What to Look For:
* **Successful Connection:** You will see a list of IP addresses and ports under the `ENDPOINTS` column (e.g., `10.244.0.15:8080`).
* **Disconnection:** If the column displays `<none>` or remains empty, the Service is **not** routing traffic to any Pods.

---

## Step 2: Cross-Reference the Pod's IP
To confirm that a *specific* Pod is the one receiving traffic from the Service, match its internal IP address against the endpoint list.

1. **Find your Pod's internal IP address:**
   ```bash
   kubectl get pod <pod-name> -o wide
   ```
2. **Compare the outputs:** Locate the value under the `IP` column and verify that it matches one of the IP addresses listed in the `kubectl get endpoints` command from Step 1.

---

## Step 3: Inspect Selectors and Labels
If the endpoints list is empty or the IP is missing, there is likely a label mismatch. You can audit the exact matching configuration using `describe`.

### 1. View the Service Selector
Inspect the Service to see what labels it is looking for:
```bash
kubectl describe service <service-name>
```
Look for the **`Selector`** line in the output:
```yaml
Selector:          app=web-backend
```

### 2. View the Pod Labels
Inspect the target Pod to see what labels it actually carries:
```bash
kubectl describe pod <pod-name>
```
Look at the **`Labels`** section at the very top of the output:
```yaml
Labels:            app=web-backend
                   environment=production
```
*The Pod must possess **all** key-value pairs specified in the Service's Selector to successfully route traffic.*

---

## Common Troubleshooting Steps

* **Label Mismatch (Case Sensitivity):** Kubernetes labels are strictly case-sensitive. A selector looking for `app=web` will fail to match a Pod labeled `App=web`.
* **Port Mismatches:** Ensure the `targetPort` defined in your Service YAML precisely matches the port your containerized application is listening on inside the Pod.
* **Failing Readiness Probes:** If your Pod defines a `readinessProbe` and it is failing, Kubernetes will automatically strip the Pod's IP from the Service endpoints list to protect users from broken traffic. Ensure the Pod shows `Ready: 1/1` in `kubectl get pods`.