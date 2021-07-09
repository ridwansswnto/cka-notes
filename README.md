<p align="center">
  <a href="https://kubernetes.io/" title="Redirect to Kubernetes page">
    <img src="https://github.com/kubernetes/kubernetes/raw/master/logo/logo.png" width="250" />
  </a>
</p>

# CKA Notes 
This is notes from my journey to get CKA

## CKA Curriculum

### Cluster Architecture, Installation, and Configuration - 25%
* [Manage role based access control (RBAC)](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
* Use Kubeadm to install a basic cluster
* Manage a highly-available Kubernetes cluster
* Provision underlying infrastructure to deploy a Kubernetes cluster
* Perform a version upgrade on a Kubernetes cluster using Kubeadm
* Implement etcd backup and restore

### Workloads & Scheduling - 15%
* [Understand deployments and how to perform rolling update and rollbacks](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* Use ConfigMaps and Secrets to configure applications
  * [ConfigMaps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)
  * [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
  * [Environment Variables](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/)
* Know how to scale applications
* Understand the primitives used to create robust, self-healing, application deployments
* Understand how resource limits can affect Pod scheduling
* Awareness of manifest management and common templating tools

### Services & Networking - 20%
* Understand host networking configuration on the cluster nodes
* Understand connectivity between Pods
* Understand ClusterIP, NodePort, LoadBalancer service types and endpoints
* Know how to use Ingress controllers and Ingress resources
* Know how to configure and use CoreDNS
* Choose an appropriate container network interface plugin

### Storage - 10%
* Understand storage classes, persistent volumes
* Understand volume mode, access modes and reclaim policies for volumes
* Understand persistent volume claims primitive
* Know how to configure applications with persistent storage

### Troubleshooting - 30%
* Evaluate cluster and node logging
* Understand how to monitor applications
* Manage container stdout & stderr logs
* Troubleshoot application failure
* Troubleshoot cluster component failure
* Troubleshoot networking