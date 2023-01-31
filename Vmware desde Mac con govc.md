# Consultas y creacion de recursos en Vmware desde Mac con govc

1. Instalacion de govc en Mac

´´´
[jmanuelcalvo@Joses-MacBook ~]$ brew install govc
==> Fetching govc
==> Downloading https://ghcr.io/v2/homebrew/core/govc/manifests/0.30.1
...
´´´


export GOVC_INSECURE=1 # Don't verify SSL certs on vCenter
export GOVC_URL=10.198.16.4 # vCenter IP/FQDN
export GOVC_USERNAME=administrator@vsphere.local # vCenter username
export GOVC_PASSWORD=P@ssw0rd # vCenter password
export GOVC_DATASTORE=vsanDatastore # Default datastore to deploy to
export GOVC_NETWORK="Cluster01-LAN-1-Routable" # Default network to deploy to
export GOVC_RESOURCE_POOL='cluster01/Resources' # Default resource pool to deploy to
export GOVC_DATACENTER=DC01 # I have multiple DCs in this VC, so i'm specifying the default here
