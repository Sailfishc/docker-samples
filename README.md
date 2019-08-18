# docker-samples
Docker部署案例

## 部署

### MYSQL 

- `cd mysql`
- `sudo docker-compose -p mysql -f ./docker-compose.yml up -d`

> 连接到MYSQL容器中

- `sudo docker exec -it [container-name] bash`