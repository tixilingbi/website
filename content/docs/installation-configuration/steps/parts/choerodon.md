+++
title = "微服务开发框架部署"
description = "微服务开发框架部署"
weight = 10
+++

# 分步部署微服务开发框架

<blockquote class="warning">
在此之前，应该准备好Mysql、Harbor、Kafka、Zookeeper、Gitlab、Minio，Chartmuseum这些组件的信息。按以下搭建顺序进行搭建，请不要随意调整搭建顺序。
以下验证部署是否成功如未特别说明则执行验证的环境为任意一台集群Master节点。
</blockquote>

<blockquote class="note">
部署成功后Choerodon平台默认登录名为admin，默认密码为admin。
</blockquote>

## 添加Choerodon Chart仓库

```
helm repo add c7n https://openchart.choerodon.com.cn/choerodon/c7n/
helm repo update
```

## 创建数据库

- 进入数据库

    ```bash
    # 获取pod的名称
    kubectl get po -n choerodon-devops-prod
    # 进入pod
    kubectl exec -it [mysql pod name] -n choerodon-devops-prod bash
    # 进入mysql命令行
    mysql -uroot -p${MYSQL_ROOT_PASSWORD}
    ```

- 创建choerodon所需数据库及用户并授权

    <blockquote class="note">
    部署完成后请注意保存用户名和密码。
    </blockquote>

    ```sql
    CREATE USER IF NOT EXISTS 'choerodon'@'%' IDENTIFIED BY "password";
    CREATE DATABASE IF NOT EXISTS event_store_service DEFAULT CHARACTER SET utf8;
    CREATE DATABASE IF NOT EXISTS iam_service DEFAULT CHARACTER SET utf8;
    CREATE DATABASE IF NOT EXISTS manager_service DEFAULT CHARACTER SET utf8;
    GRANT ALL PRIVILEGES ON event_store_service.* TO choerodon@'%';
    GRANT ALL PRIVILEGES ON iam_service.* TO choerodon@'%';
    GRANT ALL PRIVILEGES ON manager_service.* TO choerodon@'%';
    FLUSH PRIVILEGES;
    ```

## 部署register server

- 部署服务

    ```
    helm install c7n/go-register-server \
        --set service.enable=true \
        --set service.name=register-server \
        --set env.open.REGISTER_SERVICE_NAMESPACE=choerodon-devops-prod \
        --set env.open.KAFKA_ADDRESSES="kafka-0.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-1.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-2.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092" \
        --name register-server \
        --version=0.8.0 --namespace=choerodon-devops-prod
    ```

    参数名 | 含义 
    --- |  --- 
    service.enable|是否创建service对象
    service.name|service对象的名称
    env.open.KAFKA_ADDRESSES|kafka地址
    env.open.REGISTER_SERVICE_NAMESPACE|仅监听的namespace，多个namespace请用逗号隔开

- 验证部署
    - 验证命令

        ```
        curl $(kubectl get svc register-server -o jsonpath="{.spec.clusterIP}" -n choerodon-devops-prod):8000/eureka/apps
        ```
    - 出现以下类似信息即为成功部署

        ```json
        {
            "name": "go-register-server",
            "instance": [
                {
                "instanceId": "192.168.3.19:go-register-server:8000",
                "hostName": "192.168.3.19",
                "app": "go-register-server",
                "ipAddr": "192.168.3.19",
                "status": "UP",
                "overriddenstatus": "UNKNOWN",
                "port": {
                    "@enabled": true,
                    "$": 8000
                },
                "securePort": {
                    "@enabled": false,
                    "$": 443
                },
                "countryId": 8,
                "dataCenterInfo": {
                    "name": "MyOwn",
                    "@class": "com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo"
                },
                "leaseInfo": {
                    "renewalIntervalInSecs": 10,
                    "durationInSecs": 90,
                    "registrationTimestamp": 1528201698,
                    "lastRenewalTimestamp": 1528201698,
                    "evictionTimestamp": 0,
                    "serviceUpTimestamp": 1528201698
                },
                "metadata": {
                    "VERSION": "0.7.0"
                },
                "homePageUrl": "http://192.168.3.19:8000/",
                "statusPageUrl": "http://192.168.3.19:8000/info",
                "healthCheckUrl": "http://192.168.3.19:8000/health",
                "vipAddress": "go-register-server",
                "secureVipAddress": "go-register-server",
                "isCoordinatingDiscoveryServer": true,
                "lastUpdatedTimestamp": 1528201698,
                "lastDirtyTimestamp": 1528201698,
                "actionType": "ADDED"
                }
            ]
        }
        ```


## 部署config server

- 部署服务

    ```
    helm install c7n/config-server \
        --set service.enable=true \
        --set env.open.EUREKA_CLIENT_SERVICEURL_DEFAULTZONE="http://register-server.choerodon-devops-prod:8000/eureka/" \
        --set env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS="kafka-0.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-1.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-2.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092" \
        --set env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_ZK_NODES="zookeeper-0.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181\,zookeeper-1.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181\,zookeeper-2.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181" \
        --name config-server \
        --version=0.8.0 --namespace=choerodon-devops-prod
    ```
    参数名 | 含义 
    --- |  --- 
    env.open.EUREKA_CLIENT_SERVICEURL_DEFAULTZONE|注册服务地址
    env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS|kafk地址
    env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_ZK_NODES|zookeeper地址

- 验证部署
    - 验证命令

        ```
        curl -s $(kubectl get po -n choerodon-devops-prod -l choerodon.io/release=config-server -o jsonpath="{.items[0].status.podIP}"):8011/health | jq -r .status
        ```
    - 出现以下类似信息即为成功部署
        ```
        UP
        ```

## 部署manager service

- 部署服务

    ```
    helm install c7n/manager-service \
        --set preJob.preInitDB.mysql.host=choerodon-mysql \
        --set preJob.preInitDB.mysql.port=3306 \
        --set preJob.preInitDB.mysql.database=manager_service \
        --set preJob.preInitDB.mysql.username=choerodon \
        --set preJob.preInitDB.mysql.password=password \
        --set env.open.SPRING_DATASOURCE_URL="jdbc:mysql://choerodon-mysql:3306/manager_service?useUnicode=true&characterEncoding=utf-8&useSSL=false" \
        --set env.open.SPRING_DATASOURCE_USERNAME=choerodon \
        --set env.open.SPRING_DATASOURCE_PASSWORD=password \
        --set env.open.SPRING_CLOUD_CONFIG_ENABLED=true \
        --set env.open.SPRING_CLOUD_CONFIG_URI="http://config-server.choerodon-devops-prod:8010/" \
        --set env.open.CHOERODON_GATEWAY_DOMAIN="api.example.choerodon.io" \
        --set env.open.CHOERODON_SWAGGER_OAUTH_URL="http://api.example.choerodon.io/oauth/oauth/authorize" \
        --set env.open.SPRING_KAFKA_BOOTSTRAP_SERVERS="kafka-0.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-1.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-2.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092" \
        --set env.open.EUREKA_CLIENT_SERVICEURL_DEFAULTZONE="http://register-server.choerodon-devops-prod:8000/eureka/" \
        --set env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS="kafka-0.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-1.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-2.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092" \
        --set env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_ZK_NODES="zookeeper-0.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181\,zookeeper-1.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181\,zookeeper-2.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181" \
        --name manager-service \
        --version=0.8.1 --namespace=choerodon-devops-prod
    ```
    参数名 | 含义 
    --- |  --- 
    preJob.preInitDB.mysql{}|初始化数据库所需数据库信息
    env.open.SPRING_DATASOURCE_URL|数据库链接地址
    env.open.SPRING_DATASOURCE_USERNAME|数据库用户名
    env.open.SPRING_DATASOURCE_PASSWORD|数据库密码
    env.open.SPRING_CLOUD_CONFIG_ENABLED|启用配置中心
    env.open.SPRING_CLOUD_CONFIG_URI|配置中心地址
    env.open.CHOERODON_SWAGGER_OAUTH_URL|swagger授权地址
    env.open.CHOERODON_GATEWAY_DOMAIN|平台api地址
    env.open.EUREKA_CLIENT_SERVICEURL_DEFAULTZONE|注册服务地址
    env.open.SPRING_KAFKA_BOOTSTRAP_SERVERS|kafk地址
    env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS|kafk地址
    env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_ZK_NODES|zookeeper地址

- 验证部署
    - 验证命令

        ```
        curl -s $(kubectl get po -n choerodon-devops-prod -l choerodon.io/release=manager-service -o jsonpath="{.items[0].status.podIP}"):8964/health | jq -r .status
        ```
    - 出现以下类似信息即为成功部署
        ```
        UP
        ```

## 部署iam service
- 部署服务

    ```
    helm install c7n/iam-service \
        --set preJob.preConfig.mysql.host=choerodon-mysql \
        --set preJob.preConfig.mysql.port=3306 \
        --set preJob.preConfig.mysql.database=manager_service \
        --set preJob.preConfig.mysql.username=choerodon \
        --set preJob.preConfig.mysql.password=password \
        --set preJob.preInitDB.mysql.host=choerodon-mysql \
        --set preJob.preInitDB.mysql.port=3306 \
        --set preJob.preInitDB.mysql.database=iam_service \
        --set preJob.preInitDB.mysql.username=choerodon \
        --set preJob.preInitDB.mysql.password=password \
        --set env.open.SPRING_DATASOURCE_URL="jdbc:mysql://choerodon-mysql:3306/iam_service?useUnicode=true&characterEncoding=utf-8&useSSL=false" \
        --set env.open.SPRING_DATASOURCE_USERNAME=choerodon \
        --set env.open.SPRING_DATASOURCE_PASSWORD=password \
        --set env.open.EUREKA_CLIENT_SERVICEURL_DEFAULTZONE="http://register-server.choerodon-devops-prod:8000/eureka/" \
        --set env.open.SPRING_CLOUD_CONFIG_ENABLED=true \
        --set env.open.SPRING_CLOUD_CONFIG_URI="http://config-server.choerodon-devops-prod:8010/" \
        --set env.open.CHOERODON_EVENT_CONSUMER_KAFKA_BOOTSTRAP_SERVERS="kafka-0.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-1.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-2.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092" \
        --set env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS="kafka-0.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-1.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-2.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092" \
        --set env.open.SPRING_KAFKA_BOOTSTRAP_SERVERS="kafka-0.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-1.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-2.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092" \
        --set env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_ZK_NODES="zookeeper-0.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181\,zookeeper-1.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181\,zookeeper-2.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181" \
        --name iam-service \
        --version=0.8.0 --namespace=choerodon-devops-prod
    ```
    参数名 | 含义 
    --- |  --- 
    preJob.preConfig.mysql{}|初始化配置所需manager_service数据库信息
    preJob.preInitDB.mysql{}|初始化数据库所需数据库信息
    env.open.SPRING_DATASOURCE_URL|数据库链接地址
    env.open.SPRING_DATASOURCE_USERNAME|数据库用户名
    env.open.SPRING_DATASOURCE_PASSWORD|数据库密码
    env.open.SPRING_CLOUD_CONFIG_ENABLED|启用配置中心
    env.open.SPRING_CLOUD_CONFIG_URI|配置中心地址
    env.open.EUREKA_CLIENT_SERVICEURL_DEFAULTZONE|注册服务地址
    env.open.SPRING_KAFKA_BOOTSTRAP_SERVERS|kafk地址
    env.open.CHOERODON_EVENT_CONSUMER_KAFKA_BOOTSTRAP_SERVERS|kafk地址
    env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS|kafk地址
    env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_ZK_NODES|zookeeper地址

- 验证部署
    - 验证命令

        ```
        curl -s $(kubectl get po -n choerodon-devops-prod -l choerodon.io/release=iam-service -o jsonpath="{.items[0].status.podIP}"):8031/health | jq -r .status
        ```
    - 出现以下类似信息即为成功部署
        ```
        UP
        ```

## 部署api gateway
- 部署服务

    ```
    helm install c7n/api-gateway \
        --set preJob.preConfig.mysql.host=choerodon-mysql \
        --set preJob.preConfig.mysql.port=3306 \
        --set preJob.preConfig.mysql.database=manager_service \
        --set preJob.preConfig.mysql.username=choerodon \
        --set preJob.preConfig.mysql.password=password \
        --set service.enable=true \
        --set ingress.enable=true \
        --set ingress.host=api.example.choerodon.io \
        --set env.open.EUREKA_CLIENT_SERVICEURL_DEFAULTZONE="http://register-server.choerodon-devops-prod:8000/eureka/" \
        --set env.open.SPRING_CLOUD_CONFIG_ENABLED=true \
        --set env.open.SPRING_CLOUD_CONFIG_URI="http://config-server.choerodon-devops-prod:8010/" \
        --set env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS="kafka-0.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-1.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-2.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092" \
        --set env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_ZK_NODES="zookeeper-0.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181\,zookeeper-1.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181\,zookeeper-2.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181" \
        --name api-gateway \
        --version=0.8.0 --namespace=choerodon-devops-prod
    ```
    参数名 | 含义 
    --- |  --- 
    preJob.preConfig.mysql{}|初始化配置所需manager_service数据库信息
    service.enable|创建service对象
    ingress.enable|创建ingress对象
    ingress.host|域名地址，此处不能带http://
    env.open.SPRING_CLOUD_CONFIG_ENABLED|启用配置中心
    env.open.SPRING_CLOUD_CONFIG_URI|配置中心地址
    env.open.EUREKA_CLIENT_SERVICEURL_DEFAULTZONE|注册服务地址
    env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS|kafk地址
    env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_ZK_NODES|zookeeper地址

- 验证部署
    - 验证命令

        ```
        curl -s $(kubectl get po -n choerodon-devops-prod -l choerodon.io/release=api-gateway -o jsonpath="{.items[0].status.podIP}"):8081/health | jq -r .status
        ```
    - 出现以下类似信息即为成功部署
        ```
        UP
        ```

## 部署gateway helper
- 部署服务

    ```
    helm install c7n/gateway-helper \
        --set preJob.preConfig.mysql.host=choerodon-mysql \
        --set preJob.preConfig.mysql.port=3306 \
        --set preJob.preConfig.mysql.database=manager_service \
        --set preJob.preConfig.mysql.username=choerodon \
        --set preJob.preConfig.mysql.password=password \
        --set env.open.SPRING_DATASOURCE_URL="jdbc:mysql://choerodon-mysql:3306/iam_service?useUnicode=true&characterEncoding=utf-8&useSSL=false" \
        --set env.open.SPRING_DATASOURCE_USERNAME=choerodon \
        --set env.open.SPRING_DATASOURCE_PASSWORD=password \
        --set env.open.EUREKA_CLIENT_SERVICEURL_DEFAULTZONE="http://register-server.choerodon-devops-prod:8000/eureka/" \
        --set env.open.SPRING_CLOUD_CONFIG_ENABLED=true \
        --set env.open.SPRING_CLOUD_CONFIG_URI="http://config-server.choerodon-devops-prod:8010/" \
        --set env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS="kafka-0.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-1.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-2.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092" \
        --set env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_ZK_NODES="zookeeper-0.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181\,zookeeper-1.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181\,zookeeper-2.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181" \
        --name gateway-helper \
        --version=0.8.0 --namespace=choerodon-devops-prod
    ```
    参数名 | 含义 
    --- |  --- 
    preJob.preConfig.mysql{}|初始化配置所需manager_service数据库信息
    env.open.SPRING_DATASOURCE_URL|数据库链接地址
    env.open.SPRING_DATASOURCE_USERNAME|数据库用户名
    env.open.SPRING_DATASOURCE_PASSWORD|数据库密码
    env.open.SPRING_CLOUD_CONFIG_ENABLED|启用配置中心
    env.open.SPRING_CLOUD_CONFIG_URI|配置中心地址
    env.open.EUREKA_CLIENT_SERVICEURL_DEFAULTZONE|注册服务地址
    env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS|kafk地址
    env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_ZK_NODES|zookeeper地址

- 验证部署
    - 验证命令

        ```
        curl -s $(kubectl get po -n choerodon-devops-prod -l choerodon.io/release=gateway-helper -o jsonpath="{.items[0].status.podIP}"):9181/health | jq -r .status
        ```
    - 出现以下类似信息即为成功部署
        ```
        UP
        ```

## 部署oauth server
- 部署服务

    ```
    helm install c7n/oauth-server \
        --set preJob.preConfig.mysql.host=choerodon-mysql \
        --set preJob.preConfig.mysql.port=3306 \
        --set preJob.preConfig.mysql.database=manager_service \
        --set preJob.preConfig.mysql.username=choerodon \
        --set preJob.preConfig.mysql.password=password \
        --set env.open.SPRING_DATASOURCE_URL="jdbc:mysql://choerodon-mysql:3306/iam_service?useUnicode=true&characterEncoding=utf-8&useSSL=false" \
        --set env.open.SPRING_DATASOURCE_USERNAME=choerodon \
        --set env.open.SPRING_DATASOURCE_PASSWORD=password \
        --set env.open.SPRING_REDIS_HOST=devops-redis \
        --set env.open.SPRING_REDIS_PORT=6379 \
        --set env.open.CHOERODON_DEFAULT_REDIRECT_URL="http://example.choerodon.io" \
        --set env.open.EUREKA_CLIENT_SERVICEURL_DEFAULTZONE="http://register-server.choerodon-devops-prod:8000/eureka/" \
        --set env.open.SPRING_CLOUD_CONFIG_ENABLED=true \
        --set env.open.SPRING_CLOUD_CONFIG_URI="http://config-server.choerodon-devops-prod:8010/" \
        --set env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS="kafka-0.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-1.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-2.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092" \
        --set env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_ZK_NODES="zookeeper-0.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181\,zookeeper-1.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181\,zookeeper-2.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181" \
        --name oauth-server \
        --version=0.8.1 --namespace=choerodon-devops-prod
    ```
    参数名 | 含义 
    --- |  --- 
    preJob.preConfig.mysql{}|初始化配置所需manager_service数据库信息
    env.open.SPRING_DATASOURCE_URL|数据库链接地址
    env.open.SPRING_DATASOURCE_USERNAME|数据库用户名
    env.open.SPRING_DATASOURCE_PASSWORD|数据库密码
    env.open.SPRING_CLOUD_CONFIG_ENABLED|启用配置中心
    env.open.CHOERODON_DEFAULT_REDIRECT_URL|默认重定向地址
    env.open.SPRING_CLOUD_CONFIG_URI|配置中心地址
    env.open.EUREKA_CLIENT_SERVICEURL_DEFAULTZONE|注册服务地址
    env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS|kafk地址
    env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_ZK_NODES|zookeeper地址

- 验证部署
    - 验证命令

        ```
        curl -s $(kubectl get po -n choerodon-devops-prod -l choerodon.io/release=oauth-server -o jsonpath="{.items[0].status.podIP}"):8021/health | jq -r .status
        ```
    - 出现以下类似信息即为成功部署
        ```
        UP
        ```

## 部署event store service
- 部署服务

    ```
    helm install c7n/event-store-service \
        --set preJob.preConfig.mysql.host=choerodon-mysql \
        --set preJob.preConfig.mysql.port=3306 \
        --set preJob.preConfig.mysql.database=manager_service \
        --set preJob.preConfig.mysql.username=choerodon \
        --set preJob.preConfig.mysql.password=password \
        --set preJob.preInitDB.mysql.host=choerodon-mysql \
        --set preJob.preInitDB.mysql.port=3306 \
        --set preJob.preInitDB.mysql.database=event_store_service \
        --set preJob.preInitDB.mysql.username=choerodon \
        --set preJob.preInitDB.mysql.password=password \
        --set env.open.SPRING_DATASOURCE_URL="jdbc:mysql://choerodon-mysql:3306/event_store_service?useUnicode=true&characterEncoding=utf-8&useSSL=false" \
        --set env.open.SPRING_DATASOURCE_USERNAME=choerodon \
        --set env.open.SPRING_DATASOURCE_PASSWORD=password \
        --set env.open.SPRING_CLOUD_STREAM_DEFAULT_BINDER="kafka" \
        --set env.open.EUREKA_CLIENT_SERVICEURL_DEFAULTZONE="http://register-server.choerodon-devops-prod:8000/eureka/" \
        --set env.open.SPRING_CLOUD_CONFIG_ENABLED=true \
        --set env.open.SPRING_CLOUD_CONFIG_URI="http://config-server.choerodon-devops-prod:8010/" \
        --set env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS="kafka-0.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-1.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-2.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092" \
        --set env.open.SPRING_KAFKA_BOOTSTRAP_SERVERS="kafka-0.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-1.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-2.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092" \
        --set env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_ZK_NODES="zookeeper-0.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181\,zookeeper-1.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181\,zookeeper-2.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181" \
        --name event-store-service \
        --version=0.8.0 --namespace=choerodon-devops-prod
    ```
    参数名 | 含义 
    --- |  --- 
    preJob.preConfig.mysql{}|初始化配置所需manager_service数据库信息
    preJob.preInitDB.mysql{}|初始化数据库所需数据库信息
    env.open.SPRING_DATASOURCE_URL|数据库链接地址
    env.open.SPRING_DATASOURCE_USERNAME|数据库用户名
    env.open.SPRING_DATASOURCE_PASSWORD|数据库密码
    env.open.SPRING_CLOUD_CONFIG_ENABLED|启用配置中心
    env.open.SPRING_CLOUD_STREAM_DEFAULT_BINDER|中间件类型
    env.open.SPRING_CLOUD_CONFIG_URI|配置中心地址
    env.open.EUREKA_CLIENT_SERVICEURL_DEFAULTZONE|注册服务地址
    env.open.SPRING_KAFKA_BOOTSTRAP_SERVERS|kafk地址
    env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS|kafk地址
    env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_ZK_NODES|zookeeper地址

- 验证部署
    - 验证命令

        ```
        curl -s $(kubectl get po -n choerodon-devops-prod -l choerodon.io/release=event-store-service -o jsonpath="{.items[0].status.podIP}"):9011/health | jq -r .status
        ```
    - 出现以下类似信息即为成功部署
        ```
        UP
        ```

## 部署file service
- 部署服务

    ```
    helm install c7n/file-service \
        --set preJob.preConfig.mysql.host=choerodon-mysql \
        --set preJob.preConfig.mysql.port=3306 \
        --set preJob.preConfig.mysql.database=manager_service \
        --set preJob.preConfig.mysql.username=choerodon \
        --set preJob.preConfig.mysql.password=password \
        --set env.open.MINIO_ENDPOINT="http://minio.example.choerodon.io" \
        --set env.open.MINIO_ACCESSKEY=username \
        --set env.open.MINIO_SECRETKEY=password \
        --set env.open.EUREKA_CLIENT_SERVICEURL_DEFAULTZONE="http://register-server.choerodon-devops-prod:8000/eureka/" \
        --set env.open.SPRING_CLOUD_CONFIG_ENABLED=true \
        --set env.open.SPRING_CLOUD_CONFIG_URI="http://config-server.choerodon-devops-prod:8010/" \
        --set env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS="kafka-0.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-1.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-2.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092" \
        --set env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_ZK_NODES="zookeeper-0.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181\,zookeeper-1.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181\,zookeeper-2.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181" \
        --name file-service \
        --version=0.8.0 --namespace=choerodon-devops-prod
    ```
    参数名 | 含义 
    --- |  --- 
    preJob.preConfig.mysql{}|初始化配置所需manager_service数据库信息
    env.open.SPRING_CLOUD_CONFIG_ENABLED|启用配置中心
    env.open.SPRING_CLOUD_CONFIG_URI|配置中心地址
    env.open.EUREKA_CLIENT_SERVICEURL_DEFAULTZONE|注册服务地址
    env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS|kafk地址
    env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_ZK_NODES|zookeeper地址
    env.open.MINIO_ENDPOINT|minio地址
    env.open.MINIO_ACCESSKEY|minio access key
    env.open.MINIO_SECRETKEY|minio secret key

- 验证部署
    - 验证命令

        ```
        curl -s $(kubectl get po -n choerodon-devops-prod -l choerodon.io/release=file-service -o jsonpath="{.items[0].status.podIP}"):9091/health | jq -r .status
        ```
    - 出现以下类似信息即为成功部署
        ```
        UP
        ```

<!-- ### 部署hystrix dashboard
- 部署服务

    ```
    helm install c7n/hystrix-dashboard \
        --set preJob.preConfig.mysql.host=choerodon-mysql \
        --set preJob.preConfig.mysql.port=3306 \
        --set preJob.preConfig.mysql.database=manager_service \
        --set preJob.preConfig.mysql.username=choerodon \
        --set preJob.preConfig.mysql.password=password \
        --set env.open.EUREKA_CLIENT_SERVICEURL_DEFAULTZONE="http://register-server.choerodon-devops-prod:8000/eureka/" \
        --set env.open.SPRING_CLOUD_CONFIG_ENABLED=true \
        --set env.open.SPRING_CLOUD_CONFIG_URI="http://config-server.choerodon-devops-prod:8010/" \
        --set env.open.JAVA_OPTS="-server -Xmx1024m -Xms1024m -Xss2m -XX:NewSize=768m -XX:MaxNewSize=768m" \
        --name hystrix-dashboard \
        --version=0.6.0 --namespace=choerodon-devops-prod
    ```

- 验证部署
    - 验证命令

        ```
        curl -s $(kubectl get po -n choerodon-devops-prod -l choerodon.io/release=hystrix-dashboard -o jsonpath="{.items[0].status.podIP}"):9001/health | jq -r .status
        ```
    - 出现以下类似信息即为成功部署
        ```
        UP
        ```

### 部署hystrix turbine
- 部署服务
    ```
    helm install c7n/hystrix-turbine \
        --set preJob.preConfig.mysql.host=choerodon-mysql \
        --set preJob.preConfig.mysql.port=3306 \
        --set preJob.preConfig.mysql.database=manager_service \
        --set preJob.preConfig.mysql.username=choerodon \
        --set preJob.preConfig.mysql.password=password \
        --set env.open.SPRING_CLOUD_CONFIG_ENABLED=true \
        --set env.open.SPRING_CLOUD_CONFIG_URI="http://config-server.choerodon-devops-prod:8010/" \
        --set env.open.EUREKA_CLIENT_SERVICEURL_DEFAULTZONE="http://register-server.choerodon-devops-prod:8000/eureka/" \
        --set env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_BROKERS="kafka-0.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-1.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092\,kafka-2.kafka-headless.choerodon-devops-prod.svc.cluster.local:9092" \
        --set env.open.SPRING_CLOUD_STREAM_KAFKA_BINDER_ZK_NODES="zookeeper-0.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181\,zookeeper-1.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181\,zookeeper-2.zookeeper-headless.choerodon-devops-prod.svc.cluster.local:2181" \
        --name hystrix-turbine \
        --version=0.6.0 --namespace=choerodon-devops-prod
    ```

- 验证部署
    - 验证命令

        ```
        curl -s $(kubectl get po -n choerodon-devops-prod -l choerodon.io/release=hystrix-turbine -o jsonpath="{.items[0].status.podIP}"):9021/health | jq -r .status
        ```
    - 出现以下类似信息即为成功部署
        ```
        UP
        ``` -->

## 部署choerodon iam front

<blockquote class="note">
若部署<a href="../choerodon-front">整合前端</a>，请忽略本小节，因为整合前端中会包含本小节部署的功能。
</blockquote>

- 部署服务

    ```
    helm install c7n/choerodon-front-iam \
        --set preJob.preConfig.mysql.host=choerodon-mysql \
        --set preJob.preConfig.mysql.port=3306 \
        --set preJob.preConfig.mysql.dbname=iam_service \
        --set preJob.preConfig.mysql.username=choerodon \
        --set preJob.preConfig.mysql.password=password \
        --set env.open.PRO_API_HOST="api.example.choerodon.io" \
        --set env.open.PRO_CLIENT_ID="iam" \
        --set env.open.PRO_TITLE_NAME="Choerodon" \
        --set env.open.PRO_HEADER_TITLE_NAME="Choerodon" \
        --set env.open.PRO_HTTP="http" \
        --set ingress.host="iam.choerodon.example.choerodon.io" \
        --set service.enable=true \
        --set ingress.enable=true \
        --name=choerodon-front-iam \
        --version=0.8.0 --namespace=choerodon-devops-prod
    ```
    参数名 | 含义 
    --- |  --- 
    preJob.preConfig.mysql{}|初始化配置所需manager_service数据库信息
    env.open.PRO_API_HOST|api地址
    env.open.PRO_CLIENT_ID|client id
    env.open.PRO_TITLE_NAME|页面显示标题
    env.open.PRO_HEADER_TITLE_NAME|页面header标题
    env.open.PRO_HTTP|使用协议
    service.enable|创建service对象
    ingress.enable|创建ingress对象
    ingress.host|域名地址，此处不能带http://

- 验证部署
    - 验证命令

        ```
        curl $(kubectl get svc choerodon-front-iam -o jsonpath="{.spec.clusterIP}" -n choerodon-devops-prod)
        ```
    - 出现以下类似信息即为成功部署

        ```html
        <!DOCTYPE html><html><head><meta http-equiv="Content-type"content="text/html; charset=utf-8"><title>Choerodon</title><link rel="shortcut icon"href="favicon.ico"></head><body><div id="app"></div><script type="text/javascript"src="app/vendor_19e4b950.js"></script><script type="text/javascript"src="app/main_19e4b950.js"></script></body></html>
        ```

- 在访问搭建好的Choerodon的api，`api.example.choerodon.io/manager/swagger-ui.html`，选择`iam_service` -> `client-controller` -> `创建client`
  - 认证请使用用户名：admin，密码：admin
  - 提交以下数据，注意正式搭建时请替换以下值为真实值
      
        ```json
        {
            "accessTokenValidity": 60,
            "additionalInformation": "",
            "authorizedGrantTypes": "implicit,client_credentials,authorization_code,refresh_token",
            "autoApprove": "default",
            "name": "iam",
            "objectVersionNumber": 0,
            "organizationId": 1,
            "refreshTokenValidity": 60,
            "resourceIds": "default",
            "scope": "default",
            "secret": "secret",
            "webServerRedirectUri": "http://iam.choerodon.example.choerodon.io"
        }
        ```