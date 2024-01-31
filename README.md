# Helm 


<details>

<summary>Helm</summary>

### helm command to install and set the values 

```
helm install  --values stage-libs/values/values.yaml  stage-libs -o yaml ./stage-libs
```

</details>

# GCP Cheat sheet
#### gcloud to access Google Cloud using an existing service account

```
gcloud auth activate-service-account SERVICE_ACCOUNT@DOMAIN.COM --key-file=/path/key.json --project=PROJECT_ID
```
