### MySQL-exporter安装步骤

```
1.添加用户赋予权限：
CREATE USER 'nevermore'@'%' IDENTIFIED BY 'nevermore';
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'nevermore'@'%' IDENTIFIED BY 'nevermore' WITH MAX_USER_CONNECTIONS 3;
flush privileges;
```

```
2.修改MySQL-exporter地址：
按照要求修改yaml文件，修改mysql-configmap.yaml以下内容，
nevermore:nevermore@(maxscale-s.default:3306)/
修改成访问数据库的形式如 username:password@(mysqldatabase:port)/
```

```
3.修改Prometheus配置
参考prometheus-core-configmap.yaml配置文件，与普通配置的区别在于添加了
- job_name: 'mysql_exporter' 
  static_configs:
  - targets: ['mysql-exporter:9104']
安装K8S规则将Prometheus的configmap做相应修改即可。
```

