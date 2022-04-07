## Minio安装

```shell
docker run -p 9090:9000 --name minio \
-d --restart=always \
-e MINIO_ACCESS_KEY=minio \
-e MINIO_SECRET_KEY=minio@321 \
-v /data/minio-storage:/data \
-v /data/docker/minio/config:/root/.minio \
minio/minio server /data \
--console-address ":9000" --address ":9090"
```

再通过9090端口访问页面服务

