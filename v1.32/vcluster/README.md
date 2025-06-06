# Run Conformance Tests

## 1. Create a multinode minikube cluster

```bash
minikube start --kubernetes-version 1.32.1 --nodes=2
```

## 2. Create the vcluster

Create a file called `vcluster.yaml` with the following content:

```yaml
controlPlane:
  advanced:
    virtualScheduler:
      enabled: true
  backingStore:
    etcd:
      deploy:
        enabled: true
        statefulSet:
          image:
            tag: 3.5.17-0
  distro:
    k8s:
      apiServer:
        extraArgs:
          - --service-account-jwks-uri=https://kubernetes.default.svc.cluster.local/openid/v1/jwks
        image:
          tag: v1.32.1
      controllerManager:
        image:
          tag: v1.32.1
      enabled: true
      scheduler:
        image:
          tag: v1.32.1
  statefulSet:
    scheduling:
      podManagementPolicy: OrderedReady

networking:
  advanced:
    proxyKubelets:
      byHostname: false
      byIP: false

sync:
  fromHost:
    csiDrivers:
      enabled: false
    csiStorageCapacities:
      enabled: false
    nodes:
      enabled: true
      selector:
        all: true
  toHost:
    persistentVolumes:
      enabled: true
    priorityClasses:
      enabled: true
    storageClasses:
      enabled: true
```

Create a virtual cluster using `vcluster version 0.23.0` (latest) using
the following command:

```bash
# Create the vCluster
vcluster create vcluster -n vcluster -f vcluster.yaml
```

## 3. Run Tests

Download a binary release of the CLI, or build it yourself by running:

```bash
go install github.com/vmware-tanzu/sonobuoy@latest
```

Deploy a Sonobuoy pod to your cluster with:

```bash
sonobuoy run --mode=certified-conformance --dns-pod-labels=k8s-app=vcluster-kube-dns
```

View actively running pods:

```bash
sonobuoy status
```

To inspect the logs:

```bash
sonobuoy logs
```

Once sonobuoy status shows the run as completed, copy the output directory from
the main Sonobuoy pod to a local directory:

```bash
outfile=$(sonobuoy retrieve)
```

This copies a single .tar.gz snapshot from the Sonobuoy pod into your local .
directory. Extract the contents into ./results with:

```bash
mkdir ./results; tar xzf $outfile -C ./results
```
