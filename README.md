# Berglas example using Cloud Build

## Before

* Auth

```bash
gcloud auth application-default login
```


* Install berglas for Mac

```bash
brew install berglas
```

* Pull a official docker image

```bash
docker pull gcr.io/berglas/berglas:latest
```

* Export env variables

```bash
export PROJECT_ID=[Your Project ID]
export BUCKET_ID=[Your Preffered Bucke Name] # <- This bucket should not exist yet!
export KMS_KEY=projects/${PROJECT_ID}/locations/global/keyRings/berglas/cryptoKeys/berglas-key
```

* Enable GCP APIs

```bash
gcloud services enable --project ${PROJECT_ID} \
  cloudkms.googleapis.com \
  storage-api.googleapis.com \
  storage-component.googleapis.com
```

* Bootstrap

```bash
$ berglas bootstrap --project $PROJECT_ID --bucket $BUCKET_ID
Successfully created berglas environment:

  Bucket: berglas-secrets
  KMS key: projects/[Your Project ID]/locations/global/keyRings/berglas/cryptoKeys/berglas-key
```

![](7A1F0B6B-C2B2-4683-A68B-15D2C915223B.png)

![](42D8C5A6-7688-4ED4-A282-4CA5B013760D.png)


## Usage

* Enable Cloud Build API

```bash
gcloud services enable --project $PROJECT_ID cloudbuild.googleapis.com
```

* Get Cloud Build service account email

```bash
PROJECT_NUMBER=$(gcloud projects describe ${PROJECT_ID} --format 'value(projectNumber)')
export SA_EMAIL=${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com
```

* Create a database password

```bash
$ berglas create ${BUCKET_ID}/db-pass "test1234" --key ${KMS_KEY}
Successfully created secret [db-pass] with generation [xxxxxxxxxxxxxxx]
```

```bash
$ berglas access ${BUCKET_ID}/db-pass
test1234
```

* Grant service account access to the secret

```bash
$ berglas grant ${BUCKET_ID}/db-pass --member serviceAccount:${SA_EMAIL}
Successfully granted permission on [db-pass] to:
- serviceAccount:xxxxxxxxxxxx@cloudbuild.gserviceaccount.com
```

* Cloud Build

```yaml
TODO: not yet
```

```bash
gcloud builds submit \
  --project ${PROJECT_ID} \
  --substitutions=_BUCKET_ID=${BUCKET_ID} \
  .
```

## Clean up!

```bash
berglas delete ${BUCKET_ID}/db-pass
```


## 参考資料

- [GitHub - GoogleCloudPlatform/berglas: A tool for managing secrets on Google Cloud](https://github.com/GoogleCloudPlatform/berglas)
- [berglas/examples/cloudbuild at master · GoogleCloudPlatform/berglas · GitHub](https://github.com/GoogleCloudPlatform/berglas/tree/master/examples/cloudbuild)
