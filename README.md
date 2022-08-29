# STEPS TO DEPLOY A HLF BLOCKCHAIN IN KUBERNETES WITH HLF OPERATOR

1. DO cluster connected
2. Install krew
3. Install HLF operator v1.6.0

``` bash
helm repo add kfs https://kfsoftware.github.io/hlf-helm-charts --force-update


helm install hlf-operator --version=1.6.0 kfs/hlf-operator

kubectl krew install hlf

# Get storage class
kubectl get sc

# Set storage class to use with resources
export SC=$(kubectl get sc -o=jsonpath='{.items[0].metadata.name}')

# Create namespace
kubectl create ns fabric

# Create CA for the org1
kubectl hlf ca create --storage-class=$SC --capacity=2Gi --name=org1-ca --enroll-id=enroll --enroll-pw=enrollpw --namespace=fabric

# Create CA for the org2
kubectl hlf ca create --storage-class=$SC --capacity=2Gi --name=org2-ca --enroll-id=enroll --enroll-pw=enrollpw --namespace=fabric

# Create CA for the orderer
kubectl hlf ca create --storage-class=$SC --capacity=2Gi --name=ord-ca --enroll-id=enroll --enroll-pw=enrollpw --namespace=fabric

export PEER_IMAGE=hyperledger/fabric-peer
export PEER_VERSION=2.4.3
export ORDERER_IMAGE=hyperledger/fabric-orderer
export ORDERER_VERSION=2.4.3

# REGISTERING THE CA FOR ORGANIZATIONS AND ORDERERS

# register ca for org1 - peer1
kubectl hlf ca register --name=org1-ca --user=org1-peer1 --secret=peerpw --type=peer --enroll-id=enroll --enroll-secret=enrollpw --mspid=Org1MSP --namespace=fabric

# register ca for org1 - peer2
kubectl hlf ca register --name=org1-ca --user=org1-peer2 --secret=peerpw --type=peer --enroll-id=enroll --enroll-secret=enrollpw --mspid=Org1MSP --namespace=fabric

# register ca for org2 - peer1
kubectl hlf ca register --name=org2-ca --user=org2-peer1 --secret=peerpw --type=peer --enroll-id=enroll --enroll-secret=enrollpw --mspid=Org2MSP --namespace=fabric

# register ca for org2 - peer2
kubectl hlf ca register --name=org2-ca --user=org2-peer2 --secret=peerpw --type=peer --enroll-id=enroll --enroll-secret=enrollpw --mspid=Org2MSP --namespace=fabric

# CREATING THE PEERS 

# Create peer one for org1
kubectl hlf peer create --storage-class=$SC --enroll-id=org1-peer1 --mspid=Org1MSP --enroll-pw=peerpw --capacity=5Gi --name=org1-peer1 --ca-name=org1-ca.fabric --namespace=fabric --statedb=couchdb --image=$PEER_IMAGE --version=$PEER_VERSION

# Create peer two for org1
kubectl hlf peer create --storage-class=$SC --enroll-id=org1-peer2 --mspid=Org1MSP --enroll-pw=peerpw --capacity=5Gi --name=org1-peer2 --ca-name=org1-ca.fabric --namespace=fabric --statedb=couchdb --image=$PEER_IMAGE --version=$PEER_VERSION

# Create peer one for org2
kubectl hlf peer create --storage-class=$SC --enroll-id=org2-peer1 --mspid=Org2MSP --enroll-pw=peerpw --capacity=5Gi --name=org2-peer1 --ca-name=org2-ca.fabric --namespace=fabric --statedb=couchdb --image=$PEER_IMAGE --version=$PEER_VERSION

# Create peer two for org2
kubectl hlf peer create --storage-class=$SC --enroll-id=org2-peer2 --mspid=Org2MSP --enroll-pw=peerpw --capacity=5Gi --name=org2-peer2 --ca-name=org2-ca.fabric --namespace=fabric --statedb=couchdb --image=$PEER_IMAGE --version=$PEER_VERSION

# SET THE ADMIN ENTITY for the ORG1
kubectl hlf ca register --name=org1-ca --user=admin --secret=adminpw --type=admin --enroll-id=enroll --enroll-secret=enrollpw --mspid=Org1MSP --namespace=fabric

# enroll the admin
kubectl hlf ca enroll --name=org1-ca --user=admin --secret=adminpw --ca-name=ca --output org1-peer.yaml --mspid=Org1MSP --namespace=fabric

# Set the admin entity for the org2
kubectl hlf ca register --name=org2-ca --user=admin --secret=adminpw --type=admin --enroll-id=enroll --enroll-secret=enrollpw --mspid=Org2MSP --namespace=fabric

kubectl hlf ca enroll --name=org2-ca --user=admin --secret=adminpw --ca-name=ca --output org2-peer.yaml --mspid=Org2MSP --namespace=fabric

# Set the orderer entity 
# register ca
kubectl hlf ca register --name=ord-ca --user=orderer --secret=ordererpw --type=orderer --enroll-id=enroll --enroll-secret=enrollpw --mspid=OrdererMSP --namespace=fabric

kubectl hlf ordnode create --storage-class=$SC --enroll-id=orderer --mspid=OrdererMSP --enroll-pw=ordererpw --capacity=2Gi --name=ord-node1 --ca-name=ord-ca.fabric --namespace=fabric --image=$ORDERER_IMAGE --version=$ORDERER_VERSION

# Register Admin user for orderer node
kubectl hlf ca register --name=ord-ca --user=admin --secret=adminpw --type=admin --enroll-id=enroll --enroll-secret=enrollpw --mspid=OrdererMSP --namespace=fabric

# Enroll the admin user for orderer node ca - self sign certificates
kubectl hlf ca enroll --name=ord-ca --user=admin --secret=adminpw --mspid=OrdererMSP --ca-name=ca --output=admin-ordservice.yaml --namespace=fabric

# Enroll with TLS certificates
kubectl hlf ca enroll --name=ord-ca --user=admin --secret=adminpw --mspid=OrdererMSP --ca-name=tlsca --output=admin-tls-ordservice.yaml --namespace=fabric

# Inspect config files
kubectl hlf inspect --output networkConfig.yaml -o Org1MSP -o OrdererMSP -o Org2MSP

# Add users ca's to network config file
kubectl hlf utils adduser --userPath=org1-peer.yaml --config=networkConfig.yaml --username=admin --mspid=Org1MSP

kubectl hlf utils adduser --userPath=org2-peer.yaml --config=networkConfig.yaml --username=admin --mspid=Org2MSP

```


ERROR: pvc when creating the second peer of org2
Warning  ProvisioningFailed    105s (x10 over 11m)  dobs.csi.digitalocean.com_csi-do-controller-0_13afcc46-9a3e-4f06-88cc-f9fbe6cac0fb  failed to provision volume with StorageClass "do-block-storage": rpc error: code = ResourceExhausted desc = volume limit (10) has been reached. Current number of volumes: 10. Please contact support.


