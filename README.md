# How to use Harbor with Minio?

Project [Harbor](https://github.com/vmware/harbor) is an enterprise-class docker registry server to store and distribute container images. Follow this document to use Minio object storage server as a storage backend for Harbor container registry.

## Prerequesites

- Install and run [Minio server](https://github.com/minio/minio#docker-container)

```sh
docker run -p 9000:9000 --name minio \
  -e "MINIO_ACCESS_KEY=minio" \
  -e "MINIO_SECRET_KEY=minio123" \
  -v /mnt/minio/export:/export \
  -v /mnt/minio/config:/root/.minio \
  minio/minio:RELEASE.2017-02-16T01-47-30Z server /export
```

- Install Harbor registry

The binary of the installer can be downloaded from the [release](https://github.com/vmware/harbor/releases) page. Follow [harbor installation and configuration guide](https://github.com/vmware/harbor/blob/master/docs/installation_guide.md) for further instructions.

### Edit `config.yml` 

Add the `s3` configuration to the storage section in your `common/templates/registry/config.yml` (right below the cache configuration). Here is a complete example [`config.yml`](./config.yml) for your reference.

```yml
storage:
    s3:
      accesskey: minio
      secretkey: minio123
      region: us-east-1
      regionendpoint: http://YOUR-MINIO-IP:9000
      bucket: docker-registry
      encrypt: false
      secure: false
      v4auth: true
      chunksize: 5242880
      rootdirectory: /
```

### Start Harbor registry

```sh
./install.sh
[Step 1]: preparing environment ...
loaded secret key
...
...
✔ ----Harbor has been installed and started successfully.----

Now you should be able to visit the admin portal at https://172.23.0.7. 
For more details, please visit https://github.com/vmware/harbor .
```

Note that the default administrator username/password are `admin/Harbor12345`.

### Create a custom project

Visit <https://172.23.0.7> and login to create a project name `myproject`

![HARBOR_PROJECT](https://github.com/harshavardhana/harbor-minio/blob/master/project.png?raw=true)

### Push your first image

Login to the harbor registry before attempting to push your first image.

```sh
docker login 172.23.0.7
Username: admin
Password: Harbor12345
Login Succeeded
```

Proceed to push your first image to the harbor registry. 

```sh
docker tag ubuntu 172.23.0.7/myproject/myrepo
docker push 172.23.0.7/myproject/myrepo
The push refers to a repository [172.23.0.7/myproject/myrepo]
5eb5bd4c5014: Pushed 
d195a7a18c70: Pushed 
...
```

To check if image has been successfully uploaded login from Minio browser console through <http://YOUR-MINIO-IP:9000> with username `minio` and password `minio123`.

## Questions

Join [Minio community](https://slack.minio.io)
