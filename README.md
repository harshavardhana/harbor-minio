# How to use VMware Harbor with MinIO?

[Harbor](https://github.com/vmware/harbor) is an enterprise-class docker registry server to store and distribute container images. Follow this document to use MinIO object storage server as a storage backend for Harbor container registry.

## Prerequesites

- Install and run [MinIO server](https://github.com/minio/minio#stable)

```sh
podman run \
  -p 9000:9000 \
  -p 9001:9001 \
  minio/minio server /data{1...4} --console-address ":9001"
```

- Install Harbor registry

The binary of the installer can be downloaded from the [releases](https://github.com/goharbor/harbor/releases) page. Follow [harbor installation and configuration guide](https://goharbor.io/docs/2.3.0/install-config/) for further instructions.

### Edit `harbor.yml`

Add the `s3` configuration to the storage_service section in your `harbor/harbor.yml`.

```yml
storage_service:
  s3:
    accesskey: minio
    secretkey: minio123
    region: us-east-1
    regionendpoint: http://minio-endpoint:9000
    bucket: harbor
    secure: false
    v4auth: true
```

### Start Harbor registry

```sh
sudo ./install.sh
Creating harbor-jobservice ... done
Creating nginx             ... done
✔ ----Harbor has been installed and started successfully.----
```

Note that the default administrator username/password are `admin/Harbor12345`.

### Create a custom project

Visit <http://reg.example.com> and login to create a project name `myproject`

![HARBOR_PROJECT](https://github.com/harshavardhana/harbor-minio/blob/master/project.png?raw=true)

### Push your first image

Login to the harbor registry before attempting to push your first image.

```sh
docker login reg.example.com
Username: admin
Password: Harbor12345
Login Succeeded
```

Proceed to push your first image to the harbor registry.

```sh
docker tag ubuntu reg.example.com/myproject/myrepo
docker push reg.example.com/myproject/myrepo
The push refers to a repository [reg.example.com/myproject/myrepo]
5eb5bd4c5014: Pushed
d195a7a18c70: Pushed
...
```

To check if image has been successfully uploaded login from MinIO browser console through <http://YOUR-MINIO-IP:9000> with username `minio` and password `minio123`.

## Questions

Join [MinIO community](https://slack.minio.io)
