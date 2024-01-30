架构拓扑

硬件信息
role	ipAddress	

port

	version	配置信息	操作系统	

loadBalance


vm_insert	10.246.9.198	8480	v1.95.1	32c64g 200G	CentOS Linux release 7.6.1810 (Core) 	10.246.10.198
vm_select	10.246.9.199	8481	v1.95.1	32c64g 200G	CentOS Linux release 7.6.1810 (Core) 	10.246.10.199
vm_storage01	10.246.9.200	8482 insertAddr :8401 vmselectAddr:8402	v1.95.1	32c64g 2T	CentOS Linux release 7.6.1810 (Core) 	

vm_storage02	10.246.9.201	8482 insertAddr :8401 vmselectAddr:8402	v1.95.1	32c64g 2T	CentOS Linux release 7.6.1810 (Core) 	

接口信息
网址格式
数据摄取	http://<vminsert>:8480/insert/<accountID>/<suffix>	
http://10.246.10.198:8480/insert/0/prometheus/ #release
http://10.246.128.4:8480/insert/0/prometheus/ #test

Prometheus 查询 API	http://<vmselect>:8481/select/<accountID>/prometheus/<suffix>	http://10.246.10.199:8481/select/0/prometheus/
WebUI	http://<vmselect>:8481/select/<accountID>/vmui/	http://10.246.10.199:8481/select/0/vmui/
租户查询统计	http://<vmselect>:8481/api/v1/status/top_queries	http://10.246.10.199:8481/api/v1/status/top_queries
获取租户列表	http://<vmselect>:8481/admin/tenants	http://10.246.10.199:8481/admin/tenants
启动命令
#vm_insert 
ExecStart=/usr/local/sbin/vminsert-prod -loggerTimezone Asia/Shanghai -loggerFormat json -httpListenAddr :8480 -storageNode=10.246.9.200:8401,10.246.9.201:8401

#vm_select
ExecStart=/usr/local/sbin/vmselect-prod -loggerTimezone Asia/Shanghai -loggerFormat json -httpListenAddr :8481 -storageNode=10.246.9.200:8402,10.246.9.201:8402

#vm_storage01
ExecStart=/usr/local/sbin/vmstorage-prod -loggerTimezone Asia/Shanghai -storageDataPath /data/vmstorage-data -retentionPeriod 1 -loggerFormat json -httpListenAddr :8482 -vminsertAddr :8401 -vmselectAddr :8402  

#vm_storage02
ExecStart=/usr/local/sbin/vmstorage-prod -loggerTimezone Asia/Shanghai -storageDataPath /data/vmstorage-data -retentionPeriod 1 -loggerFormat json -httpListenAddr :8482 -vminsertAddr :8401 -vmselectAddr :8402  
Migrating data from prometheus
[root@bd-test-10-255-23-41 prometheus]# curl -XPOST 172.30.15.241:9090/api/v1/admin/tsdb/snapshot
{"status":"success","data":{"name":"20231211T055110Z-43c0ec6001ac3191"}}
[root@bd-test-10-255-23-41 prometheus]# vmctl-prod prometheus --vm-addr=http://10.246.9.198:8480 --vm-concurrency=16 --prom-snapshot=snapshots/20231211T055110Z-43c0ec6001ac3191/







监控面板

https://cnp-grafana.gwm.cn/d/victoria/

参考文档

https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html

https://github.com/VictoriaMetrics/VictoriaMetrics

https://prometheus.io/docs/practices/remote_write/#max_shards
