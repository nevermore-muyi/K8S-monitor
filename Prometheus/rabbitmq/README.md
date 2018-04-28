### RabbitMQ-exporter安装步骤

```
1.修改rabbitmq-exporter地址：
按照要求修改yaml文件，修改rabbitmq-exporter-deployment.yaml环境变量，
RABBIT_URL修改成集群内rabbitmq访问域名；
RABBIT_USER修改成rabbitmq访问用户名，默认为guest；
RABBIT_PASSWORD修改成rabbitmq访问密码。默认为guest；
PUBLISH_PORT为Pod内部端口号，9090
```

```
2.修改Prometheus配置
参考rabbitmqs-core-configmap.yaml配置文件，与普通配置的区别在于添加了
- job_name: 'rabbitmq_exporter' 
  static_configs:
  - targets: ['rabbitmq-exporter:9099']
安装K8S规则将Prometheus的configmap做相应修改即可。
```

