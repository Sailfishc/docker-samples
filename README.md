# docker-samples
Docker部署案例

## 部署

### MYSQL 

- `cd mysql`
- `sudo docker-compose -p mysql -f ./docker-compose.yml up -d`

> 连接到MYSQL容器中

- `sudo docker exec -it [container-name] bash`

### ZK

- `cd zk`
- `sh ./bin/compose.sh up -d`

## Docker常用命令

- 进入docker redis-cli：docker exec -it [container-name] redis-cli
- 进入docker zk：docker exec -it [zk-name] zkCli.sh
- docker-compose启动：docker-compose [-f] [docker-compose-file-alias] -d[&]
    - -f是可选的，如果文件不是docker-compose.yaml则需要指定文件名
    - -d是标准启动，&将终端窗口返回，不太正规，但是可以再开发环境看到日志输出，方便开发结果
- docker-compose top：列出docker进程


### OpenGrok

- https://hub.docker.com/r/opengrok/docker/

## Nas

### NextCloud

- https://hub.docker.com/_/nextcloud/

### jeyllfin