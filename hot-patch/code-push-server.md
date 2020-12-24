# Code-Push-Server私有部署

CodePush Server is a CodePush progam server! microsoft CodePush cloud is slow in China, we can use this to build our's. I use qiniu to store the files, because it's simple and quick! Or you can use [local/s3/oss/tencentcloud] storage, just modify config.js file, it's simple configure.

源码地址: [code-push-server](https://github.com/lisong/code-push-server)

## docker 部署

开源地址提供的 docker 比较老，需要自行打包成 docker 镜像后部署
参考：https://github.com/lisong/code-push-server/issues/257

###1. 在 docker 文件夹中新建脚本 build.sh

```
PROJECT_NAME = 'code-push-server'
cd code-push-server
rm -rf temp
mkdir temp
cd ../../..
tar -zcv --exclude='.git' --exclude='.gitignore' -f test.tar.gz ./\*
mv ./test.tar.gz ./code-push-server/docker/code-push-server/temp
cd code-push-server/docker/code-push-server

docker build -f ./Dockerfile -t xuwaer/code-push-server:5.7.1 ./
```

###2. 修改 docker/code-push-server/Dockerfile

```
FROM node:8.11.4-alpine

COPY ./temp/test.tar.gz .
RUN tar xfz test.tar.gz; rm -rf test.tar.gz; cd code-push-server

ENTRYPOINT node ./code-push-server/bin/www
```

###3. 完成后修改 docker-compose.yml

```
tablee/code-push-server:v0.5.2 -> xuwaer/code-push-server:5.7.1
```

###4. 部署服务

参考：https://github.com/lisong/code-push-server/blob/master/docker/README.md

```
sudo docker stack deploy -c docker-compose.yml code-push-server
```