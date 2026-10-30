# Lab 2 — hello-app on Kubernetes

Your copy of this repository is your submission. The lab sheet is on the course site; this file is where you record what you did.

What's here:

| File | What it is |
|---|---|
| `src/`, `index.html` | a small React page that shows which Pod answered |
| `nginx.conf` | serves the page, and answers `/whoami` with the Pod name |
| `Dockerfile` | builds the page, then serves it with nginx on port 80 |

You don't change any of these. You add `hello-app.yaml` and fill in the sections below.

## Before you push, check

- [ ] `hello-app.yaml` is in the root of this repository
- [ ] `screenshot.png` is in the root of this repository
- [ ] every section below is filled in

---

## 1. My image

Docker Hub image I deployed (for example `amritanjal/hello-app:v1`):

```
GitHub Repository: `https://github.com/AmritanjalSingh/ms-cloud-devops-lab2`
Docker Hub Image: `amritanjal/hello-app:v1`
```

## 2. Pods across nodes

Output of `kubectl get pods -o wide` after you applied `hello-app.yaml`:

```
NAME                         READY   STATUS    RESTARTS   AGE   IP           NODE              NOMINATED NODE   READINESS GATES
hello-app-75d45bb648-ktql2   1/1     Running   0          8s    10.244.1.6   desktop-worker2              
hello-app-75d45bb648-lb4cf   1/1     Running   0          8s    10.244.2.7   desktop-worker               
hello-app-75d45bb648-qwdh4   1/1     Running   0          8s    10.244.2.8   desktop-worker
```

## 3. The Service and its endpoints

Output of `kubectl get service hello-app` and `kubectl get endpointslices -l kubernetes.io/service-name=hello-app`:

```
NAME        TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
hello-app   LoadBalancer   10.96.4.200   172.18.0.7    8080:31347/TCP   9s

NAME              ADDRESSTYPE   PORTS   ENDPOINTS                         AGE
hello-app-7cfwr   IPv4          80      10.244.1.6,10.244.2.8,10.244.2.7   9s
```

## 4. Browser

Add `screenshot.png`: the page at `http://localhost:8080` with at least two Pods in the "Answers so far" table.

## 5. The four traps

For each trap, write what you saw and why it happened, in two or three lines of your own words.

**Trap 1 — the local image.**
When deploying `hello-app:v1` without a Docker Hub username, cluster nodes normally fail with `ErrImagePull` or `ImagePullBackOff` because worker nodes have separate image storage from the laptop and try to pull `library/hello-app:v1` from Docker Hub, which does not exist. (On my Docker Desktop setup, the local `desktop-containerd-registry-mirror` cached the image, so pushing `amritanjal/hello-app:v1` to Docker Hub ensured all worker nodes could pull it from a real registry).

**Trap 2 — the selector typo.**
When I changed the Service selector to `app: helo-app`, the browser immediately showed an error and `kubectl get endpointslices` showed `<unset>` for both `PORTS` and `ENDPOINTS`. Because no Pods had the misspelled label `app: helo-app`, the Service could not find any Pods and had nowhere to send traffic.

**Trap 3 — the wrong targetPort.**
When I changed `targetPort` to `8080`, the browser failed again, even though `kubectl get endpointslices` still listed all my Pod IPs (`10.244.1.7,10.244.1.6,10.244.2.8`). The `PORTS` column changed to `8080`, meaning the Service sent traffic to port 8080 inside the Pods, where nothing was listening since nginx only listens on port 80.

**Trap 4 — the EXTERNAL-IP address.**
`kubectl get service hello-app` showed `172.18.0.7` as the `EXTERNAL-IP`, but opening `http://172.18.0.7:8080` in the browser never loaded. That IP address lives inside Docker Desktop's private internal network, which my Windows laptop cannot reach directly; instead, Docker Desktop publishes LoadBalancer ports on `http://localhost:8080`.

## 6. Your YAML vs the generated YAML

Name two fields in the `--dry-run=client -o yaml` output that were not in your file. Why does your file still work without them?
Two extra fields in the generated YAML are `strategy: {}` (or `resources: {}` / `creationTimestamp: null`) and `status: {}`. My file still works without them because Kubernetes automatically fills in default values for optional configuration fields (like `RollingUpdate` for `strategy`), sets metadata like `creationTimestamp` on creation, and dynamically populates `status` at runtime.
