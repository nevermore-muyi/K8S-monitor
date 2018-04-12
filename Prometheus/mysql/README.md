### MySQL-exporter安装步骤

```
1.修改MySQL-exporter地址：
按照要求修改yaml文件，修改mysql-configmap.yaml以下内容，
maxscale:root@(maxscale-s.default:3306)/mysql
修改成访问数据库的形式如 username:password@(mysqldatabase:port)/databasename
```

```
2.修改Prometheus配置
参考prometheus-core-configmap.yaml配置文件，与普通配置的区别在于添加了
- job_name: 'mysql_exporter' 
  static_configs:
  - targets: ['mysql-exporter:9104']
安装K8S规则将Prometheus的configmap做相应修改即可。
```

