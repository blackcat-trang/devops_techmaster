# Giới thiệu

Bài lab tổng hợp với thao tác build và deploy một ưng dụng Java Maven lên môi trường Kubernetes

project demo được lấy tại github: https://github.com/liamhubian/techmaster-obo-web

# Mục tiêu bài lab

# Chuẩn bị

Để hoàn thành bài lab, người học cần chuẩn bị:

- Cài đặt kubectl trên client
- Cluster Kubernetes đang hoạt động và người học có khả năng kết nối tới cluster này
- Có thể sử dụng các công cụ như minikube, docker-desktop kubernetes, KinD,... trên môi trường lab
- thực hiện cấu hình alias trên terminal bằng command "alias k=kubectl"

# Các bước thực hiện
### Bước 0: (optional) chuẩn bị Cluster

### Bước 1: triển khai cơ sở dữ liệu MySQL
- create new mysql container
```bash
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123 mysql
```
- copy obo.sql to mysql container
```bash
docker cp obo.sql <container-name>:/
```
- import data && check data in mysql
```bash
# exec mysql container
docker exec -ti <container-name> bash 
# create new database
docker run -d -p 3306:3306 -e 
mysql> create database obo;
# import database
mysql -u root -p obo < obo.sql
# check table
mysql> use obo;
mysql> show tables;
```
### Bước 2: cài đặt ứng dụng và đóng gói dưới dạng container
- Update ip address of mysql in ~/obo-web/src/main/resources/application-dev.properties file (to get ip: docker inspect <container name>)
- Run maven container 
```bash
docker run -ti -v $PWD:/obo-web maven bash
mvn install
```
- Build image && push image to docker
```bash
docker build -t <<username>/obo-web:1.0 .
docker login
docker push <username>/obo-web:1.0
# Test maven app
docker run -p 8080:8080 obo-web:1.0
```
### Bước 3: triển khai ứng dụng trên môi trường Kubernetes
- create templates: deploy && config map
```bash
k create deployment obo-web --image obo-web:1.0 -o yaml --dry-run=client > ../templates/obo-web-deploy.yaml
k create configmap obo-web --from-file=src/main/resources/application-dev.properties -o yaml --dry-run=client > ../templates/obo-web-config.yaml

k apply -f templates/obo-web-config.yaml
k apply -f templates/obo-web-deploy.yaml

# check deploy?
k get deploy
```
### Bước 4: truy cập tới ứng dụng
```bash
k port-forward deploy/obo-web --address 0.0.0.0 8080:8080
```
Access page: curl 127.0.0.1:8080

<img src="/k8s/sample-project/images/sample-project.png" width="50%">

# Clean up
Sau khi hoàn thành bài lab, học viên thực hiện xóa các tài nguyên
```bash
k delete -f template/
```
> **Note**
> Hiện tại obo-web bị lỗi code nên obo-web-config.yaml không dùng được, phải sài ip address cứng (ip -a) khi đóng build images
