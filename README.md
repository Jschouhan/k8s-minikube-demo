Kubernetes Cluster Locally with Minikube
Objective
Deploy and manage a simple app (nginx) on a local Kubernetes cluster using Minikube, demonstrating core Kubernetes concepts: deployments, services, scaling, and inspecting pod status/logs.
Tools Used
Minikube
kubectl
Docker (as Minikube's driver)
Project Structure
Code
Prerequisites
Docker Desktop installed and running (Minikube can use Docker as its driver instead of a full VM — simplest option on Windows).
Install kubectl:
Code
Or download from https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/ and add it to your System PATH (same approach as Terraform/Docker in earlier tasks).
Install Minikube:
Download the Windows installer from https://minikube.sigs.k8s.io/docs/start/ (choose Windows → x86-64 → installer, or use winget install minikube), then add it to PATH if needed. Verify with:
Code
Steps to Run
1. Start the cluster
Bash
This spins up a single-node Kubernetes cluster inside a Docker container. First run downloads images and can take a few minutes.
2. Deploy the app
Bash
Creates a Deployment named nginx-deployment running 2 replica pods of nginx:latest.
3. Expose the app
Bash
Creates a NodePort Service so the app is reachable from outside the cluster, on port 30080.
4. Verify pods are running
Bash
Should show 2 pods with STATUS: Running.
5. Access the app
Bash
Copy the printed URL into a browser — you should see the nginx welcome page.
6. Scale the deployment
Bash
You should now see 4 pods instead of 2, demonstrating horizontal scaling.
7. Inspect a pod (logs & events)
Bash
describe shows scheduling events, container state, and any errors. logs shows the actual stdout/stderr from the container — useful for debugging a crashing app.
8. Clean up
Bash
What's in deployment.yaml
replicas: 2 — runs 2 identical pods for redundancy.
selector / labels — how the Deployment finds and manages its pods.
resources — CPU/memory requests and limits, good practice even in a local demo.
What's in service.yaml
type: NodePort — exposes the app on a fixed port (30080) on the Minikube node itself, reachable via minikube service or the node's IP.
selector: app: nginx-demo — routes traffic to any pod matching this label, regardless of how many replicas exist.
Execution Logs & Screenshots
See execution-logs.txt for expected output at each step — replace it with your own real terminal output once you run these commands.
Also take actual screenshots of:
kubectl get pods (showing 2, then 4 replicas after scaling)
kubectl get services
The nginx welcome page in a browser
kubectl describe pod ... output
Save them in a screenshots/ folder and reference them in this README, e.g.:
Markdown
Notes / Learnings
A Deployment manages a set of identical pods and handles restarts, rolling updates, and scaling automatically — you rarely create raw Pods directly in real projects.
A Service gives pods a stable network identity, since individual pod IPs change every time they're recreated.
kubectl scale changes the desired replica count; Kubernetes then creates or terminates pods to match that number.
kubectl describe is usually the first place to look when a pod is stuck in Pending or CrashLoopBackOff — the Events section at the bottom explains why.