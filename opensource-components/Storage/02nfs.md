## nfs server docker安装

```shell
docker run -d --name nfs --privileged -p 2049:2049 -v /data/nfs-storage:/nfsshare -e SHARED_DIRECTORY=/nfsshare itsthenetwork/nfs-server-alpine:latest
```

