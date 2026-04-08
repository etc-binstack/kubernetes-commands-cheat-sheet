# Kubernetes Commands Cheat Sheet

![Kubernetes](https://img.shields.io/badge/Kubernetes-1.28+-326CE5?style=flat&logo=kubernetes&logoColor=white)
![Platform](https://img.shields.io/badge/Platform-Self--Managed%20EC2-FF9900?style=flat&logo=amazon-aws&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=flat)
![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat)

## 📦 Kubernetes Commands Cheat Sheet (Production Ready)

A **real-world Kubernetes commands cheat sheet** for DevOps engineers covering:

- 📦 Daily `kubectl` commands
- 🔍 Kubernetes troubleshooting guide (CrashLoopBackOff, DiskPressure, etc.)
- 🌐 Services, Networking, and Storage (PV/PVC)
- 🔐 RBAC, ConfigMaps, Secrets
- ⚙️ Production debugging workflows

💡 Built from managing a **self-hosted Kubernetes cluster on AWS EC2**

⭐ If this helps you, consider starring the repo!

---

## 🔥 Quick Access

- Kubernetes Commands Cheat Sheet
- kubectl Debugging Commands
- Kubernetes Troubleshooting Guide
- DevOps Production Runbooks

---

## ⚡ Quick Start (POC Setup)

Want to spin up your own Kubernetes cluster in minutes?

👉 [Kubernetes POC Setup Guide (v1.29)](./k8s-poc-setup.md)

- Single-node & multi-node support  
- containerd (modern) + Docker (optional)  
- Includes MetalLB LoadBalancer setup  
- Works on local VM & cloud (AWS EC2)

---

## 📌 Who is this for?

- DevOps Engineers
- SREs
- Backend Engineers working with Kubernetes
- Anyone managing production K8s clusters

---

> A production-grade reference for day-to-day Kubernetes operations — covering core commands, storage management, and real-world troubleshooting cookbooks. Built from running a self-managed single-node K8s cluster on AWS EC2.

---

## Table of Contents

- [Placeholders](#-placeholders)
- [1. Cluster & Nodes](#1-cluster--nodes)
- [2. Namespaces](#2-namespaces)
- [3. Pods](#3-pods)
- [4. Deployments & Rollouts](#4-deployments--rollouts)
- [5. Services & Networking](#5-services--networking)
- [6. Storage — PV & PVC](#6-storage--pv--pvc)
- [7. Secrets & ConfigMaps](#7-secrets--configmaps)
- [8. Resource Inspection & Usage](#8-resource-inspection--usage)
- [9. Events & Logs](#9-events--logs)
- [10. JSONPath & Custom Output](#10-jsonpath--custom-output)
- [11. Cleanup & Force Actions](#11-cleanup--force-actions)
- [12. Misc & Power Commands](#12-misc--power-commands)
- [13. Labels & Annotations](#13-labels--annotations)
- [14. RBAC & Access Control](#14-rbac--access-control)
- [15. Contexts & Kubeconfig](#15-contexts--kubeconfig)
- [16. Scheduling — Taints, Tolerations & Affinity](#16-scheduling--taints-tolerations--affinity)
- [17. Autoscaling](#17-autoscaling)
- [18. Jobs & CronJobs](#18-jobs--cronjobs)
- [19. StatefulSets & DaemonSets](#19-statefulsets--daemonsets)
- [20. NetworkPolicy](#20-networkpolicy)
- [21. Probes — Debugging Liveness & Readiness](#21-probes--debugging-liveness--readiness)
- [22. Cluster Admin & Certificates](#22-cluster-admin--certificates)
- [Troubleshooting Cookbooks](#-troubleshooting-cookbooks)
  - [Disk Pressure & Eviction Recovery](#-disk-pressure--eviction-recovery)
  - [PV/PVC Binding Issues](#-pvpvc-binding-issues)
  - [Pod CrashLoopBackOff](#-pod-crashloopbackoff)
  - [ImagePullBackOff / ErrImagePull](#-imagepullbackoff--errimagepull)
  - [Pod Stuck in Pending](#-pod-stuck-in-pending)
  - [Service Not Reachable](#-service-not-reachable)
- [Node-Level Operations](#️-node-level-operations)
  - [containerd & crictl](#-containerd--crictl)
  - [Kubelet Config](#️-kubelet-config)
  - [EC2 Stop/Start Cycle Issues](#️-ec2-stopstart-cycle-issues)
  - [EBS Volume Auto-Expand](#-ebs-volume-auto-expand)

---

## 📋 Placeholders

> Replace these tags with your actual values before running any command.

| Placeholder | Meaning | Category |
|-------------|---------|----------|
| `<node-name>` | Kubernetes node name (get from `kubectl get nodes`) | Cluster & Nodes |
| `<namespace>` | Kubernetes namespace (e.g. `default`, `nova`, `kube-system`) | All |
| `<pod-name>` | Full pod name including hash (get from `kubectl get pods`) | Pods |
| `<deployment-name>` | Deployment name (e.g. `nova-auth-svc`) | Deployments |
| `<service-name>` | Kubernetes Service name | Services & Networking |
| `<secret-name>` | Secret resource name | Secrets & ConfigMaps |
| `<configmap-name>` | ConfigMap resource name | Secrets & ConfigMaps |
| `<container-name>` | Container name inside a pod (matches `name:` in Deployment spec) | Pods |
| `<image-name>` | Container image name or partial string to grep | Deployments |
| `<local-port>` | Port on your local machine for port-forwarding | Services & Networking |
| `<target-port>` | Port the service/pod listens on inside the cluster | Services & Networking |
| `<key>` | Key name inside a Secret or ConfigMap | Secrets & ConfigMaps |
| `<revision>` | Rollout revision number (from `rollout history`) | Deployments & Rollouts |
| `<pv-name>` | PersistentVolume name | Storage — PV & PVC |
| `<pvc-name>` | PersistentVolumeClaim name | Storage — PV & PVC |
| `<role-name>` | Role or ClusterRole name | RBAC |
| `<sa-name>` | ServiceAccount name | RBAC |
| `<user>` | Kubernetes user or service account for RBAC checks | RBAC |
| `<context-name>` | kubeconfig context name (e.g. `prod`, `dev`) | Contexts & Kubeconfig |
| `<label-key>` | Label key (e.g. `app`, `env`, `tier`) | Labels & Annotations |
| `<label-value>` | Label value (e.g. `nova-auth-svc`, `prod`) | Labels & Annotations |
| `<job-name>` | Job or CronJob name | Jobs & CronJobs |
| `<hpa-name>` | HorizontalPodAutoscaler name | Autoscaling |
| `<taint-key>` | Taint key applied to a node (e.g. `dedicated`, `gpu`) | Scheduling |

---

## 1. Cluster & Nodes

> The entry point for any cluster investigation. Before debugging a pod or service,
> always check node health first — a node under DiskPressure or MemoryPressure will
> silently evict pods and block scheduling.

```bash
# List all nodes and their status
kubectl get nodes

# Show detailed node info — OS, kernel, kubelet version, resource capacity
kubectl describe node <node-name>

# Show allocated CPU/memory on a node vs actual capacity
kubectl describe node <node-name> | grep -A 5 "Allocated resources"

# Check all node conditions — spot MemoryPressure / DiskPressure / PIDPressure
kubectl describe node <node-name> | grep -A 10 "Conditions:"

# Node conditions summary across all nodes (one-liner)
kubectl get nodes -o custom-columns=NAME:.metadata.name,STATUS:.status.conditions[-1].type

# Real-time CPU and memory usage per node (requires metrics-server)
kubectl top nodes

# Show full node spec as YAML
kubectl get node <node-name> -o yaml

# Cordon a node — mark unschedulable (no new pods placed here)
kubectl cordon <node-name>

# Uncordon a node — re-enable scheduling
kubectl uncordon <node-name>

# Drain a node safely — evicts all pods before maintenance
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

---

## 2. Namespaces

> Namespaces are soft isolation boundaries — they scope names, RBAC, and resource quotas.
> Most `kubectl` commands are namespace-scoped; forgetting `-n` silently queries `default`
> and makes you think resources don't exist.

```bash
# List all namespaces
kubectl get namespaces

# List all resources inside a namespace
kubectl get all -n <namespace>

# Set a default namespace for current context (avoids repeating -n every command)
kubectl config set-context --current --namespace=<namespace>

# Show current context and active namespace
kubectl config view --minify | grep namespace

# Create a namespace
kubectl create namespace <namespace>

# Delete a namespace and all its resources — irreversible
kubectl delete namespace <namespace>
```

---

## 3. Pods

> Pods are the smallest deployable unit. You rarely create them directly — Deployments manage them.
> But you debug them directly: logs, exec, describe. The most-used section in day-to-day operations.

```bash
# List all pods in a namespace with status
kubectl get pods -n <namespace>

# List pods with node placement and IP info
kubectl get pods -n <namespace> -o wide

# Watch pod list update in real time
kubectl get pods -n <namespace> -w

# List pods across all namespaces
kubectl get pods -A

# Describe a pod — events, conditions, resource limits, failure reason
kubectl describe pod <pod-name> -n <namespace>

# Extract failure reason from a pod (OOM, eviction, error message)
kubectl describe pod <pod-name> -n <namespace> | grep -A 10 "Message\|Reason\|Conditions"

# Get current container logs
kubectl logs <pod-name> -n <namespace>

# Get previous container logs — use when pod has already crashed
kubectl logs <pod-name> -n <namespace> --previous

# Stream logs in real time
kubectl logs <pod-name> -n <namespace> -f

# Logs from a specific container inside a multi-container pod
kubectl logs <pod-name> -n <namespace> -c <container-name>

# Exec into a running pod (interactive shell)
kubectl exec -it <pod-name> -n <namespace> -- /bin/sh

# Check environment variables injected into a pod
kubectl exec <pod-name> -n <namespace> -- printenv

# Get full pod spec as live YAML (reflects actual running state)
kubectl get pod <pod-name> -n <namespace> -o yaml

# Show pods sorted by restart count — quickly spot crash-looping pods
kubectl get pods -n <namespace> --sort-by='.status.containerStatuses[0].restartCount'

# List all failed / evicted pods in a namespace
kubectl get pods -n <namespace> --field-selector=status.phase=Failed

# List pods by label selector
kubectl get pods -n <namespace> -l app=<deployment-name>
```

---

## 4. Deployments & Rollouts

> Deployments manage the desired state of your application — replicas, image version, update strategy.
> Rollout commands are your undo button. Always check `rollout history` before a rollback
> so you know which revision is actually stable.

```bash
# List all deployments in a namespace
kubectl get deployments -n <namespace>

# Describe a deployment — replica counts, strategy, conditions
kubectl describe deployment <deployment-name> -n <namespace>

# Watch rollout progress — waits until complete or fails
kubectl rollout status deployment/<deployment-name> -n <namespace>

# View rollout history with revision numbers
kubectl rollout history deployment/<deployment-name> -n <namespace>

# Rollback to the previous version
kubectl rollout undo deployment/<deployment-name> -n <namespace>

# Rollback to a specific revision
kubectl rollout undo deployment/<deployment-name> -n <namespace> --to-revision=<revision>

# Force restart all pods in a deployment (no image change — useful to pick up new Secrets/ConfigMaps)
kubectl rollout restart deployment/<deployment-name> -n <namespace>

# Update deployment image directly (used by CI/CD pipelines)
kubectl set image deployment/<deployment-name> \
  <container-name>=<image-name>:<tag> -n <namespace>

# Scale a deployment up or down
kubectl scale deployment/<deployment-name> --replicas=3 -n <namespace>

# Show resource requests and limits per container across all pods
kubectl get pods -n <namespace> \
  -o=jsonpath="{range .items[*]}{.metadata.name}{'\t'}{.spec.containers[*].resources}{'\n'}{end}"
```

---

## 5. Services & Networking

> Services give pods a stable DNS name and load-balance traffic across replicas.
> The most common mistake: a service with no endpoints — which means the selector labels
> on the Service don't match the labels on the pods. Always check endpoints first.

```bash
# List all services in a namespace
kubectl get svc -n <namespace>

# Describe a service — port mappings, selector, endpoints
kubectl describe svc <service-name> -n <namespace>

# Show endpoints (actual pod IPs) backing a service — empty means no pods match the selector
kubectl get endpoints <service-name> -n <namespace>

# Port-forward a service to your local machine (useful for local testing without LoadBalancer)
kubectl port-forward svc/<service-name> -n <namespace> <local-port>:<target-port>

# Port-forward directly to a pod (bypasses the service entirely)
kubectl port-forward pod/<pod-name> -n <namespace> <local-port>:<target-port>

# Test service DNS resolution from inside the cluster
kubectl exec -it <pod-name> -n <namespace> -- nslookup <service-name>

# Call another service from inside a pod
kubectl exec -it <pod-name> -n <namespace> -- curl http://<service-name>:<target-port>

# List all IngressRoutes (Traefik CRD)
kubectl get ingressroute -n <namespace>

# Describe an IngressRoute
kubectl describe ingressroute <name> -n <namespace>

# List standard Ingress resources
kubectl get ingress -n <namespace>

# Describe ingress rules
kubectl describe ingress -n <namespace>

# List Traefik middlewares
kubectl get middleware -n <namespace>
```

---

## 6. Storage — PV & PVC

> PersistentVolumes (PV) are the actual storage. PersistentVolumeClaims (PVC) are requests for storage.
> Binding is automatic but can go wrong — especially after a PVC is deleted and the PV enters
> `Released` state. Always pin PVs with `claimRef` in production to prevent cross-binding.

```bash
# List all PersistentVolumes (cluster-scoped — no namespace needed)
kubectl get pv

# List all PersistentVolumeClaims in a namespace
kubectl get pvc -n <namespace>

# Show PV with capacity, access mode, reclaim policy, and status
kubectl get pv -o wide

# Show PVC with bound PV name and storage class
kubectl get pvc -n <namespace> -o wide

# Describe a PV — shows claimRef, node affinity, events
kubectl describe pv <pv-name>

# Describe a PVC — shows bound PV, access mode, events
kubectl describe pvc <pvc-name> -n <namespace>

# Get PV as YAML — useful to inspect or edit claimRef
kubectl get pv <pv-name> -o yaml

# Get PVC as YAML
kubectl get pvc <pvc-name> -n <namespace> -o yaml

# List StorageClasses
kubectl get storageclass

# Pin a PV to a specific PVC (prevents cross-binding) — set claimRef on the PV
kubectl patch pv <pv-name> -p '{
  "spec": {
    "claimRef": {
      "name": "<pvc-name>",
      "namespace": "<namespace>"
    }
  }
}'

# Release a PV — clear its claimRef so it returns to Available (use after PVC is deleted)
kubectl patch pv <pv-name> -p '{"spec":{"claimRef": null}}'

# Apply a PV/PVC manifest
kubectl apply -f pv-pvc.yaml

# Delete a PVC (only when no pod is mounting it)
kubectl delete pvc <pvc-name> -n <namespace>

# Delete a PV
kubectl delete pv <pv-name>

# Check which pods are using a PVC
kubectl get pods -n <namespace> -o json \
  | jq -r '.items[] | select(.spec.volumes[].persistentVolumeClaim.claimName=="<pvc-name>") | .metadata.name'
```

---

## 7. Secrets & ConfigMaps

> ConfigMaps hold non-sensitive config. Secrets hold sensitive values — but base64 is encoding,
> not encryption. Anyone with `kubectl get secret` access can read them.
> After updating either, pods don't automatically reload — use `rollout restart` to pick up changes.

```bash
# Decode a specific secret value (base64-decoded)
kubectl get secret <secret-name> -n <namespace> \
  -o jsonpath="{.data.<key>}" | base64 --decode

# View full secret YAML — values are base64-encoded, not encrypted
kubectl get secret <secret-name> -n <namespace> -o yaml

# List all secrets in a namespace
kubectl get secrets -n <namespace>

# Create a generic secret from literal values
kubectl create secret generic <secret-name> \
  --from-literal=<key>=<value> -n <namespace>

# View ConfigMap
kubectl get configmap <configmap-name> -n <namespace> -o yaml

# List all ConfigMaps in a namespace
kubectl get configmaps -n <namespace>

# Edit a ConfigMap in-place
kubectl edit configmap <configmap-name> -n <namespace>
```

---

## 8. Resource Inspection & Usage

> `kubectl top` shows live usage. `kubectl describe` shows requested/limited values.
> These are different things — a pod can request 100m CPU, use 900m, and not be throttled
> until it hits its limit. Understanding both is key to right-sizing your workloads.

```bash
# Real-time CPU and memory per pod (requires metrics-server)
kubectl top pods -n <namespace>

# Top pods sorted by CPU across all namespaces
kubectl top pods -A --sort-by=cpu

# Top pods sorted by memory across all namespaces
kubectl top pods -A --sort-by=memory

# Real-time usage per node
kubectl top nodes

# Show all resource types and their API groups
kubectl api-resources

# Explain any resource field — built-in docs without leaving the terminal
kubectl explain pod.spec.containers
kubectl explain pv.spec.persistentVolumeReclaimPolicy

# List all supported API versions on this cluster
kubectl api-versions

# Get any resource as YAML
kubectl get <resource-type> <name> -n <namespace> -o yaml

# Get any resource as JSON
kubectl get <resource-type> <name> -n <namespace> -o json
```

---

## 9. Events & Logs

> Events are the first place to look when something breaks — they capture scheduling failures,
> image pull errors, probe failures, and OOM kills. They expire after ~1 hour by default,
> so check them immediately when an incident occurs.

```bash
# Show all events in a namespace sorted by time (most recent last)
kubectl get events -n <namespace> --sort-by=.metadata.creationTimestamp

# Watch events in real time
kubectl get events -n <namespace> -w

# Show only Warning events — filter out normal noise
kubectl get events -n <namespace> --field-selector type=Warning

# Show events for a specific pod or object
kubectl get events -n <namespace> --field-selector involvedObject.name=<pod-name>

# Follow logs from all pods matching a label (e.g. a Deployment)
kubectl logs -n <namespace> -l app=<deployment-name> -f

# Get last N lines of logs
kubectl logs <pod-name> -n <namespace> --tail=100

# Get logs since a time window
kubectl logs <pod-name> -n <namespace> --since=1h

# Search logs for a keyword
kubectl logs <pod-name> -n <namespace> | grep -i "error\|exception\|failed"

# Pipe logs and filter with grep — highlight matches
kubectl logs <pod-name> -n <namespace> -f | grep --line-buffered -i "error\|warn"
```

---

## 10. JSONPath & Custom Output

> Default `kubectl get` output hides most fields. JSONPath and custom-columns let you extract
> exactly what you need — useful for scripting, CI/CD pipelines, and quick cross-resource comparisons
> without writing a full script.

```bash
# Extract all pod names in a namespace
kubectl get pods -n <namespace> -o jsonpath="{.items[*].metadata.name}"

# Get the node a specific pod is running on
kubectl get pod <pod-name> -n <namespace> -o jsonpath="{.spec.nodeName}"

# Get the image of a container inside a pod
kubectl get pod <pod-name> -n <namespace> \
  -o jsonpath="{.spec.containers[?(@.name=='<container-name>')].image}"

# List all PVs with name, capacity, status and claim
kubectl get pv -o custom-columns=\
NAME:.metadata.name,\
CAPACITY:.spec.capacity.storage,\
STATUS:.status.phase,\
CLAIM:.spec.claimRef.name

# Custom columns — pods with node placement and status
kubectl get pods -n <namespace> \
  -o custom-columns=NAME:.metadata.name,NODE:.spec.nodeName,STATUS:.status.phase

# List images running across all pods in a namespace
kubectl get pods -n <namespace> \
  -o jsonpath="{range .items[*]}{.metadata.name}{'\t'}{.spec.containers[*].image}{'\n'}{end}"

# Get all pod IPs in a namespace
kubectl get pods -n <namespace> \
  -o jsonpath="{range .items[*]}{.metadata.name}{'\t'}{.status.podIP}{'\n'}{end}"
```

---

## 11. Cleanup & Force Actions

> Use with care — most of these are irreversible. Force-deleting a pod bypasses graceful shutdown,
> which can cause data loss or split-brain if the pod holds a lock or open connection.
> Always prefer graceful deletion; use `--force` only when a pod is genuinely stuck.

```bash
# Force delete a stuck pod immediately (skips graceful shutdown)
kubectl delete pod <pod-name> -n <namespace> --grace-period=0 --force

# Delete all failed / evicted pods in a namespace
kubectl get pods -n <namespace> --field-selector=status.phase=Failed -o name \
  | xargs kubectl delete -n <namespace>

# Delete all completed (Succeeded) pods in a namespace
kubectl delete pod -n <namespace> --field-selector=status.phase==Succeeded

# Delete all resources (pods, services, deployments) in a namespace — use with care
kubectl delete all --all -n <namespace>

# Delete a specific resource
kubectl delete <resource-type> <name> -n <namespace>

# Remove a finalizer from a stuck resource (e.g. PVC stuck in Terminating)
kubectl patch pvc <pvc-name> -n <namespace> \
  -p '{"metadata":{"finalizers":[]}}' --type=merge
```

---

## 12. Misc & Power Commands

> `--dry-run=client` and `kubectl diff` are underused but invaluable — validate before you apply.
> The ephemeral debug pod (`kubectl run tmp-shell`) is your Swiss Army knife for in-cluster
> network debugging when your app containers don't have curl or nslookup installed.

```bash
# Dry run — validate a manifest without applying it to the cluster
kubectl apply -f <file>.yaml --dry-run=client

# Diff — show what would change if you applied a file (requires server-side dry-run)
kubectl diff -f <file>.yaml

# Apply a manifest
kubectl apply -f <file>.yaml

# Apply all manifests in a directory
kubectl apply -f <directory>/

# Apply with Kustomize overlay
kubectl apply -k k8s_deployment/overlays/production/

# Launch an ephemeral debug container on a node (useful for network/filesystem debugging)
kubectl debug node/<node-name> -it --image=busybox

# Run a temporary pod and drop into a shell — auto-deleted on exit
kubectl run tmp-shell --rm -i --tty --image=busybox -n <namespace> -- sh

# Copy a file from a pod to local machine
kubectl cp <namespace>/<pod-name>:/path/to/file ./local-file

# Copy a file from local machine into a pod
kubectl cp ./local-file <namespace>/<pod-name>:/path/to/file

# Open a port-forward to the Traefik dashboard
kubectl port-forward -n <namespace> svc/traefik 9000:9000
```

---

## 🔧 Troubleshooting Cookbooks

> Step-by-step runbooks for the most common real-world failure patterns.
> Each starts with symptoms and ends with prevention.

---

### 💽 Disk Pressure & Eviction Recovery

**Symptoms:** Pods show `Evicted` status. Node condition `DiskPressure=True`. New pods won't schedule.

#### Step 1 — Identify the pressure type
```bash
# Which condition is True: MemoryPressure / DiskPressure / PIDPressure
kubectl describe node <node-name> | grep -A 8 "Conditions:"

# Full resource allocation view
kubectl describe node <node-name> | grep -A 10 "Allocated resources"
```

#### Step 2 — Check disk usage on the node (SSH in)
```bash
df -h                             # overall disk usage by partition
du -sh /var/lib/containerd        # container image store — usually the largest
du -sh /var/log                   # system + pod logs
du -sh /var/lib/kubelet           # kubelet state and volumes
du -sh /tmp                       # temp files
crictl images                     # list all cached container images with sizes
```

#### Step 3 — Free up disk space (SSH in)
```bash
# Remove all images not used by any running container (safe)
crictl rmi --prune

# Find large pod log files over 100MB
find /var/log/pods -name "*.log" -size +100M

# Check systemd journal disk usage
journalctl --disk-usage

# Truncate journal logs to 500MB
sudo journalctl --vacuum-size=500M

# Verify space recovered
df -h
```

#### Step 4 — Clean up evicted pod records
```bash
# Kubelet flips DiskPressure to False within ~30s once space is freed
kubectl describe node <node-name> | grep DiskPressure

# Evicted pods don't auto-delete — clean up the dead records
kubectl get pods -n <namespace> --field-selector=status.phase=Failed -o name \
  | xargs kubectl delete -n <namespace>
```

#### Prevention — tune kubelet GC thresholds
```bash
# Edit /var/lib/kubelet/config.yaml on the node:
#
#   imageGCHighThresholdPercent: 70   # trigger GC at 70% full (default: 85)
#   imageGCLowThresholdPercent: 60    # stop GC at 60% (default: 80)
#   evictionHard:
#     nodefs.available: "10%"
#     nodefs.inodesFree: "5%"
#     imagefs.available: "10%"
#
sudo systemctl daemon-reload && sudo systemctl restart kubelet
kubectl describe node <node-name> | grep DiskPressure
```

---

### 📦 PV/PVC Binding Issues

**Symptoms:** PVC stuck in `Pending`. PV stuck in `Released` (old claim deleted but PV not reused). Wrong PV binds to wrong PVC (cross-binding). PVC stuck in `Terminating`.

#### Inspect current state
```bash
# Check PV statuses — look for Released, Pending, Bound
kubectl get pv

# Check PVC statuses — look for Pending or Lost
kubectl get pvc -n <namespace>

# See claimRef on a Released PV (who it was bound to before)
kubectl describe pv <pv-name> | grep -A 5 "Claim:"

# Check why a PVC is Pending — look at events
kubectl describe pvc <pvc-name> -n <namespace> | grep -A 10 "Events:"
```

#### Fix: PV is Released — clear claimRef to make it Available again
```bash
# A Released PV won't rebind until its claimRef is cleared
kubectl patch pv <pv-name> -p '{"spec":{"claimRef": null}}'

# Verify it moved to Available
kubectl get pv <pv-name>
```

#### Fix: Pin PV to a specific PVC (prevent cross-binding)
```bash
# Set claimRef on the PV before creating the PVC
# This makes the PV bind ONLY to the named PVC
kubectl patch pv <pv-name> -p '{
  "spec": {
    "claimRef": {
      "name": "<pvc-name>",
      "namespace": "<namespace>"
    }
  }
}'

# Then apply PVC — Kubernetes matches by claimRef, cross-binding can't happen
kubectl apply -f pv-pvc.yaml
```

#### Full clean-bind sequence (when PVCs are gone and PVs are Released)
```bash
# 1. Clear stale claimRef from each Released PV
kubectl patch pv <pv-name-1> -p '{"spec":{"claimRef": null}}'
kubectl patch pv <pv-name-2> -p '{"spec":{"claimRef": null}}'

# 2. Verify both are Available
kubectl get pv

# 3. Apply the manifest (creates PVCs + sets claimRef pinning on PVs)
kubectl apply -f pv-pvc.yaml

# 4. Verify correct binding
kubectl get pv && kubectl get pvc -n <namespace>
```

#### Fix: PVC stuck in Terminating (finalizer blocking deletion)
```bash
# Check for finalizers
kubectl get pvc <pvc-name> -n <namespace> -o jsonpath="{.metadata.finalizers}"

# Remove finalizers to force deletion
kubectl patch pvc <pvc-name> -n <namespace> \
  -p '{"metadata":{"finalizers":[]}}' --type=merge
```

#### Prevention — always pin PVs with claimRef in your YAML
```yaml
# In your PV spec — add this to prevent any PVC from grabbing the wrong PV
spec:
  claimRef:
    name: my-specific-pvc
    namespace: my-namespace
```

---

### 🔁 Pod CrashLoopBackOff

**Symptoms:** Pod status shows `CrashLoopBackOff`. Restart count keeps climbing.

```bash
# Step 1 — check the restart count and last exit code
kubectl get pods -n <namespace> --sort-by='.status.containerStatuses[0].restartCount'

# Step 2 — get the crash reason
kubectl describe pod <pod-name> -n <namespace> | grep -A 10 "Last State\|Reason\|Exit Code"

# Step 3 — read previous container logs (the crashed instance)
kubectl logs <pod-name> -n <namespace> --previous

# Step 4 — check if the issue is a missing env var or bad config
kubectl exec <pod-name> -n <namespace> -- printenv | grep -i "db\|url\|host\|pass"

# Step 5 — check events for OOMKilled or config errors
kubectl get events -n <namespace> --field-selector involvedObject.name=<pod-name> \
  --sort-by=.metadata.creationTimestamp
```

**Common causes:**

| Exit Code | Meaning |
|-----------|---------|
| `0` | Process exited cleanly — check app startup logic |
| `1` | Application error — check logs |
| `137` | OOMKilled — pod exceeded memory limit |
| `139` | Segfault in the container process |
| `143` | SIGTERM not handled — container didn't shut down gracefully |

---

### 🖼️ ImagePullBackOff / ErrImagePull

**Symptoms:** Pod status shows `ImagePullBackOff` or `ErrImagePull`. Pod never starts.

```bash
# Step 1 — see which image it's trying to pull
kubectl describe pod <pod-name> -n <namespace> | grep -A 5 "Image:"

# Step 2 — check pull error details
kubectl describe pod <pod-name> -n <namespace> | grep -A 10 "Events:"

# Step 3 — verify imagePullSecret is attached to the deployment
kubectl get deployment <deployment-name> -n <namespace> -o yaml \
  | grep -A 3 "imagePullSecrets"

# Step 4 — verify the secret exists and has the right keys
kubectl get secret <secret-name> -n <namespace> -o yaml \
  | grep -E "\.dockerconfigjson|type:"

# Step 5 — list secrets of type kubernetes.io/dockerconfigjson
kubectl get secrets -n <namespace> \
  | grep kubernetes.io/dockerconfigjson
```

**Common causes:** wrong image tag, ECR auth token expired, `imagePullSecrets` missing from the deployment spec, or the service account doesn't have the secret attached.

---

### ⏳ Pod Stuck in Pending

**Symptoms:** Pod status stays `Pending` indefinitely. Never transitions to `Running`.

```bash
# Step 1 — find the reason from events
kubectl describe pod <pod-name> -n <namespace> | grep -A 15 "Events:"

# Step 2 — check if it's a scheduling failure
kubectl describe pod <pod-name> -n <namespace> | grep -A 5 "Warning\|Insufficient\|Unschedulable"

# Step 3 — check node has enough resources
kubectl describe node <node-name> | grep -A 5 "Allocated resources"
kubectl top nodes

# Step 4 — check if PVC is unbound (storage not ready)
kubectl get pvc -n <namespace>
kubectl describe pvc <pvc-name> -n <namespace>

# Step 5 — check node selector or tolerations mismatch
kubectl get pod <pod-name> -n <namespace> -o yaml \
  | grep -A 5 "nodeSelector\|tolerations\|affinity"
```

**Common causes:** insufficient CPU/memory on the node, PVC stuck in Pending, node selector mismatch, taint without matching toleration.

---

### 🌐 Service Not Reachable

**Symptoms:** `curl` or application call fails between pods. DNS resolves but connection refused or times out.

```bash
# Step 1 — check if endpoints exist (no endpoints = selector mismatch)
kubectl get endpoints <service-name> -n <namespace>

# Step 2 — verify selector labels match pod labels
kubectl describe svc <service-name> -n <namespace> | grep Selector
kubectl get pods -n <namespace> --show-labels | grep <deployment-name>

# Step 3 — test DNS from inside the cluster
kubectl exec -it <pod-name> -n <namespace> -- nslookup <service-name>

# Step 4 — test TCP connectivity from inside the cluster
kubectl exec -it <pod-name> -n <namespace> -- curl -v http://<service-name>:<target-port>

# Step 5 — check if pod is actually listening on the right port
kubectl exec -it <pod-name> -n <namespace> -- netstat -tlnp
# or if netstat not available:
kubectl exec -it <pod-name> -n <namespace> -- ss -tlnp

# Step 6 — port-forward directly to pod to bypass service (isolate the issue)
kubectl port-forward pod/<pod-name> -n <namespace> <local-port>:<target-port>
curl http://localhost:<local-port>/health
```

---

## ⚙️ Node-Level Operations

### 🐳 containerd & crictl

> `crictl` is the CLI for containerd. Use this on Kubernetes nodes instead of `docker`.
> EKS ≥ 1.24 and kubeadm clusters use containerd by default.

#### One-time setup
```bash
# Fix endpoint warnings (run once per node)
crictl config --set runtime-endpoint=unix:///run/containerd/containerd.sock
crictl config --set image-endpoint=unix:///run/containerd/containerd.sock
```

#### Image management
```bash
# List all cached images with sizes
crictl images

# Remove images not used by any running container (safe)
crictl rmi --prune

# Remove a specific image by ID
crictl rmi <image-id>
```

#### Container management
```bash
# List all containers including stopped/dead ones
crictl ps -a

# Count total containers (running + stopped)
crictl ps -a | wc -l

# Remove all stopped/exited/dead containers
crictl rm --all --force

# Check how much disk space containerd is using
du -sh /var/lib/containerd/
du -sh /var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/
```

#### Full cleanup sequence (run on node via SSH)
```bash
# 1. Remove stopped containers
crictl rm --all --force 2>/dev/null

# 2. Remove unused images
crictl rmi --prune

# 3. Clean pod logs older than 3 days
find /var/log/pods -mtime +3 -name "*.log" -delete

# 4. Shrink systemd journal
journalctl --vacuum-size=200M

# 5. Verify space freed
df -h /
```

#### Restart runtime in correct order
```bash
# Fixes stale stats that cause false DiskPressure after a restart
sudo systemctl restart containerd
sleep 10
sudo systemctl restart kubelet
sleep 15
kubectl describe node <node-name> | grep DiskPressure | head -2
```

---

### ⚙️ Kubelet Config

> Config file location: `/var/lib/kubelet/config.yaml` on the node.
> After any edit: `sudo systemctl daemon-reload && sudo systemctl restart kubelet`

#### Recommended config for a dev single-node cluster
```yaml
containerRuntimeEndpoint: "unix:///run/containerd/containerd.sock"

# Image GC — start cleanup at 80%, stop at 70%
imageGCHighThresholdPercent: 80
imageGCLowThresholdPercent: 70

# Hard eviction — kill pods when these thresholds are breached
evictionHard:
  memory.available: "500Mi"
  nodefs.available: "5%"
  nodefs.inodesFree: "5%"
  imagefs.available: "5%"

# Soft eviction — warn and give grace period before killing
evictionSoft:
  memory.available: "1Gi"
  nodefs.available: "10%"
evictionSoftGracePeriod:
  memory.available: "2m"
  nodefs.available: "2m"

# Prevents flip-flopping between pressure/no-pressure on startup
evictionPressureTransitionPeriod: 5m

# Give pods time to terminate cleanly when instance stops
shutdownGracePeriod: 30s
shutdownGracePeriodCriticalPods: 10s
```

#### Verify kubelet picked up config changes
```bash
sudo systemctl status kubelet | grep -i "active\|loaded"
sudo journalctl -u kubelet --since "5 minutes ago" | grep -i "evict\|disk\|imagefs"
```

---

### 🖥️ EC2 Stop/Start Cycle Issues

> Dev clusters are often stopped overnight. These issues occur on every restart if not fixed.

#### Problem 1 — IP address changes break Kubernetes
EC2 instances get a new private IP on every start unless you assign a static one.
- kubelet registers with the old IP
- API server certificates are bound to the old IP
- etcd cluster state has stale node info

**Fix:** AWS Console → EC2 → Network Interface → assign a fixed private IP.

#### Problem 2 — Services start in wrong order
```bash
# Check if kubelet depends on containerd
sudo systemctl cat kubelet | grep -i "after\|require"

# Add the dependency if missing
sudo systemctl edit kubelet
```
Add under `[Unit]`:
```ini
[Unit]
After=containerd.service
Requires=containerd.service
```

#### Problem 3 — containerd stats stale after restart (causes false DiskPressure)
```bash
# Quick manual fix — run after each EC2 start
sudo crictl rmi --prune
sudo systemctl restart containerd && sleep 5 && sudo systemctl restart kubelet
```

#### Automated fix — systemd startup service
```bash
# Create the cleanup script
sudo tee /usr/local/bin/k8s-startup.sh > /dev/null <<'EOF'
#!/bin/bash
sleep 10
crictl rmi --prune 2>/dev/null || true
systemctl restart containerd
sleep 5
systemctl restart kubelet
echo "$(date): k8s startup complete" >> /var/log/k8s-startup.log
EOF
sudo chmod +x /usr/local/bin/k8s-startup.sh

# Create the systemd service
sudo tee /etc/systemd/system/k8s-startup.service > /dev/null <<'EOF'
[Unit]
Description=K8s startup cleanup
After=network-online.target containerd.service
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/k8s-startup.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable k8s-startup.service
```

---

### 📈 EBS Volume Auto-Expand

> Automatically expands the EBS volume when disk usage exceeds a threshold.
> Requires the EC2 instance to have an IAM role with EBS permissions.

#### Step 1 — IAM policy (attach to node's IAM role)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeVolumes",
        "ec2:DescribeVolumesModifications",
        "ec2:ModifyVolume"
      ],
      "Resource": "*"
    }
  ]
}
```

#### Step 2 — Install dependencies
```bash
sudo apt-get install -y awscli cloud-guest-utils
```

#### Step 3 — Create the auto-expand script
```bash
sudo tee /usr/local/bin/ebs-auto-expand.sh > /dev/null <<'EOF'
#!/bin/bash

THRESHOLD=75          # expand when disk usage exceeds this %
EXPAND_BY_GB=20       # how many GB to add each time
MAX_SIZE_GB=200       # safety cap — never expand beyond this

USED=$(df / | awk 'NR==2 {print $5}' | tr -d '%')

if [ "$USED" -lt "$THRESHOLD" ]; then
  exit 0
fi

TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
INSTANCE_ID=$(curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id)
REGION=$(curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/placement/region)

VOLUME_ID=$(aws ec2 describe-volumes \
  --region "$REGION" \
  --filters "Name=attachment.instance-id,Values=$INSTANCE_ID" \
            "Name=attachment.device,Values=/dev/sda1,/dev/xvda,/dev/nvme0n1" \
  --query 'Volumes[0].VolumeId' --output text)

CURRENT_SIZE=$(aws ec2 describe-volumes \
  --region "$REGION" \
  --volume-ids "$VOLUME_ID" \
  --query 'Volumes[0].Size' --output text)

NEW_SIZE=$((CURRENT_SIZE + EXPAND_BY_GB))

if [ "$NEW_SIZE" -gt "$MAX_SIZE_GB" ]; then
  echo "$(date): Disk at ${USED}% but at max cap (${MAX_SIZE_GB}GB). Manual action needed." \
    >> /var/log/ebs-auto-expand.log
  exit 1
fi

echo "$(date): Disk at ${USED}%. Expanding $VOLUME_ID: ${CURRENT_SIZE}GB → ${NEW_SIZE}GB" \
  >> /var/log/ebs-auto-expand.log

aws ec2 modify-volume --region "$REGION" --volume-id "$VOLUME_ID" --size "$NEW_SIZE"

while true; do
  STATE=$(aws ec2 describe-volumes-modifications \
    --region "$REGION" \
    --volume-ids "$VOLUME_ID" \
    --query 'VolumesModifications[0].ModificationState' --output text)
  [ "$STATE" = "completed" ] && break
  sleep 10
done

DEVICE=$(lsblk -no PKNAME $(df / | awk 'NR==2{print $1}'))
PART_NUM=$(lsblk -no NAME $(df / | awk 'NR==2{print $1}') | grep -o '[0-9]*$')

growpart /dev/$DEVICE $PART_NUM
resize2fs $(df / | awk 'NR==2{print $1}')

echo "$(date): Expansion complete. New size: ${NEW_SIZE}GB" >> /var/log/ebs-auto-expand.log
EOF
sudo chmod +x /usr/local/bin/ebs-auto-expand.sh
```

#### Step 4 — Add cron job (checks every 5 minutes)
```bash
(sudo crontab -l 2>/dev/null; echo "*/5 * * * * /usr/local/bin/ebs-auto-expand.sh 2>> /var/log/ebs-auto-expand.log") \
  | sudo crontab -
```

#### Step 5 — Test and monitor
```bash
# Test manually
sudo /usr/local/bin/ebs-auto-expand.sh

# Watch the log
tail -f /var/log/ebs-auto-expand.log
```

#### Manual EBS expand + filesystem resize (AWS Console method)
```bash
# After expanding in AWS Console (EC2 → Volumes → Modify Volume):
lsblk                          # confirm new size is visible
sudo growpart /dev/nvme0n1 1   # resize partition (adjust device name if needed)
sudo resize2fs /dev/nvme0n1p1  # resize filesystem
df -h /                        # verify
```

---

---

## 13. Labels & Annotations

> Labels are the glue of Kubernetes — selectors, scheduling, and routing all depend on them.
> Annotations carry metadata that tools (Prometheus, Traefik, ArgoCD) read.

```bash
# Show all labels on pods
kubectl get pods -n <namespace> --show-labels

# Filter pods by label
kubectl get pods -n <namespace> -l <label-key>=<label-value>

# Filter with multiple labels (AND logic)
kubectl get pods -n <namespace> -l app=<deployment-name>,env=prod

# Add a label to a pod (live — doesn't persist after pod restart)
kubectl label pod <pod-name> -n <namespace> <label-key>=<label-value>

# Overwrite an existing label
kubectl label pod <pod-name> -n <namespace> <label-key>=<label-value> --overwrite

# Remove a label from a pod
kubectl label pod <pod-name> -n <namespace> <label-key>-

# Add a label to a node (used for node selectors and affinity)
kubectl label node <node-name> <label-key>=<label-value>

# Check labels on nodes
kubectl get nodes --show-labels

# Add an annotation (metadata for tools — not used by K8s scheduler)
kubectl annotate pod <pod-name> -n <namespace> <key>=<value>

# View annotations on a resource
kubectl get pod <pod-name> -n <namespace> -o jsonpath="{.metadata.annotations}"
```

---

## 14. RBAC & Access Control

> RBAC controls who can do what in the cluster. Day-to-day you mostly read and debug — rarely create from scratch.

```bash
# Check if the current user/service account can perform an action
kubectl auth can-i get pods -n <namespace>
kubectl auth can-i delete deployments -n <namespace>

# Check permissions for a specific service account
kubectl auth can-i get secrets -n <namespace> \
  --as=system:serviceaccount:<namespace>:<sa-name>

# List all permissions for the current user
kubectl auth can-i --list -n <namespace>

# List all service accounts in a namespace
kubectl get serviceaccounts -n <namespace>

# Describe a service account (shows mounted secrets)
kubectl describe serviceaccount <sa-name> -n <namespace>

# List all roles in a namespace
kubectl get roles -n <namespace>

# List all cluster-wide roles
kubectl get clusterroles | grep -v "^system:"

# List role bindings in a namespace
kubectl get rolebindings -n <namespace>

# Describe a role binding — see who is bound to which role
kubectl describe rolebinding <role-name> -n <namespace>

# List cluster role bindings
kubectl get clusterrolebindings | grep -v "^system:"
```

---

## 15. Contexts & Kubeconfig

> If you manage multiple clusters (dev, staging, prod), contexts let you switch without editing files.

```bash
# Show all contexts
kubectl config get-contexts

# Show the currently active context
kubectl config current-context

# Switch to a different context (switches cluster + user + namespace)
kubectl config use-context <context-name>

# View full kubeconfig
kubectl config view

# View merged kubeconfig from all sources
kubectl config view --merge --flatten

# Set a namespace as default for the current context
kubectl config set-context --current --namespace=<namespace>

# Create a new context entry
kubectl config set-context <context-name> \
  --cluster=<cluster-name> --user=<user> --namespace=<namespace>

# Delete a context
kubectl config delete-context <context-name>

# Use a specific kubeconfig file (useful for CI/CD pipelines)
KUBECONFIG=/path/to/kubeconfig kubectl get pods -n <namespace>

# Merge two kubeconfig files
KUBECONFIG=~/.kube/config:/path/to/other-config kubectl config view --merge --flatten \
  > ~/.kube/merged-config
```

---

## 16. Scheduling — Taints, Tolerations & Affinity

> Taints repel pods from nodes. Tolerations allow specific pods to land on tainted nodes.
> Affinity gives you fine-grained control over which pods go where.

```bash
# Add a taint to a node (NoSchedule = new pods without toleration won't schedule here)
kubectl taint node <node-name> <taint-key>=<value>:NoSchedule

# Add a taint with NoExecute (also evicts existing pods that don't tolerate it)
kubectl taint node <node-name> <taint-key>=<value>:NoExecute

# Remove a taint from a node (note the trailing minus)
kubectl taint node <node-name> <taint-key>:NoSchedule-

# View taints on all nodes
kubectl describe nodes | grep -A 3 "Taints:"

# Check why a pod was not scheduled on a node
kubectl describe pod <pod-name> -n <namespace> | grep -A 10 "Events:\|Warning\|didn't match"

# View node affinity on a pod
kubectl get pod <pod-name> -n <namespace> -o jsonpath="{.spec.affinity}"

# List PodDisruptionBudgets (controls minimum availability during voluntary disruptions)
kubectl get pdb -n <namespace>

# Describe a PDB — see min available, allowed disruptions
kubectl describe pdb -n <namespace>
```

**Toleration snippet (add to pod spec to allow scheduling on a tainted node):**
```yaml
tolerations:
  - key: "<taint-key>"
    operator: "Equal"
    value: "<value>"
    effect: "NoSchedule"
```

---

## 17. Autoscaling

> HPA scales pod replicas based on CPU/memory. Requires metrics-server running in the cluster.

```bash
# List all HPAs in a namespace — shows current/desired replicas and CPU%
kubectl get hpa -n <namespace>

# Watch HPA in real time (useful during load tests)
kubectl get hpa -n <namespace> -w

# Describe an HPA — see scaling events and thresholds
kubectl describe hpa <hpa-name> -n <namespace>

# Create an HPA for a deployment from CLI (quick way to test autoscaling)
kubectl autoscale deployment <deployment-name> -n <namespace> \
  --min=1 --max=5 --cpu-percent=70

# Delete an HPA (reverts to static replica count on the deployment)
kubectl delete hpa <hpa-name> -n <namespace>

# Check if metrics-server is running (HPA won't work without it)
kubectl get pods -n kube-system | grep metrics-server

# Get current pod CPU/memory — same data HPA uses to decide scaling
kubectl top pods -n <namespace>
```

**Why it matters:** If HPA shows `<unknown>/70%` for CPU, metrics-server is not running or the deployment has no resource `requests` set — HPA can't calculate a ratio without a baseline.

---

## 18. Jobs & CronJobs

> Jobs run a task to completion. CronJobs schedule Jobs on a time expression.
> Useful for migrations, data exports, cleanup tasks.

```bash
# List all Jobs in a namespace
kubectl get jobs -n <namespace>

# List all CronJobs in a namespace
kubectl get cronjobs -n <namespace>

# Describe a Job — see completion count, active pods, conditions
kubectl describe job <job-name> -n <namespace>

# Check logs of a Job's pod
kubectl logs -n <namespace> -l job-name=<job-name>

# Manually trigger a CronJob immediately (creates a one-off Job from it)
kubectl create job <job-name>-manual --from=cronjob/<job-name> -n <namespace>

# Suspend a CronJob (stops new Jobs from being created)
kubectl patch cronjob <job-name> -n <namespace> -p '{"spec":{"suspend": true}}'

# Resume a suspended CronJob
kubectl patch cronjob <job-name> -n <namespace> -p '{"spec":{"suspend": false}}'

# Delete a completed Job and its pods
kubectl delete job <job-name> -n <namespace>

# View the last schedule time and next schedule of a CronJob
kubectl get cronjob <job-name> -n <namespace> \
  -o custom-columns=NAME:.metadata.name,SCHEDULE:.spec.schedule,LAST:.status.lastScheduleTime,SUSPEND:.spec.suspend
```

---

## 19. StatefulSets & DaemonSets

> StatefulSets give pods stable identity and ordered startup/shutdown — used for databases, message queues.
> DaemonSets ensure one pod runs on every node — used for log collectors, monitoring agents.

```bash
# List StatefulSets
kubectl get statefulsets -n <namespace>

# Describe a StatefulSet — see update strategy, replicas, volume claim templates
kubectl describe statefulset <deployment-name> -n <namespace>

# Watch StatefulSet pods come up in order (they start 0, 1, 2... sequentially)
kubectl get pods -n <namespace> -l app=<deployment-name> -w

# Rollout restart a StatefulSet (respects ordered rollout — one pod at a time)
kubectl rollout restart statefulset/<deployment-name> -n <namespace>

# Watch rollout status
kubectl rollout status statefulset/<deployment-name> -n <namespace>

# Scale a StatefulSet (scale-down terminates highest-ordinal pod first)
kubectl scale statefulset/<deployment-name> --replicas=3 -n <namespace>

# List DaemonSets
kubectl get daemonsets -n <namespace>

# Describe a DaemonSet — shows desired/current/ready counts per node
kubectl describe daemonset <deployment-name> -n <namespace>

# Check which nodes a DaemonSet pod is running on
kubectl get pods -n <namespace> -l app=<deployment-name> -o wide

# Rollout restart a DaemonSet
kubectl rollout restart daemonset/<deployment-name> -n <namespace>
```

---

## 20. NetworkPolicy

> NetworkPolicy is a firewall for pods — controls which pods can talk to which.
> By default, all pods can communicate freely. A NetworkPolicy restricts that.

```bash
# List all NetworkPolicies in a namespace
kubectl get networkpolicy -n <namespace>

# Describe a NetworkPolicy — see ingress/egress rules, pod selectors
kubectl describe networkpolicy -n <namespace>

# Check if a namespace has any NetworkPolicies (empty = all traffic allowed)
kubectl get networkpolicy -n <namespace> --no-headers | wc -l

# Get a NetworkPolicy as YAML to understand its rules
kubectl get networkpolicy <name> -n <namespace> -o yaml
```

**Key concept — default deny all (apply this first, then open only what's needed):**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: <namespace>
spec:
  podSelector: {}     # matches ALL pods in the namespace
  policyTypes:
    - Ingress
    - Egress
```

**Troubleshooting tip:** If a service suddenly stops responding after adding a NetworkPolicy, describe it and check that your source pod's labels match the `from.podSelector` — a single wrong label blocks everything silently.

---

## 21. Probes — Debugging Liveness & Readiness

> **Liveness:** Is the container alive? Fail → restart the container.
> **Readiness:** Is the container ready to serve traffic? Fail → remove from Service endpoints.
> **Startup:** Is the app done initialising? Prevents liveness from killing a slow-starting app.

```bash
# See probe config on a running pod
kubectl get pod <pod-name> -n <namespace> -o yaml \
  | grep -A 15 "livenessProbe\|readinessProbe\|startupProbe"

# See probe failure events — tells you which probe failed and what it got back
kubectl describe pod <pod-name> -n <namespace> | grep -A 10 "Liveness\|Readiness\|Startup\|Warning"

# Check if pod is being removed from Service endpoints due to readiness failure
kubectl get endpoints <service-name> -n <namespace>

# Test the probe endpoint manually from inside the pod
kubectl exec <pod-name> -n <namespace> -- curl -s http://localhost:<target-port>/health

# Watch pod restart count — increments on each liveness probe eviction
kubectl get pods -n <namespace> -w

# Get last termination reason — "OOMKilled" or "Error" after probe-triggered restart
kubectl describe pod <pod-name> -n <namespace> | grep -A 5 "Last State:"
```

**Common causes of probe failure:**

| Symptom | Likely Cause |
|---------|-------------|
| Pod restarts every few minutes | Liveness probe timing too aggressive (increase `failureThreshold` or `periodSeconds`) |
| Pod never gets traffic despite Running | Readiness probe failing — check endpoint, wrong port, or app not ready |
| Pod killed during slow startup | Startup probe missing — app needs more time than liveness allows |
| `connection refused` on probe | App not listening on the port defined in the probe |

---

## 22. Cluster Admin & Certificates

> These are infrequent but critical — upgrade paths, certificate rotation, etcd health.
> On self-managed clusters (kubeadm) you own all of this.

```bash
# Check kubeadm upgrade plan — shows available versions and what will change
kubeadm upgrade plan

# Apply the upgrade (control plane components only — kubelet upgraded separately)
kubeadm upgrade apply v1.29.0

# Check certificate expiry — certs expire after 1 year by default on kubeadm clusters
kubeadm certs check-expiration

# Renew all certificates (do before they expire — cluster breaks if they do)
kubeadm certs renew all

# Check CSR (CertificateSigningRequest) status — used for kubelet cert rotation
kubectl get csr

# Approve a pending CSR
kubectl certificate approve <csr-name>

# Deny a CSR
kubectl certificate deny <csr-name>

# Check etcd health (run on control plane node)
ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  endpoint health

# Take an etcd snapshot (backup before upgrades or major changes)
ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify the snapshot
ETCDCTL_API=3 etcdctl snapshot status /tmp/etcd-backup.db
```

---

*Built from running a self-managed Kubernetes cluster on AWS EC2. Contributions welcome — open a PR.*
