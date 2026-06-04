Yes. For **Docker + Kubernetes**, the equivalent of what you’re asking for should not be “top 50 questions.” You want a stack that covers **container internals, image building, Docker security, Kubernetes architecture, workloads, networking, storage, scheduling, observability, troubleshooting, GitOps, operators, and production failure modes**.

Here is the **no-slop list**.

***

# Best “Peyman-style” Docker + Kubernetes resources

## 1. **Containers and Kubernetes — Interview Preparation**

This is probably the closest match to what you want. It is explicitly designed for **senior software engineering / platform / SRE / infrastructure interviews**, and it covers Docker internals, Linux kernel primitives, image construction, Kubernetes workloads, networking, storage, operations, Operators, security, and GitOps. [\[github.com\]](https://github.com/BrendanJamesLynskey/Interview_Containers_Kubernetes)

Why it is good:

*   It covers **namespaces, cgroups, union filesystems, OCI spec, image layers, runC, and containerd**, which are the internals interviewers expect when Docker questions go beyond commands. [\[github.com\]](https://github.com/BrendanJamesLynskey/Interview_Containers_Kubernetes), [\[docs.docker.com\]](https://docs.docker.com/engine/security/)
*   It includes Kubernetes core topics like **Pods, Deployments, StatefulSets, DaemonSets, Jobs, CronJobs, Services, Ingress, CNI, PersistentVolumes, StorageClasses, ConfigMaps, Secrets, and CSI drivers**. [\[github.com\]](https://github.com/BrendanJamesLynskey/Interview_Containers_Kubernetes), [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/)
*   It also includes day-2 operational areas such as **scheduling, scaling, observability, debugging, operators, security, and GitOps**, which are what differentiate senior candidates from “I know kubectl” candidates. [\[github.com\]](https://github.com/BrendanJamesLynskey/Interview_Containers_Kubernetes), [\[livingdevops.com\]](https://livingdevops.com/kubernetes/advanced-kubernetes-interview-questions-the-complete-guide-to-production-troubleshooting-architecture-and-design-patterns/)

Use it for:

*   Docker internals.
*   Kubernetes architecture.
*   Platform/SRE interviews.
*   Troubleshooting questions.
*   Operators and GitOps.
*   Security and production best practices.

**Verdict:** start here.

***

## 2. **Kubesimplify Interview Prep**

This is a curated interview-prep repository for **Kubernetes, DevOps, and SRE roles**. It focuses on real-world scenarios, hands-on troubleshooting, system design, production incidents, CI/CD debugging, and architectural decision-making. [\[github.com\]](https://github.com/kubesimplify/interview-prep)

Why it is good:

*   It explicitly avoids standard shallow Q\&A and focuses on **production incidents, war stories, broken clusters, CI/CD pipelines, and system design**. [\[github.com\]](https://github.com/kubesimplify/interview-prep)
*   It is useful for interviews where the question is not “what is a Pod?” but “a deployment is failing in production; how do you debug it?” [\[github.com\]](https://github.com/kubesimplify/interview-prep), [\[livingdevops.com\]](https://livingdevops.com/kubernetes/advanced-kubernetes-interview-questions-the-complete-guide-to-production-troubleshooting-architecture-and-design-patterns/)

Use it for:

*   DevOps/SRE interview scenarios.
*   Kubernetes troubleshooting.
*   CI/CD pipeline debugging.
*   Cluster operations.
*   System-design style discussions.

**Verdict:** use this after the first repo to drill real-world scenarios.

***

## 3. **Advanced Kubernetes Interview Questions — Production Troubleshooting, Architecture, and Design Patterns**

This is a strong production-focused Kubernetes interview guide. It covers pod debugging, StatefulSets, persistent storage, cluster scaling, autoscaling, network policies, security, multi-tenancy, node troubleshooting, QoS, zero-downtime deployments, service mesh, operators, CRDs, logging, etcd performance, image policies, multi-region deployments, and ingress scaling. [\[livingdevops.com\]](https://livingdevops.com/kubernetes/advanced-kubernetes-interview-questions-the-complete-guide-to-production-troubleshooting-architecture-and-design-patterns/)

Why it is good:

*   It frames answers as **systematic debugging approaches**, not memorized commands. For example, for `CrashLoopBackOff`, it emphasizes pod lifecycle, events, `kubectl describe`, conditions, events, and last state rather than only checking logs. [\[livingdevops.com\]](https://livingdevops.com/kubernetes/advanced-kubernetes-interview-questions-the-complete-guide-to-production-troubleshooting-architecture-and-design-patterns/)
*   It covers the areas interviewers actually probe at senior level: **architecture, failure isolation, HA, performance, and operational tradeoffs**. [\[livingdevops.com\]](https://livingdevops.com/kubernetes/advanced-kubernetes-interview-questions-the-complete-guide-to-production-troubleshooting-architecture-and-design-patterns/), [\[livingdevops.com\]](https://livingdevops.com/kubernetes/15-advanced-kubernetes-interview-questions-answers-2025/)

Use it for:

*   Scenario-based Kubernetes interviews.
*   Production debugging.
*   Stateful workloads.
*   Autoscaling.
*   Networking/security.
*   Multi-region and high-availability design.

**Verdict:** very good for senior DevOps/SRE/Kubernetes rounds.

***

## 4. **15 Advanced Kubernetes Interview Questions with Detailed Answers**

This guide is narrower but useful for edge cases. It covers advanced Kubernetes behavior such as init container failures, restart policies, StatefulSet pod identity, persistent names, ordered deployment, storage behavior, and tricky lifecycle scenarios. [\[livingdevops.com\]](https://livingdevops.com/kubernetes/15-advanced-kubernetes-interview-questions-answers-2025/)

Why it is useful:

*   It explains subtle Kubernetes semantics, for example that StatefulSet Pods keep ordinal identities like `myapp-0`, `myapp-1`, and deleted Pods are recreated with the same name rather than renaming other replicas. [\[livingdevops.com\]](https://livingdevops.com/kubernetes/15-advanced-kubernetes-interview-questions-answers-2025/)
*   It is good for interviews where the interviewer tests whether you understand Kubernetes behavior under edge cases rather than only resource definitions. [\[livingdevops.com\]](https://livingdevops.com/kubernetes/15-advanced-kubernetes-interview-questions-answers-2025/), [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/)

Use it for:

*   StatefulSet edge cases.
*   Init container behavior.
*   Pod lifecycle.
*   Restart policies.
*   Tricky Kubernetes semantics.

**Verdict:** good as a second-pass edge-case guide.

***

# Source-of-truth resources

## 5. **Official Kubernetes Concepts Documentation**

This is mandatory. Kubernetes docs define the official model: cluster architecture, containers, workloads, Pods, Services, networking, storage, configuration, security, policies, scheduling, cluster administration, and extension mechanisms. [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/), [\[kubernetes.io\]](https://kubernetes.io/docs/home/)

Why it matters:

*   Kubernetes is a declarative orchestration system, and the official docs are the source of truth for the abstractions interviewers expect: **Pods, workloads, services, storage, security, policies, scheduling, and extensions**. [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/), [\[kubernetes.io\]](https://kubernetes.io/docs/home/)
*   If you read only interview blogs, you will miss exact semantics around controllers, reconciliation, scheduling, object lifecycle, and API behavior. [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/), [\[k8s.info\]](https://k8s.info/docs/foundations/architecture)

Use it for:

*   Correct terminology.
*   Architecture.
*   Object model.
*   Workloads.
*   Services and networking.
*   Storage.
*   Security.
*   Scheduling.
*   Extending Kubernetes.

**Verdict:** source of truth.

***

## 6. **Kubernetes Visual Handbook — Architecture**

This is useful because it explains Kubernetes architecture visually and with the correct mental model: the control plane as the “brain,” worker nodes as the “muscle,” etcd as source of truth, API server as central hub, and controllers as reconciliation loops. [\[k8s.info\]](https://k8s.info/docs/foundations/architecture)

Why it is good:

*   It explains the core Kubernetes loop: observe current state, compare with desired state, and act to reconcile differences. [\[k8s.info\]](https://k8s.info/docs/foundations/architecture)
*   It walks through what happens when you run `kubectl apply`, which is a common interview-style architecture question. [\[k8s.info\]](https://k8s.info/docs/foundations/architecture), [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/)

Use it for:

*   Control plane internals.
*   API server.
*   etcd.
*   Scheduler.
*   Controller manager.
*   Kubelet.
*   kube-proxy.
*   Reconciliation mental model.

**Verdict:** excellent for architecture intuition.

***

## 7. **Official Docker Build Best Practices**

This is the Docker image-building source of truth. Docker’s docs recommend multi-stage builds, reusable stages, choosing the right base image, trusted images, small/minimal production images, and rebuilding images regularly to keep dependencies secure. [\[docs.docker.com\]](https://docs.docker.com/build/building/best-practices/)

Why it matters:

*   Docker images should be built with **multi-stage builds** to separate build dependencies from runtime artifacts and reduce final image size. [\[docs.docker.com\]](https://docs.docker.com/build/building/best-practices/)
*   Base image choice affects security and size; Docker recommends trusted, minimal images and separate build/test images from production runtime images. [\[docs.docker.com\]](https://docs.docker.com/build/building/best-practices/), [\[docs.docker.com\]](https://docs.docker.com/dhi/core-concepts/hardening/)
*   Docker images are immutable snapshots, so rebuilding regularly is important for updated dependencies and security patches. [\[docs.docker.com\]](https://docs.docker.com/build/building/best-practices/)

Use it for:

*   Dockerfile best practices.
*   Multi-stage builds.
*   Base image selection.
*   Build caching.
*   Production image optimization.
*   CI/CD image builds.

**Verdict:** source of truth for Dockerfile/image questions.

***

## 8. **Official Docker Engine Security**

Docker’s security docs explain the actual container security model: namespaces, cgroups, Docker daemon attack surface, container configuration loopholes, and kernel hardening features. [\[docs.docker.com\]](https://docs.docker.com/engine/security/)

Why it matters:

*   Docker uses **namespaces** for isolation, so processes inside one container cannot normally see or affect processes in other containers or the host. [\[docs.docker.com\]](https://docs.docker.com/engine/security/)
*   Docker uses **cgroups** for resource accounting and limiting, which helps prevent a single container from exhausting CPU, memory, or I/O resources. [\[docs.docker.com\]](https://docs.docker.com/engine/security/)
*   Docker security must consider kernel security, Docker daemon exposure, container profiles, and kernel hardening features. [\[docs.docker.com\]](https://docs.docker.com/engine/security/)

Use it for:

*   Namespaces.
*   Cgroups.
*   Docker daemon security.
*   Container isolation.
*   Resource limits.
*   Kernel hardening.
*   Container breakout risk.

**Verdict:** mandatory for deep Docker internals.

***

## 9. **OWASP Docker Security Cheat Sheet**

This is a strong practical security checklist. It emphasizes keeping host and Docker updated, not exposing the Docker daemon socket, not mounting `/var/run/docker.sock` into containers, running containers as non-root, and using user namespaces. [\[cheatsheet....owasp.org\]](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)

Why it matters:

*   Exposing the Docker socket is effectively giving root-level control over the host because the Docker API can create privileged containers and mount host paths. [\[cheatsheet....owasp.org\]](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)
*   Running as an unprivileged user is one of the main ways to reduce privilege-escalation risk in containers. [\[cheatsheet....owasp.org\]](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html), [\[docs.docker.com\]](https://docs.docker.com/dhi/core-concepts/hardening/)
*   Containers share the host kernel, so host kernel and Docker Engine patching is part of container security. [\[cheatsheet....owasp.org\]](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html), [\[docs.docker.com\]](https://docs.docker.com/engine/security/)

Use it for:

*   Docker security interviews.
*   Container hardening.
*   Non-root containers.
*   Docker socket risks.
*   Host-kernel risk.
*   Least privilege.

**Verdict:** use this for security rounds.

***

# Good but lower-priority resources

## 10. **Docker & Kubernetes Interview Questions Deep Dive — GitHub**

This has 25+ senior-level Docker and Kubernetes questions with explanations, examples, diagrams, best practices, and interview-ready answers. It covers Docker under the hood, namespaces, cgroups, union filesystems, image layers, Docker Engine, Dockerfile layers, and image/container differences. [\[github.com\]](https://github.com/AhmedAbdelbasetAli/full-stack-interview-qa/blob/main/deployment/docker%20%26%20k8s.md)

Good for:

*   Quick revision.
*   Docker internals.
*   Image vs container.
*   Dockerfile layers.
*   Kubernetes basics.

Limitation:

*   It is useful, but I would still trust official Docker/Kubernetes docs for exact semantics. [\[github.com\]](https://github.com/AhmedAbdelbasetAli/full-stack-interview-qa/blob/main/deployment/docker%20%26%20k8s.md), [\[docs.docker.com\]](https://docs.docker.com/engine/security/), [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/)

***

## 11. **Dataquest Docker Interview Questions**

This is a decent Docker-specific interview prep article. It covers beginner, intermediate, advanced, and scenario-based Docker questions, with emphasis on concepts rather than only command memorization. [\[dataquest.io\]](https://www.dataquest.io/blog/docker-interview-questions-and-answers/)

Good for:

*   Docker interview warm-up.
*   Image vs container.
*   Docker vs VM.
*   Dockerfile.
*   Compose.
*   Health checks.
*   Secrets.

Limitation:

*   Less deep than Docker official security/build docs and less internals-focused than the senior GitHub repo. [\[dataquest.io\]](https://www.dataquest.io/blog/docker-interview-questions-and-answers/), [\[docs.docker.com\]](https://docs.docker.com/build/building/best-practices/), [\[docs.docker.com\]](https://docs.docker.com/engine/security/)

***

## 12. **DataCamp Kubernetes Interview Questions**

This is a broad Kubernetes interview guide with fundamental, intermediate, advanced, and scenario-based questions. It emphasizes hands-on experience, troubleshooting skills, and real-world problem solving. [\[datacamp.com\]](https://www.datacamp.com/blog/kubernetes-interview-questions)

Good for:

*   Basic-to-intermediate Kubernetes revision.
*   Concept coverage.
*   Scenario questions.
*   Confidence building.

Limitation:

*   It is not as deep as the production troubleshooting guide or official Kubernetes docs. [\[datacamp.com\]](https://www.datacamp.com/blog/kubernetes-interview-questions), [\[livingdevops.com\]](https://livingdevops.com/kubernetes/advanced-kubernetes-interview-questions-the-complete-guide-to-production-troubleshooting-architecture-and-design-patterns/), [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/)

***

# My no-slop shortlist

If you want only the highest-signal resources:

1.  **Containers and Kubernetes — Interview Preparation** — best Docker + K8s senior interview repo. [\[github.com\]](https://github.com/BrendanJamesLynskey/Interview_Containers_Kubernetes)
2.  **Kubesimplify Interview Prep** — best scenario/SRE-style interview repo. [\[github.com\]](https://github.com/kubesimplify/interview-prep)
3.  **Advanced Kubernetes Interview Questions — Production Troubleshooting** — best production debugging guide. [\[livingdevops.com\]](https://livingdevops.com/kubernetes/advanced-kubernetes-interview-questions-the-complete-guide-to-production-troubleshooting-architecture-and-design-patterns/)
4.  **Official Kubernetes Concepts** — source of truth. [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/)
5.  **Kubernetes Visual Handbook — Architecture** — best architecture mental model. [\[k8s.info\]](https://k8s.info/docs/foundations/architecture)
6.  **Official Docker Build Best Practices** — source of truth for Dockerfiles/images. [\[docs.docker.com\]](https://docs.docker.com/build/building/best-practices/)
7.  **Official Docker Engine Security** — source of truth for Docker internals/security. [\[docs.docker.com\]](https://docs.docker.com/engine/security/)
8.  **OWASP Docker Security Cheat Sheet** — best practical Docker hardening checklist. [\[cheatsheet....owasp.org\]](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)
9.  **15 Advanced Kubernetes Interview Questions** — good edge-case drill. [\[livingdevops.com\]](https://livingdevops.com/kubernetes/15-advanced-kubernetes-interview-questions-answers-2025/)
10. **Docker & Kubernetes Interview Questions Deep Dive — GitHub** — good revision notes. [\[github.com\]](https://github.com/AhmedAbdelbasetAli/full-stack-interview-qa/blob/main/deployment/docker%20%26%20k8s.md)

***

# What to master for Docker interviews

## 1. Container internals

You should be able to explain that containers are isolated processes using Linux kernel primitives, especially **namespaces** for isolation and **cgroups** for resource accounting/limits. Docker’s security docs explicitly identify namespaces and cgroups as major parts of Docker’s security and isolation model. [\[docs.docker.com\]](https://docs.docker.com/engine/security/)

Master:

*   PID namespace.
*   Mount namespace.
*   Network namespace.
*   User namespace.
*   IPC namespace.
*   UTS namespace.
*   cgroups v1/v2.
*   OverlayFS / union filesystem.
*   Image layers.
*   Writable container layer.
*   OCI image/runtime specs.
*   containerd and runc.

Best resource: **Containers and Kubernetes — Interview Preparation** + **Docker Engine Security**. [\[github.com\]](https://github.com/BrendanJamesLynskey/Interview_Containers_Kubernetes), [\[docs.docker.com\]](https://docs.docker.com/engine/security/)

***

## 2. Docker image construction

You should know how Dockerfile layers work, why build cache matters, why `COPY package.json` before source code helps cache dependency installation, why multi-stage builds reduce final image size, and why production images should not contain compilers/build tools. Docker’s official docs recommend multi-stage builds, reusable stages, trusted/minimal base images, and regular image rebuilds for security. [\[docs.docker.com\]](https://docs.docker.com/build/building/best-practices/)

Master:

*   `FROM`
*   `RUN`
*   `COPY` vs `ADD`
*   `CMD` vs `ENTRYPOINT`
*   `ARG` vs `ENV`
*   `.dockerignore`
*   Layer caching.
*   Multi-stage builds.
*   BuildKit.
*   Distroless/slim/alpine tradeoffs.
*   Image scanning.
*   SBOM basics.
*   Reproducible builds.

Best resource: **Docker Build Best Practices**. [\[docs.docker.com\]](https://docs.docker.com/build/building/best-practices/)

***

## 3. Docker security

You should be able to explain why Docker daemon access is dangerous, why mounting `/var/run/docker.sock` into containers is usually a critical risk, why non-root containers matter, and why minimal/hardened images reduce attack surface. OWASP explicitly warns against exposing the Docker daemon socket and recommends unprivileged users; Docker Hardened Images docs explain that hardened base images remove shells, compilers, package managers, and debugging tools to reduce attack surface. [\[cheatsheet....owasp.org\]](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html), [\[docs.docker.com\]](https://docs.docker.com/dhi/core-concepts/hardening/)

Master:

*   Non-root user.
*   Rootless Docker.
*   Docker socket risk.
*   Capabilities: `--cap-drop`.
*   seccomp.
*   AppArmor/SELinux.
*   Read-only root filesystem.
*   Secrets.
*   Image signing.
*   Vulnerability scanning.
*   Base image hardening.
*   Supply-chain risk.

Best resources: **OWASP Docker Security Cheat Sheet** + **Docker Engine Security** + **Docker Hardened Images docs**. [\[cheatsheet....owasp.org\]](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html), [\[docs.docker.com\]](https://docs.docker.com/engine/security/), [\[docs.docker.com\]](https://docs.docker.com/dhi/core-concepts/hardening/)

***

# What to master for Kubernetes interviews

## 1. Kubernetes architecture

You must know the control plane and node components deeply. Kubernetes docs organize concepts around cluster architecture, workloads, services/networking, storage, configuration, security, policies, scheduling, cluster administration, and extensions. [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/)

Master:

*   API server.
*   etcd.
*   Scheduler.
*   Controller manager.
*   Cloud controller manager.
*   Kubelet.
*   kube-proxy.
*   Container runtime.
*   CNI plugin.
*   CSI driver.
*   Reconciliation loop.
*   Desired state vs actual state.

Best resources: **Official Kubernetes Concepts** + **Kubernetes Visual Handbook**. [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/), [\[k8s.info\]](https://k8s.info/docs/foundations/architecture)

***

## 2. Workloads

You need to know when to use each workload abstraction. Kubernetes docs classify Pods as the smallest deployable compute object and higher-level abstractions as workload controllers that help run applications. [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/)

Master:

*   Pod.
*   ReplicaSet.
*   Deployment.
*   StatefulSet.
*   DaemonSet.
*   Job.
*   CronJob.
*   Init containers.
*   Sidecars.
*   Probes: startup, readiness, liveness.
*   Pod disruption budgets.
*   Rolling updates and rollbacks.

Use the advanced interview guides for edge cases around init containers, restart policies, and StatefulSet identities. [\[livingdevops.com\]](https://livingdevops.com/kubernetes/15-advanced-kubernetes-interview-questions-answers-2025/), [\[livingdevops.com\]](https://livingdevops.com/kubernetes/advanced-kubernetes-interview-questions-the-complete-guide-to-production-troubleshooting-architecture-and-design-patterns/)

***

## 3. Networking

Kubernetes networking interviews get deep quickly. You should know Services, kube-proxy, CNI, DNS, Ingress, NetworkPolicies, service mesh basics, and debugging traffic flow. Kubernetes docs explicitly separate Services, Load Balancing, and Networking as a core concept area. [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/)

Master:

*   Pod IP model.
*   Service types: ClusterIP, NodePort, LoadBalancer, ExternalName.
*   kube-proxy iptables/IPVS.
*   CoreDNS.
*   Ingress and ingress controllers.
*   CNI plugins: Calico, Cilium, Flannel.
*   NetworkPolicy.
*   Service mesh basics.
*   mTLS.
*   East-west vs north-south traffic.
*   DNS debugging.

Use production interview guides for NetworkPolicy, ingress scaling, service mesh, and external connectivity scenarios. [\[livingdevops.com\]](https://livingdevops.com/kubernetes/advanced-kubernetes-interview-questions-the-complete-guide-to-production-troubleshooting-architecture-and-design-patterns/), [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/)

***

## 4. Storage

Kubernetes storage questions usually test PV/PVC/StorageClass behavior, dynamic provisioning, StatefulSets, CSI, access modes, reclaim policies, and backup/restore. Kubernetes docs list storage as a core concept area for providing both long-term and temporary storage to Pods. [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/)

Master:

*   Volume.
*   PersistentVolume.
*   PersistentVolumeClaim.
*   StorageClass.
*   Dynamic provisioning.
*   CSI.
*   Access modes.
*   Reclaim policies.
*   StatefulSet storage.
*   Volume expansion.
*   Backup and restore.
*   Data locality.

Use the advanced Kubernetes guides for StatefulSets and persistent storage scenarios. [\[livingdevops.com\]](https://livingdevops.com/kubernetes/advanced-kubernetes-interview-questions-the-complete-guide-to-production-troubleshooting-architecture-and-design-patterns/), [\[livingdevops.com\]](https://livingdevops.com/kubernetes/15-advanced-kubernetes-interview-questions-answers-2025/)

***

## 5. Scheduling and autoscaling

You should understand how Pods get assigned to nodes and how workloads scale. Kubernetes docs include scheduling, preemption, and eviction as a lower-level concept area. [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/)

Master:

*   Requests and limits.
*   QoS classes.
*   Node selectors.
*   Node affinity.
*   Pod affinity/anti-affinity.
*   Taints and tolerations.
*   Topology spread constraints.
*   HPA.
*   VPA.
*   Cluster Autoscaler.
*   Karpenter.
*   Eviction.
*   Preemption.
*   ResourceQuota.
*   LimitRange.

Use the production guide for resource management, QoS, autoscaling, and node troubleshooting. [\[livingdevops.com\]](https://livingdevops.com/kubernetes/advanced-kubernetes-interview-questions-the-complete-guide-to-production-troubleshooting-architecture-and-design-patterns/), [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/)

***

## 6. Security

Kubernetes security questions can cover RBAC, service accounts, secrets, network policies, admission controllers, Pod Security Standards, image policies, and supply-chain controls. Kubernetes docs include security and policies as major concept sections. [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/)

Master:

*   RBAC.
*   ServiceAccounts.
*   Secrets.
*   NetworkPolicy.
*   Pod Security Standards.
*   Admission controllers.
*   OPA/Gatekeeper/Kyverno.
*   ImagePullSecrets.
*   Image scanning.
*   Least privilege.
*   Runtime security.
*   Audit logs.
*   Multi-tenancy.

Use the advanced guide for network policies, image security, policies, and multi-tenant architecture. [\[livingdevops.com\]](https://livingdevops.com/kubernetes/advanced-kubernetes-interview-questions-the-complete-guide-to-production-troubleshooting-architecture-and-design-patterns/), [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/)

***

# 4-week prep plan

## Week 1 — Docker internals and image engineering

Read:

*   Containers and Kubernetes repo: container fundamentals. [\[github.com\]](https://github.com/BrendanJamesLynskey/Interview_Containers_Kubernetes)
*   Docker Engine Security. [\[docs.docker.com\]](https://docs.docker.com/engine/security/)
*   Docker Build Best Practices. [\[docs.docker.com\]](https://docs.docker.com/build/building/best-practices/)
*   OWASP Docker Security Cheat Sheet. [\[cheatsheet....owasp.org\]](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)

Build:

*   One bad Dockerfile and one optimized Dockerfile.
*   Multi-stage build for Java/Node/Python/Go.
*   Non-root image.
*   Healthcheck.
*   Compose file with secrets.
*   Scan image with a vulnerability scanner.

Be ready to answer:

*   Docker vs VM.
*   Image vs container.
*   What are namespaces and cgroups?
*   How do image layers work?
*   Why multi-stage builds?
*   CMD vs ENTRYPOINT.
*   Why not run as root?
*   Why is Docker socket dangerous?
*   How do you reduce image size and CVEs?

***

## Week 2 — Kubernetes core architecture and workloads

Read:

*   Kubernetes Concepts. [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/)
*   Kubernetes Visual Handbook architecture. [\[k8s.info\]](https://k8s.info/docs/foundations/architecture)
*   Containers and Kubernetes repo: Kubernetes core. [\[github.com\]](https://github.com/BrendanJamesLynskey/Interview_Containers_Kubernetes)

Build:

*   Deployment with rolling update.
*   StatefulSet with PVC.
*   DaemonSet for log agent.
*   Job and CronJob.
*   ConfigMap and Secret.
*   Probes: startup/readiness/liveness.

Be ready to answer:

*   What happens after `kubectl apply`?
*   What does the API server do?
*   Why is etcd critical?
*   How does reconciliation work?
*   Deployment vs StatefulSet.
*   DaemonSet use cases.
*   Readiness vs liveness.
*   Init container vs sidecar.

***

## Week 3 — Networking, storage, scheduling

Read:

*   Kubernetes official networking/storage/scheduling sections. [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/)
*   Advanced Kubernetes production guide. [\[livingdevops.com\]](https://livingdevops.com/kubernetes/advanced-kubernetes-interview-questions-the-complete-guide-to-production-troubleshooting-architecture-and-design-patterns/)

Build:

*   ClusterIP/NodePort/LoadBalancer examples.
*   Ingress with path routing.
*   NetworkPolicy deny-all then allow-specific.
*   PVC with StorageClass.
*   Pod affinity and taints/tolerations.
*   HPA with metrics-server.

Be ready to answer:

*   How does Service routing work?
*   How does DNS work in Kubernetes?
*   Ingress vs Service.
*   NetworkPolicy debugging.
*   PV vs PVC vs StorageClass.
*   Why StatefulSets need stable identity.
*   Requests vs limits.
*   QoS classes.
*   HPA vs VPA vs Cluster Autoscaler.

***

## Week 4 — Production troubleshooting, security, and system design

Read:

*   Advanced Kubernetes Interview Questions. [\[livingdevops.com\]](https://livingdevops.com/kubernetes/advanced-kubernetes-interview-questions-the-complete-guide-to-production-troubleshooting-architecture-and-design-patterns/)
*   15 Advanced Kubernetes Questions. [\[livingdevops.com\]](https://livingdevops.com/kubernetes/15-advanced-kubernetes-interview-questions-answers-2025/)
*   Kubesimplify interview-prep scenarios. [\[github.com\]](https://github.com/kubesimplify/interview-prep)

Practice:

*   Pod stuck in `Pending`.
*   Pod in `CrashLoopBackOff`.
*   `ImagePullBackOff`.
*   Readiness probe failing.
*   Service has no endpoints.
*   DNS failure.
*   PVC stuck in pending.
*   Node pressure eviction.
*   Rolling update stuck.
*   Ingress returns 502.
*   HPA not scaling.

Be ready to answer:

*   How do you debug `CrashLoopBackOff` with no logs?
*   How do you debug service connectivity?
*   How do you design multi-tenant Kubernetes?
*   How do you secure a cluster?
*   How do you design zero-downtime deployments?
*   How do you run stateful workloads safely?
*   How do you handle cluster upgrades?
*   How do you design observability for Kubernetes?

***

# Interview summary note

For Docker, interviewers test whether you understand that containers are **isolated Linux processes**, not lightweight VMs. You should be able to explain namespaces, cgroups, image layers, container runtime, Dockerfile caching, multi-stage builds, minimal base images, non-root execution, and Docker daemon/socket security. [\[docs.docker.com\]](https://docs.docker.com/engine/security/), [\[docs.docker.com\]](https://docs.docker.com/build/building/best-practices/), [\[cheatsheet....owasp.org\]](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)

For Kubernetes, interviewers test whether you understand the **declarative reconciliation model**. You should be able to explain the control plane, API server, etcd, scheduler, controllers, kubelet, Services, CNI, storage, scheduling, autoscaling, RBAC, NetworkPolicy, and production debugging flow. [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/), [\[k8s.info\]](https://k8s.info/docs/foundations/architecture), [\[livingdevops.com\]](https://livingdevops.com/kubernetes/advanced-kubernetes-interview-questions-the-complete-guide-to-production-troubleshooting-architecture-and-design-patterns/)

My recommended core stack:

```text
1. Interview_Containers_Kubernetes
2. Kubernetes official docs
3. Kubernetes Visual Handbook
4. Docker official build docs
5. Docker official security docs
6. OWASP Docker Security Cheat Sheet
7. Advanced Kubernetes production troubleshooting guide
8. Kubesimplify interview-prep
```

That stack gives you both **interview readiness** and **real production depth**.
