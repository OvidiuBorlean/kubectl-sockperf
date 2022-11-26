# kubectl-sockperf
Kubectl Sockperf plugin - Latency Measurement in Kubernetes

# Objective
This document will provide a simple implementation of a kubectl plugin used for latency measurements between two Pods on a Kubernetes cluster. It can be run as a standalone bash script or as plugin with kubectl binary after copying the script content in somewhere in the local PATH.

# Prerequisites

This script uses the sockperf binary installed in pre-existing Pods in the AKS environment. It has been tested with a nginx based image container as in the sockpers is already present in the standard repositories. As it is using the kubectl to interact with AKS cluster, we need to have the binary installed and configured with proper credentials.

# Running the script

```
kubectl sockperf start <serverPod> <ClientPod> <Port>
```

By default, the client-side tests will run for 100 seconds, the results will be available on the stdout after the test finishes.

# Plugin code

```
#!/bin/bash

# optional argument handling
if [[ "$1" == "start" ]]
then
    #Getting the IP Address of the server Pod
    serverIP=$(kubectl get pod $2 -o jsonpath={.status.podIP})
    echo $serverIP
    echo "Starting Sockperf Server Install"
    kubectl exec -it $2 -- apt update
    kubectl exec -it $2 -- apt install sockperf
    kubectl exec -it $2 -- echo $2
    kubectl exec $2 -- sockperf sr --tcp -i $serverIP -p $4 &
    echo "Starting Sockperf Client Tests"
    kubectl exec -it $3 -- apt update
    kubectl exec -it $3 -- apt install sockperf
    kubectl exec -it $3 -- sockperf ping-pong -i $serverIP --tcp -m 350 -t 101 -p $4  --full-rtt
    echo "Done"

fi

if [[ "$1" == "version" ]]
then
    echo "1.0.2"
    exit 0
fi

# optional argument handling
if [[ "$1" == "config" ]]
then
    echo "$KUBECONFIG"
    exit 0
fi

echo "kubectl Sockperf Test Plugin v0.20211122"
echo "Usage: kubectl-sockperf <pod server> <pod client> <port>"
```
