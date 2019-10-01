# BerglasとCloud Buildを使って秘密情報をセキュアにdeploy（できるかも）

## デモ用に最初にやること

```bash
PS1="$ "
```

## Berglasなどのセットアップ

* 認証

```bash
gcloud auth application-default login
```

* ローカルPCにberglasをインストールします（Mac）

```bash
brew install berglas
```

* berglasの公式Dockerイメージを取得します

```bash
docker pull gcr.io/berglas/berglas:latest
```

* 環境変数を設定します

```bash
export PROJECT_ID=[Your Project ID]
export BUCKET_ID=[Your Preffered Bucke Name] # <- bucketは新しく作成するため、すでに存在する名前を設定してはダメ！
export KMS_KEY=projects/${PROJECT_ID}/locations/global/keyRings/berglas/cryptoKeys/berglas-key
```

* berglas-cloud-build-sample

```bash
export PROJECT_ID=berglas-cloud-build-sample
export BUCKET_ID=berglas-cloud-build-sample-bucket # <- bucketは新しく作成するため、すでに存在する名前を設定してはダメ！
export KMS_KEY=projects/${PROJECT_ID}/locations/global/keyRings/berglas/cryptoKeys/berglas-key
```

確認してみる

```bash
echo $PROJECT_ID
echo $BUCKET_ID
echo $KMS_KEY
```

* GCPのAPIを有効にします

```bash
gcloud services enable --project ${PROJECT_ID} \
  cloudkms.googleapis.com \
  storage-api.googleapis.com \
  storage-component.googleapis.com
```

* berglas環境を構築します

```bash
$ berglas bootstrap --project $PROJECT_ID --bucket $BUCKET_ID
Successfully created berglas environment:

  Bucket: berglas-secrets
  KMS key: projects/[Your Project ID]/locations/global/keyRings/berglas/cryptoKeys/berglas-key
```

## DBのパスワードをberglas経由でDockerイメージに渡す例

* Cloud Build APIを有効にします

```bash
gcloud services enable --project $PROJECT_ID cloudbuild.googleapis.com
```

* Cloud Buildのサービスアカウントのemailを取得します

```bash
PROJECT_NUMBER=$(gcloud projects describe ${PROJECT_ID} --format 'value(projectNumber)')
export SA_EMAIL=${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com
```

確認してみる

```bash
echo $PROJECT_NUMBER
echo $SA_EMAIL
```

* DBパスワードをberglasに渡します

```bash
$ berglas create ${BUCKET_ID}/db-pass "test1234" --key ${KMS_KEY}
Successfully created secret [db-pass] with generation [xxxxxxxxxxxxxxx]
```

作成者自身なのでもちろんDBパスワードに何が設定されているか見れます。

```bash
$ berglas access ${BUCKET_ID}/db-pass
test1234
```

* Cloud Buildのサービスアカウントに先程作成したsecretへのアクセス権を渡します

```bash
$ berglas grant ${BUCKET_ID}/db-pass --member serviceAccount:${SA_EMAIL}
Successfully granted permission on [db-pass] to:
- serviceAccount:xxxxxxxxxxxx@cloudbuild.gserviceaccount.com
```

* Cloud Buildで実行しててみます

```bash
gcloud builds submit \
  --project ${PROJECT_ID} \
  --substitutions=_BUCKET_ID=${BUCKET_ID} \
  .
```

`test1234` というDBパスワードがechoされれば成功です!

## 最後にsecretとして作成したDBパスワードを削除します

```bash
berglas delete ${BUCKET_ID}/db-pass
```

## 参考資料

* [GitHub - GoogleCloudPlatform/berglas: A tool for managing secrets on Google Cloud](https://github.com/GoogleCloudPlatform/berglas)
* [berglas/examples/cloudbuild at master · GoogleCloudPlatform/berglas · GitHub](https://github.com/GoogleCloudPlatform/berglas/tree/master/examples/cloudbuild)
