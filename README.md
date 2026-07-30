# Kubernetes Cluster Locally with Minikube

## Objective
Deploy and manage a simple app (nginx) on a local Kubernetes cluster using
Minikube, demonstrating core Kubernetes concepts: deployments, services,
scaling, and inspecting pod status/logs.

## Tools Used
- Minikube
- kubectl
- Docker (as Minikube's driver)

## Project Structure
```
k8s-minikube-demo/
├── deployment.yaml       # Defines the nginx Deployment (2 replicas)
├── service.yaml           # Exposes the Deployment via NodePort
├── execution-logs.txt     # Sample terminal output for each command
└── screenshots/            # (add your own screenshots here)
```

## Prerequisites
1. **Docker Desktop** installed and running (Minikube can use Docker as its
   driver instead of a full VM — simplest option on Windows).
2. **Install kubectl**:
   ```
   winget install -e --id Kubernetes.kubectl
   ```
   Or download from https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/
   and add it to your System PATH (same approach as Terraform/Docker in
   earlier tasks).
3. **Install Minikube**:
   Download the Windows installer from
   https://minikube.sigs.k8s.io/docs/start/ (choose Windows → x86-64 →
   installer, or use `winget install minikube`), then add it to PATH if
   needed. Verify with:
   ```
   minikube version
   kubectl version --client
   ```

## Steps to Run

### 1. Start the cluster
```bash
minikube start
```
This spins up a single-node Kubernetes cluster inside a Docker container.
First run downloads images and can take a few minutes.

### 2. Deploy the app
```bash
kubectl apply -f deployment.yaml
```
Creates a Deployment named `nginx-deployment` running 2 replica pods of
`nginx:latest`.

### 3. Expose the app
```bash
kubectl apply -f service.yaml
```
Creates a `NodePort` Service so the app is reachable from outside the
cluster, on port `30080`.

### 4. Verify pods are running
```bash
kubectl get pods
```
Should show 2 pods with `STATUS: Running`.

### 5. Access the app
```bash
minikube service nginx-service --url
```
Copy the printed URL into a browser — you should see the nginx welcome
page.

### 6. Scale the deployment
```bash
kubectl scale deployment nginx-deployment --replicas=4
kubectl get pods
```
You should now see 4 pods instead of 2, demonstrating horizontal scaling.

### 7. Inspect a pod (logs & events)
```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```
`describe` shows scheduling events, container state, and any errors.
`logs` shows the actual stdout/stderr from the container — useful for
debugging a crashing app.

### 8. Clean up
```bash
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
minikube stop
```

## What's in `deployment.yaml`
- **`replicas: 2`** — runs 2 identical pods for redundancy.
- **`selector` / `labels`** — how the Deployment finds and manages its pods.
- **`resources`** — CPU/memory requests and limits, good practice even in
  a local demo.

## What's in `service.yaml`
- **`type: NodePort`** — exposes the app on a fixed port (`30080`) on the
  Minikube node itself, reachable via `minikube service` or the node's IP.
- **`selector: app: nginx-demo`** — routes traffic to any pod matching
  this label, regardless of how many replicas exist.

## Execution Logs & Screenshots
See `execution-logs.txt` for expected output at each step — **replace it
with your own real terminal output** once you run these commands.

Also take actual screenshots of:
- `kubectl get pods` (showing 2, then 4 replicas after scaling)
- `kubectl get services`
- The nginx welcome page in a browser
- `kubectl describe pod ...` output

Save them in a `screenshots/` folder and reference them in this README,
e.g.:
```markdown
![Pods running](screenshots/pods-running.png)
```

## Notes / Learnings
- A **Deployment** manages a set of identical pods and handles restarts,
  rolling updates, and scaling automatically — you rarely create raw Pods
  directly in real projects.
- A **Service** gives pods a stable network identity, since individual
  pod IPs change every time they're recreated.
- `kubectl scale` changes the desired replica count; Kubernetes then
  creates or terminates pods to match that number.
- `kubectl describe` is usually the first place to look when a pod is
  stuck in `Pending` or `CrashLoopBackOff` — the Events section at the
  bottom explains why.
- `minikube start` uses Docker as its driver here, so Docker Desktop must
  be running first — same requirement as Tasks 1-3.

## Pipeline Status
✅ Cluster started, app deployed, exposed, scaled, and inspected successfully.

- `minikube start` → cluster came up on Docker driver, Kubernetes v1.35.1
- `kubectl apply -f deployment.yaml` → 2/2 pods `Running`
- `kubectl apply -f service.yaml` → `nginx-service` exposed on NodePort
- `minikube service nginx-service --url` → nginx welcome page confirmed in browser
- `kubectl scale deployment nginx-deployment --replicas=4` → scaled to 4/4 `Running` pods
- `kubectl logs <pod>` → clean nginx startup logs, no errors
- `kubectl describe pod <pod>` → confirmed scheduling + container events
- `kubectl delete` + `minikube stop` → cleaned up successfully

### Issues encountered & fixed (for anyone reviewing this repo)
1. **`winget install minikube` — "No package found matching input criteria."**
   The winget package ID/index didn't resolve the package as expected.
   Fixed by skipping winget entirely and downloading the official Windows
   installer directly from minikube.sigs.k8s.io, which handles PATH setup
   automatically.
2. **Non-fatal warning during `minikube start`:**
   ```
   E0730 ... Unable to scale down deployment "coredns" ...
   the object has been modified; please apply your changes to the latest
   version and try again
   ```
   This is a known, cosmetic timing conflict when Minikube tries to scale
   CoreDNS during startup — it does **not** prevent the cluster from
   starting successfully. Confirmed the cluster was healthy afterward via
   `kubectl get nodes` (status `Ready`) and `kubectl get pods -A` (all
   system pods `Running`), so no action was needed beyond verifying.

Both issues were environment/tooling quirks rather than problems with the
`deployment.yaml` / `service.yaml` manifests themselves, which applied and
ran correctly on the first try once the cluster was up.
