# [WIP](https://youtu.be/ubrA3W1JMk0?t=2093)

**Based on:**  
HGF k8s Workshop  
Github: https://github.com/aidtechnology/hgf-k8s-workshop  
Workshop part 1: https://www.youtube.com/watch?v=ubrA3W1JMk0  
Workshop part 2: https://www.youtube.com/watch?v=3tVk7yrGSSE

Pricing of running Kubernetes cluster on Azure:
5 nodes * €0,1746/hour = €0,87/hour = €637,15/month
Make sure to [stop](https://docs.microsoft.com/nl-nl/azure/aks/start-stop-cluster) or delete the cluster after following this guide (if you don't like to pay €600 a month)._

## Development cluster on Azure

Requirements:
- Azure CLI (check: `az --version` & `az account list`. `az login` if not authenticated)
- Azure subscription 
- Docker with kubernetes (check: `docker -v` & `kubectl version`)
- Helm (check: `helm version`. [Helm install docs](https://helm.sh/docs/intro/install/))
- Cryptogen (check: `cryptogen version`. [HLF install binaries](https://hyperledger-fabric.readthedocs.io/en/release-2.2/install.html))

```bash
# == Create kubernetes cluster on Azure

# Name for resource group
export GROUP=hlf-getting-started;

# Location of resource group
export LOCATION=westeurope;

# Name of cluster
export NAME=${GROUP}-aks

# Resource group
az group create -n $GROUP -l $LOCATION;

# Create Kubernetes cluster
az aks create -g $GROUP -n $NAME -s Standard_D2s_v4 --node-count 4 --generate-ssh-keys;

# Get access credentials for managed Kubernetes cluster
az aks get-credentials -g $GROUP -n $NAME;

# Check access to cluster
kubectl get nodes;

# == Generate cryptomaterial

# Clone basic configuration repository
git clone https://github.com/aidtechnology/hgf-k8s-workshop;
cd hgf-k8s-workshop;

# Navigate to config dir of example directory
cd ./dev_example/config/;

# Optional: take a look at crypto-config.yaml for some more information. You will recognize stuff later on.
# code ./crypto-config.yaml;

# Generate some crypto material 
cryptogen generate --config ./crypto-config.yaml;

# Optional: Take a look at the generated secrets & certs 
# tree ./crypto-config;
# code ./crypto-config;

# Create a kubernetes namespace for the orderers & peers
kubectl create ns orderers;
kubectl create ns peers;

# === Store some of the generated certificates in kubernetes

# Store orderer certificates variables
export MSP_DIR=./crypto-config/ordererOrganizations/orderers.svc.cluster.local/users/Admin@orderers.svc.cluster.local/msp/;
export ORG_CERT=./crypto-config/ordererOrganizations/orderers.svc.cluster.local/users/Admin@orderers.svc.cluster.local/msp/admincerts/Admin@orderers.svc.cluster.local-cert.pem;
export CA_CERT=./crypto-config/ordererOrganizations/orderers.svc.cluster.local/users/Admin@orderers.svc.cluster.local/msp/cacerts/ca.orderers.svc.cluster.local-cert.pem;

# Create orderer secrets in kubernetes namespace so the containers can access it
kubectl create secret generic -n orderers hlf--ord-admincert --from-file=cert.pem=$ORG_CERT;
kubectl create secret generic -n orderers hlf--ord-cacert --from-file=cert.pem=$CA_CERT;

# Same thing for peer certificates & key
export ORG_CERT=./crypto-config/peerOrganizations/peers.svc.cluster.local/users/Admin@peers.svc.cluster.local/msp/admincerts/Admin@peers.svc.cluster.local-cert.pem;
export CA_CERT=./crypto-config/peerOrganizations/peers.svc.cluster.local/users/Admin@peers.svc.cluster.local/msp/cacerts/ca.peers.svc.cluster.local-cert.pem;
export ORG_KEY=./crypto-config/peerOrganizations/peers.svc.cluster.local/users/Admin@peers.svc.cluster.local/msp/keystore/priv_sk;

kubectl create secret generic -n peers hlf--peer-admincert --from-file=cert.pem=$ORG_CERT;
kubectl create secret generic -n peers hlf--peer-cacert --from-file=cert.pem=$CA_CERT;
kubectl create secret generic -n peers hlf--peer-adminkey --from-file=cert.pem=$ORG_KEY;





```
