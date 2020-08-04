# aspnetcore_skywalking_docker-compose
Aspnet Core skywalking_skywalking_docker-compose
# docker 方式部署 aspnetcore 下的skywalking apm 7.0监控
因之前6.0版本可能出现运行一段时间可能出现数据出现rpc 连接出错，
  因此使用新版本skywalking7.0+es7.0版本实现


- elasticsearch:7.6.2
- skywalking-oap-server:7.0.0-es7
- apache/skywalking-ui:7.0.0

docker-compose.yml 
```
version: '3.3'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.6.2
    container_name: elasticsearch
    restart: always
    ports:
      - 9200:9200
    environment:
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - TZ=Asia/Shanghai
      
    ulimits:
      memlock:
        soft: -1
        hard: -1
  oap:
    image: apache/skywalking-oap-server:7.0.0-es7
    container_name: oap
    depends_on:
      - elasticsearch
    links:
      - elasticsearch
    restart: always
    ports:
      - 11800:11800
      - 12800:12800
    environment:
      SW_STORAGE: elasticsearch7
      SW_STORAGE_ES_CLUSTER_NODES: elasticsearch:9200
      TZ: Asia/Shanghai
  ui:
    image: apache/skywalking-ui:7.0.0
    container_name: ui
    depends_on:
      - oap
    links:
      - oap
    restart: always
    ports:
      - 8080:8080
    environment:
      SW_OAP_ADDRESS: oap:12800
      TZ: Asia/Shanghai
```
#常见问题
1.数据无法正常展示：注意保证修改docker下的时区，使其保持一致
```
    environment:
       TZ: Asia/Shanghai
```
2.查看对应的docker 日志可能需要重启下 ui 容器
```
docker restart ui
```
3.若出现无法连接其他如UI容器无法连接oap:12800，尝试使用ip替换容器名称/或者使用network
```
environment:
      SW_OAP_ADDRESS: 【ip】:12800
```
https://github.com/apache/skywalking
