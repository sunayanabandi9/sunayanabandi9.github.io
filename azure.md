

Azure (Microsoft Azure) is a cloud computing service created by Microsoft for building, testing, deploying, and managing applications and services through Microsoft-managed data centers.

#### Federated Identity Credential Generation for AKS using Managed Identity

```
 cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: jump
  namespace: ${NAMESPACE}
spec:
  containers:
  - image: smallstep/step-cli
    name: step-cli
    command:
    - /bin/sh
    - -c
    - cat /var/run/secrets/tokens/test-token | step crypto jwt inspect --insecure
    volumeMounts:
    - mountPath: /var/run/secrets/tokens
      name: test-token
  serviceAccountName: ${SERVICE_ACCOUNT_NAME}
  volumes:
  - name: test-token
    projected:
      sources:
      - serviceAccountToken:
          path: test-token
          expirationSeconds: 3600
          audience: test
EOF
```
once the pod is up and running get the SERVICE_ACCOUNT_ISSUER  from the pod using the command

```
kubectl logs jump  | jq -r '.payload.iss'
```

Export the following environment variables:

```
export SERVICE_ACCOUNT_NAMESPACE="..."
export SERVICE_ACCOUNT_NAME="..."
export SERVICE_ACCOUNT_ISSUER="..."

# if you are using a Azure AD application
export APPLICATION_NAME="<your application name>"

# if you are using a user-assigned managed identity
export USER_ASSIGNED_MANAGED_IDENTITY_NAME="<your user-assigned managed identity name>"
export RESOURCE_GROUP="<your user-assigned managed identity resource group>"
```
Another approach

```
export RESOURCE_GROUP="myResourceGroup"
export LOCATION="westcentralus"
export SERVICE_ACCOUNT_NAMESPACE="default"
export SERVICE_ACCOUNT_NAME="workload-identity-sa"
export SUBSCRIPTION="$(az account show --query id --output tsv)"
export USER_ASSIGNED_IDENTITY_NAME="myIdentity"
export FEDERATED_IDENTITY_CREDENTIAL_NAME="myFedIdentity"
export AKS_OIDC_ISSUER="$(az aks show -n myAKSCluster -g "${RESOURCE_GROUP}" --query "oidcIssuerProfile.issuerUrl" -o tsv)"
az identity create --name "${USER_ASSIGNED_IDENTITY_NAME}" --resource-group "${RESOURCE_GROUP}" --location "${LOCATION}" --subscription "${SUBSCRIPTION}"
```
az login using fedrated token
```
az login --service-principal -u ${AZURE_CLIENT_ID} -t ${AZURE_TENANT_ID} --federated-token $(cat ${AZURE_FEDERATED_TOKEN_FILE})
```
