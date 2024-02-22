

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
