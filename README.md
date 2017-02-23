# How to use Harbor with Minio?

Project [Harbor](https://github.com/vmware/harbor) is an enterprise-class registry server that stores and distributes Docker images. By default, Harbor stores images on your local filesystem this document covers how to use Minio as your storage backend to store docker images.

## Prerequesites

- A running [Minio server](https://github.com/minio/minio#docker-container)

```sh
docker run -p 9000:9000 --name minio \
  -v /mnt/minio/export:/export \
  -v /mnt/minio/config:/root/.minio \
  minio/minio:RELEASE.2017-02-16T01-47-30Z server /export
```

- Installed Harbor registry

The binary of the installer can be downloaded from the [release](https://github.com/vmware/harbor/releases) page. Follow [harbor installation and configuration guide](https://github.com/vmware/harbor/blob/master/docs/installation_guide.md) for further instructions.

### Generate self-signed certificate

You need to download [generate_cert.go](https://golang.org/src/crypto/tls/generate_cert.go?m=text) which is a simple go tool for generating self-signed certificates. `generate_cert.go` also provides proper SAN certificates.

```sh
go run generate_cert.go -ca --host "172.23.0.7"
mv cert.pem /data/cert/server.crt
mv key.pem /data/cert/server.key
```

Make sure to restart your docker daemon with `--insecure-registry="172.23.0.7"` before proceeding further. IP or FQDN here points to the nginx proxy container started by Harbor registry docker compose file.

### Replace `harbor.cfg` 

Copy the [`harbor.cfg`](./harbor.cfg) to your local harbor directory. Make sure to edit the details for your local needs.

### Replace registry `config.yml` 

Copy the [`config.yml`](./config.yml) to your local harbor directory at `common/templates/registry/config.yml` . Make sure to edit storage details with your local Minio server credentials.

### Start Harbor registry

```sh
./install.sh
[Step 1]: preparing environment ...
loaded secret key
Clearing the configuration file: ./common/config/registry/root.crt
....
....
Generated configuration file: ./common/config/ui/private_key.pem
Generated configuration file: ./common/config/registry/root.crt
The configuration files are ready, please use docker-compose to start the service.

[Step 2]: checking existing instance of Harbor ...

Note: stopping existing Harbor instance ...
Stopping nginx ... done
Stopping harbor-jobservice ... done
Stopping harbor-ui ... done
Stopping harbor-db ... done
Stopping registry ... done
Stopping harbor-log ... done
Removing nginx ... done
Removing harbor-jobservice ... done
Removing harbor-ui ... done
Removing harbor-db ... done
Removing registry ... done
Removing harbor-log ... done
Removing network harbor_default

[Step 3]: starting Harbor ...
Creating network "harbor_default" with the default driver
Creating harbor-log
Creating harbor-db
Creating harbor-ui
Creating registry
Creating harbor-jobservice
Creating nginx

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
Password: 
Login Succeeded
```

Proceed to push your first image to the harbor registry. 

```sh
docker tag ubuntu 172.23.0.7/myproject/myrepo
docker push 172.23.0.7/myproject/myrepo
The push refers to a repository [172.23.0.7/myproject/myrepo]
5eb5bd4c5014: Pushed 
d195a7a18c70: Pushed 
af605e724c5a: Pushed 
59f161c3069d: Pushed 
4f03495a4d7d: Pushed 
latest: digest: sha256:4c0b138bdaaefa6a1c290ba8d8a97a568f43c0f8f25c733af54d3999da12dfd4 size: 1357
5eb5bd4c5014: Layer already exists 
d195a7a18c70: Layer already exists 
af605e724c5a: Layer already exists 
59f161c3069d: Layer already exists 
4f03495a4d7d: Layer already exists 
mytag: digest: sha256:4c0b138bdaaefa6a1c290ba8d8a97a568f43c0f8f25c733af54d3999da12dfd4 size: 1357
```

Now to validate if these are indeed stored on Minio server we we can use [mc](https://github.com/minio/mc), please install if you haven't done so already.

```sh
mc ls home/docker-registry/docker/registry/v2/repositories/
[2017-02-23 01:31:24 PST]     0B myproject/

mc ls home/docker-registry/docker/registry/v2/repositories/myproject/
[2017-02-23 01:31:27 PST]     0B myrepo/
```

That's it you are done.