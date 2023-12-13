# Kubernetes Certification (CKA, CKD and CKS) Study Guide
This repository was created for the propouse of the CKA, CKD and CKS Certification. All notes are made based for study of articles, documentation and courses taken by me.

Links on appropriate pages!

> **_NOTE:_**  This study guide is not finished!

# Certified Kubernetes Administrator (CKA)
Link: https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/

### Domains & Competencies
1. Application Design and Build `20%`
	* Define, build and modify container images
	* Choose and use the right workload resource (Deployment, DaemonSet, CronJob, etc.)
	* Understand multi-container Pod design patterns (e.g. sidecar, init and others)
	* Utilize persistent and ephemeral volumes

2. Application Deployment `20%`
	* Use Kubernetes primitives to implement common deployment strategies (e.g. blue/green or canary)
	* Understand Deployments and how to perform rolling updates
	* Use the Helm package manager to deploy existing packages
	* Kustomize

3. Application Observability and Maintenance `15%`
	* Understand API deprecations
	* Implement probes and health checks
	* Use built-in CLI tools to monitor Kubernetes applications
	* Utilize container logs
	* Debugging in Kubernetes

4. Application Environment, Configuration and Security `25%`
	* Discover and use resources that extend Kubernetes (CRD, Operators)
	* Understand authentication, authorization and admission control
	* Understand requests, limits, quotas
	* Understand ConfigMaps
	* Define resource requirements
	* Create & consume Secrets
	* Understand ServiceAccounts
	* Understand Application Security (SecurityContexts, Capabilities, etc.)

5.	Services and Networking `20%`
	* Demonstrate basic understanding of NetworkPolicies
	* Provide and troubleshoot access to applications via services
	* Use Ingress rules to expose applications


# Certified Kubernetes Application Developer (CKAD)
Link: https://training.linuxfoundation.org/certification/certified-kubernetes-application-developer-ckad/

### Domains & Competencies
1. Application Design and Build `20%`
	* Define, build and modify container images
	* Choose and use the right workload resource (Deployment, DaemonSet, CronJob, etc.)
	* Understand multi-container Pod design patterns (e.g. sidecar, init and others)
	* Utilize persistent and ephemeral volumes

2. Application Deployment `20%`
	* Use Kubernetes primitives to implement common deployment strategies (e.g. blue/green or canary)
	* Understand Deployments and how to perform rolling updates
	* Use the Helm package manager to deploy existing packages
	* Kustomize

3. Application Observability and Maintenance `15%`
	* Understand API deprecations
	* Implement probes and health checks
	* Use built-in CLI tools to monitor Kubernetes applications
	* Utilize container logs
	* Debugging in Kubernetes

4. Application Environment, Configuration and Security `25%`
	* Discover and use resources that extend Kubernetes (CRD, Operators)
	* Understand authentication, authorization and admission control
	* Understand requests, limits, quotas
	* Understand ConfigMaps
	* Define resource requirements
	* Create & consume Secrets
	* Understand ServiceAccounts
	* Understand Application Security (SecurityContexts, Capabilities, etc.)

5. Services and Networking `20%`
	* Demonstrate basic understanding of NetworkPolicies
	* Provide and troubleshoot access to applications via services
	* Use Ingress rules to expose applications


# Certified Kubernetes Security Specialist (CKS)
Link: https://training.linuxfoundation.org/certification/certified-kubernetes-security-specialist/

### Domains & Competencies
1. Cluster Setup `10%`
	* Use Network security policies to restrict cluster level access
	* Use CIS benchmark to review the security configuration of Kubernetes components (etcd, kubelet, kubedns, kubeapi)
	* Properly set up Ingress objects with security control
	* Protect node metadata and endpoints
	* Minimize use of, and access to, GUI elements
	* Verify platform binaries before deploying

2. Cluster Hardening `15%`
	* Restrict access to Kubernetes API
	* Use Role Based Access Controls to minimize exposure
	* Exercise caution in using service accounts e.g. disable defaults, minimize permissions on newly created ones
	* Update Kubernetes frequently

3. System Hardening `15%`
	* Minimize host OS footprint (reduce attack surface)
	* Minimize IAM roles
	* Minimize external access to the network
	* Appropriately use kernel hardening tools such as AppArmor, seccomp

4. Minimize Microservice Vulnerabilities `20%`
	* Setup appropriate OS level security domains
	* Manage Kubernetes secrets
	* Use container runtime sandboxes in multi-tenant environments (e.g. gvisor, kata containers)
	* Implement pod to pod encryption by use of mTLS

5. Supply Chain Security `20%`
	* Minimize base image footprint
	* Secure your supply chain: whitelist allowed registries, sign and validate images
	* Use static analysis of user workloads (e.g.Kubernetes resources, Docker files)
	* Scan images for known vulnerabilities

6. Monitoring, Logging and Runtime Security `20%`
	* Perform behavioral analytics of syscall process and file activities at the host and container level to detect malicious activities
	* Detect threats within physical infrastructure, apps, networks, data, users and workloads
	* Detect all phases of attack regardless where it occurs and how it spreads
	* Perform deep analytical investigation and identification of bad actors within environment
	* Ensure immutability of containers at runtime
	* Use Audit Logs to monitor access
