### MongoDB-exporter安装步骤

```
1.修改mongodb-exporter地址：
安装之前需要给用户增加帐号，
db.createUser({
    user: "nevermore",
    pwd: "nevermore",
    roles: [
        { role: "clusterMonitor", db: "admin" },
        { role: "read", db: "local" }
    ]
})
按照要求修改yaml文件，修改mongodb-exporter-deployment.yaml环境变量，
MONGODB_URL修改成集群内mongodb访问域名；
```

```
2.修改Prometheus配置
参考mongodb-core-configmap.yaml配置文件，与普通配置的区别在于添加了
- job_name: 'mongodb-exporter' 
  static_configs:
  - targets: ['mongodb-exporter:9104']
安装K8S规则将Prometheus的configmap做相应修改即可。
```

