# Money Tracker

Project deployment submission Dicoding untuk aplikasi Money Tracker.

## Struktur

- `money-tracker` = frontend PHP CodeIgniter 3
- `money-tracker-api` = backend Node.js

## Resource Google Cloud

- App Engine
- Cloud SQL MySQL 5.7
- Cloud Storage
- Service Account untuk upload file

## Konfigurasi

### Cloud SQL

Buat database dan user untuk aplikasi.

Contoh konfigurasi backend:

```js
const connection = mysql.createConnection({
    host: 'CLOUD_SQL_PUBLIC_IP',
    user: 'DB_USER',
    database: 'DB_NAME',
    password: 'DB_PASSWORD'
})
```

### Cloud Storage

Isi project ID dan bucket name di backend:

```js
const gcs = new Storage({
    projectId: 'GCP_PROJECT_ID',
    keyFilename: pathKey
})

const bucketName = 'BUCKET_NAME'
```

### Service Account

Simpan credential JSON backend di:

```text
money-tracker-api/serviceaccountkey.json
```

Template contoh tersedia di:

```text
money-tracker-api/serviceaccountkey.sample.json
```

### Frontend

Set URL backend di:

```text
money-tracker/application/models/Record_model.php
```

Set base URL frontend di:

```text
money-tracker/application/config/config.php
```

## Deploy

### Backend

Masuk ke folder backend:

```bash
cd money-tracker-api
gcloud app deploy
```

Catat URL hasil deploy backend.

### Frontend

Masuk ke folder frontend:

```bash
cd money-tracker
gcloud app deploy
```

Catat URL hasil deploy frontend.

## Catatan

- File `serviceaccountkey.json` tidak ikut di-commit
- Bucket attachment harus public agar file bisa diakses
- Setelah deploy selesai, lakukan uji tambah record dan upload attachment
