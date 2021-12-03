# :computer: vCluster on OSX with Minikube

## Introduction - vClusters

vClusters are another cluster running inside your K8s cluster and they're very useful for deploying feature branches for testing. Find a lot more information at Loft.sh [vcluster site](https://www.vcluster.com/). At some point, this will be available [directly from Kubernetes](https://github.com/kubernetes-sigs/cluster-api-provider-nested/tree/main/virtualcluster) but the "paint isn't yet dry" on those features.

## what you'll get

The vCluster described herein uses service type `loadBalancer` which is layer 4. vCluster using ingress is in the works, and that is layer 7.

## what you'll need

- minikube

```console
brew install minikube
```

- hyperkit (there are issues with Docker networking)

```console
brew install hyperkit
```

- vcluster

```console
curl -s -L "https://github.com/loft-sh/vcluster/releases/latest" | sed -nE 's!.*"([^"]*vcluster-darwin-amd64)".*!https://github.com\1!p' | xargs -n 1 curl -L -o vcluster && chmod +x vcluster;
sudo mv vcluster /usr/local/bin;
```

## `minikube` with MetalLB

```console
minikube start --driver=hyperkit
minikube addons enable metallb
minikube addons configure metallb
```

metallb will ask you for `start` and `end` IP ranges.

IPs need to be in the same /24 that minikube is in. Determine the minikube address space by running `minikube ip`. The default is 192.168.64.0/24. It's safe to pick a range high up in the block 192.168.64.220 for start and 192.168.64.240 for end.

## create your vCluster

open antother terminal windows using a different window profile _maybe `grass` or `red sands`_. This will be your `vCluster` shell.

in your vcluster shell...

1. `mkdir cluster1 && cd cluster1` and create and use your vcluster named vcluster1

```console
mkdir cluster1 && cd cluster1
vcluster create vcluster-1 -n host-namespace-1 --expose
vcluster connect vcluster-1 -n host-namespace-1
```

now run `kubectl get nodes` from both the vcluster and regular shell.

run `kubectl get all --all-namespaces` from your regular shell to see what's been deployed.

## Let's deploy some pods into the vcluster

> use the vcluster shell for this

this will place a `kubeconfig.yaml` into this directory. export the kubeconfig to the current shell.

```console
export KUBECONFIG=./kubeconfig.yaml
```

1. deploy an Nginx instance with service type loadbalancer
    - `kubectl apply -f ../nginxdemo.yaml`
    - `k get svc` will show you the external IP and port which you can hit using cURL. Chances are it's the first IP in the IP pool you picked.

    ```console
    curl 192.168.64.220:8080
    ```

    - any further work on the vCluster should be done using the vCluster window but any services are exposed to any console.

when you're done with the vCluster you can delete it with `vcluster delete <vcluster name> -n <namespace>`
