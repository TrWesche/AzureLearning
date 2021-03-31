1. Replicate a registry to a target region
    - az acr replication create --registry $ACR_NAME --location <location>
        - ACR_NAME = Name of registry
        - location = Azure location where replicant will be created.  For example japaneast.
2. Retrieve list of all container image replicas
    - az acr replication list --registry $ACR_NAME --output table

Note: This can also be done via the Azure portal via the Replications map