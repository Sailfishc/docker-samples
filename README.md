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