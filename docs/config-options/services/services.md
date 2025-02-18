---
title: Default Kubernetes Services
description: To deploy Kubernetes, RKE deploys several default Kubernetes services. Read about etcd, kube-api server, kubelet, kube-proxy and more
---

To deploy Kubernetes, RKE deploys several core components or services in Docker containers on the nodes. Based on the roles of the node, the containers deployed may be different.

:::note

All services support <b>additional custom arguments, Docker mount binds, and extra environment variables.</b>

To configure advanced options for Kubernetes services such as `kubelet`, `kube-controller`, and `kube-apiserver` that are not documented below, see the [`extra_args` documentation](config-options/services/services-extras/) for more details.

:::

| Component               | Services key name in cluster.yml |
|-------------------------|----------------------------------|
| etcd                    | `etcd`                           |
| kube-apiserver          | `kube-api`                       |
| kube-controller-manager | `kube-controller`                |
| kubelet                 | `kubelet`                        |
| kube-scheduler          | `scheduler`                      |
| kube-proxy              | `kubeproxy`                      |

## etcd

Kubernetes uses [etcd](https://etcd.io/) as a store for cluster state and data. Etcd is a reliable, consistent and distributed key-value store.

RKE supports running etcd in a single node mode or in HA cluster mode. It also supports adding and removing etcd nodes to the cluster.

You can enable etcd to [take recurring snapshots](etcd-snapshots/#recurring-snapshots). These snapshots can be used to [restore etcd](etcd-snapshots/#etcd-disaster-recovery).

By default, RKE will deploy a new etcd service, but you can also run Kubernetes with an [external etcd service](config-options/services/external-etcd/).

## Kubernetes API Server

:::note Note for Rancher 2 users

If you are configuring Cluster Options using a [Config File](https://ranchermanager.docs.rancher.com/reference-guides/cluster-configuration/rancher-server-configuration/rke1-cluster-configuration#rke-cluster-config-file-reference) when creating [Rancher Launched Kubernetes](https://ranchermanager.docs.rancher.com/pages-for-subheaders/launch-kubernetes-with-rancher), the names of services should contain underscores only: `kube_api`. This only applies to Rancher v2.0.5 and v2.0.6.

:::

The [Kubernetes API](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) REST service, which handles requests and data for all Kubernetes objects and provide shared state for all the other Kubernetes components.

```yaml
services:
  kube-api:
    # IP range for any services created on Kubernetes
    # This must match the service_cluster_ip_range in kube-controller
    service_cluster_ip_range: 10.43.0.0/16
    # Expose a different port range for NodePort services
    service_node_port_range: 30000-32767
    pod_security_policy: false
    # Valid values are either restricted or privileged
    pod_security_configuration: restricted
    # Enable AlwaysPullImages Admission controller plugin
    # Available as of v0.2.0
    always_pull_images: false
    secrets_encryption_config:
      enabled: true
```

### Kubernetes API Server Options

RKE supports the following options for the `kube-api` service :

- **Service Cluster IP Range** (`service_cluster_ip_range`) - This is the virtual IP address that will be assigned to services created on Kubernetes. By default, the service cluster IP range is `10.43.0.0/16`. If you change this value, then it must also be set with the same value on the Kubernetes Controller Manager (`kube-controller`).
- **Node Port Range** (`service_node_port_range`) - The port range to be used for Kubernetes services created with the [type](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types) `NodePort`. By default, the port range is `30000-32767`.
- **Pod Security Policy** (`pod_security_policy`) - An option to enable the [Kubernetes Pod Security Policy](https://kubernetes.io/docs/concepts/policy/pod-security-policy/). By default, we do not enable pod security policies as it is set to `false`. This feature is removed as of Kubernetes v1.25.
    > **Note:** If you set `pod_security_policy` value to `true`, RKE will configure an  open policy to allow any pods to work on the cluster. You will need to configure your own policies to fully utilize PSP.
- **Pod Security Admission** (`pod_security_configuration`) - An option to enable the [Kubernetes Pod Security Admission](https://kubernetes.io/docs/concepts/security/pod-security-admission/). This feature is available as of RKE v1.4.4 for Kubernetes v1.23 and above.
- **Always Pull Images** (`always_pull_images`) - Enable `AlwaysPullImages` Admission controller plugin.  Enabling `AlwaysPullImages` is a security best practice. It forces Kubernetes to validate the image and pull credentials with the remote image registry. Local image layer cache will still be used, but it does add a small bit of overhead when launching containers to pull and compare image hashes. _Note: Available as of v0.2.0_
- **Secrets Encryption Config** (`secrets_encryption_config`) - Manage Kubernetes at-rest data encryption. Documented [here](../secrets-encryption/secrets-encryption.md)
## Kubernetes Controller Manager

:::note Note for Rancher 2 users

If you are configuring Cluster Options using a [Config File](https://ranchermanager.docs.rancher.com/reference-guides/cluster-configuration/rancher-server-configuration/rke1-cluster-configuration#rke-cluster-config-file-reference) when creating [Rancher Launched Kubernetes](https://ranchermanager.docs.rancher.com/pages-for-subheaders/launch-kubernetes-with-rancher), the names of services should contain underscores only: `kube_controller`. This only applies to Rancher v2.0.5 and v2.0.6.

:::

The [Kubernetes Controller Manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/) service is the component responsible for running Kubernetes main control loops. The controller manager monitors the cluster desired state through the Kubernetes API server and makes the necessary changes to the current state to reach the desired state.

```yaml
services:
    kube-controller:
      # CIDR pool used to assign IP addresses to pods in the cluster
      cluster_cidr: 10.42.0.0/16
      # IP range for any services created on Kubernetes
      # This must match the service_cluster_ip_range in kube-api
      service_cluster_ip_range: 10.43.0.0/16
```

### Kubernetes Controller Manager Options

RKE supports the following options for the `kube-controller` service:

- **Cluster CIDR** (`cluster_cidr`) - The CIDR pool used to assign IP addresses to pods in the cluster. By default, each node in the cluster is assigned a `/24` network from this pool for pod IP assignments. The default value for this option is `10.42.0.0/16`.
- **Service Cluster IP Range** (`service_cluster_ip_range`) - This is the virtual IP address that will be assigned to services created on Kubernetes. By default, the service cluster IP range is `10.43.0.0/16`. If you change this value, then it must also be set with the same value on the Kubernetes API server (`kube-api`).

## Kubelet

The [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/) services acts as a "node agent" for Kubernetes. It runs on all nodes deployed by RKE, and gives Kubernetes the ability to manage the container runtime on the node.

```yaml
services:
    kubelet:
     # Base domain for the cluster
     cluster_domain: cluster.local
     # IP address for the DNS service endpoint
     cluster_dns_server: 10.43.0.10
     # Fail if swap is on
     fail_swap_on: false
     # Generate per node serving certificate
     generate_serving_certificate: false
```

### Kubelet Options

RKE supports the following options for the `kubelet` service:

- **Cluster Domain** (`cluster_domain`) - The [base domain](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) for the cluster. All services and DNS records created on the cluster. By default, the domain is set to `cluster.local`.
- **Cluster DNS Server** (`cluster_dns_server`) - The IP address assigned to the DNS service endpoint within the cluster. DNS queries will be sent to this IP address which is used by KubeDNS. The default value for this option is `10.43.0.10`
- **Fail if Swap is On** (`fail_swap_on`) - In Kubernetes, the default behavior for the kubelet is to **fail** if swap is enabled on the node. RKE does **not** follow this default and allows deployments on nodes with swap enabled. By default, the value is `false`. If you'd like to revert to the default kubelet behavior, set this option to `true`.
- **Generate Serving Certificate** (`generate_serving_certificate`) - Generate a certificate signed by the `kube-ca` Certificate Authority for the kubelet to use as a server certificate. The default value for this option is `false`. Before enabling this option, please read [the requirements](#kubelet-serving-certificate-requirements)

### Kubelet Serving Certificate Requirements

If `hostname_override` is configured for one or more nodes in `cluster.yml`, please make sure the correct IP address is configured in `address` (and the internal address in `internal_address`) to make sure the generated certificate contains the correct IP address(es).

An example of an error situation is an EC2 instance where the the public IP address is configured in `address`, and `hostname_override` is used, the connection between `kube-apiserver` and `kubelet` will fail because the `kubelet` will be contacted on the private IP address and the generated certificate will not be valid (the error `x509: certificate is valid for value_in_address, not private_ip` will be seen). The resolution is to provide the internal IP address in `internal_address`.

For more information on host overrides, refer to the [node configuration page.](config-options/nodes/#overriding-the-hostname)

## Kubernetes Scheduler

The [Kubernetes Scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/) service is responsible for scheduling cluster workloads based on various configurations, metrics, resource requirements and workload-specific requirements.

Currently, RKE doesn't support any specific options for the `scheduler` service.

## Kubernetes Network Proxy
The [Kubernetes network proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/) service runs on all nodes and manages endpoints created by Kubernetes for TCP/UDP ports.

Currently, RKE doesn't support any specific options for the `kubeproxy` service.
