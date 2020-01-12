\# hbase-increment-export-snapshot
create_namespace 'test'


create_namespace 'demo'
create 'demo:test', {NAME => 'f1',COMPRESSION => 'SNAPPY'}
put 'demo:test', 'key1','f1:name','zhangsan'
put 'demo:test', 'key1','f1:sex','男'
put 'demo:test', 'key1','f1:age','20'

put 'demo:test', 'key2','f1:name','lisi'
put 'demo:test', 'key2','f1:sex','男'
put 'demo:test', 'key2','f1:age','23'

put 'demo:test', 'key3','f1:name','erya'
put 'demo:test', 'key3','f1:sex','女'
put 'demo:test', 'key3','f1:age','20'

put 'demo:test', 'key4','f1:name','wangwu'
put 'demo:test', 'key4','f1:sex','男'
put 'demo:test', 'key4','f1:age','21'

snapshot 'demo:test','demo_test_1'


\----------------------------------------------验证1------------------------------------------------------------------------
\#使用自带导出工具导出
hbase org.apache.hadoop.hbase.snapshot.ExportSnapshot -snapshot demo_test_1 -copy-to hdfs:///hbasebackup/demo_test_1
\#查看导出文件
hadoop fs -du -h /hbasebackup/demo_test_1

\#删除表后是使用导出文件恢复
hbase(main):031:0> disable 'demo:test'
hbase(main):032:0> drop 'demo:test'

sudo -u hbase hadoop fs -cp /hbasebackup/demo_test_1/.hbase-snapshot/demo_test_1 /hbase/.hbase-snapshot
sudo -u hbase hadoop fs -cp /hbasebackup/demo_test_1/archive/data/demo/test /hbase/archive/data/demo

\#重新创建表
create 'demo:test', {NAME => 'f1',COMPRESSION => 'SNAPPY'}
\#先禁用表
disable 'demo:test'
restore_snapshot 'demo_test_1'
\#等待COMPLETE完成再启用
enable 'demo:test'
\#校验数据
count 'demo:test'

\--------------------------------------------验证增量-------------------------------------------------------
put 'demo:test', 'key4','f1:name','wangwu'
put 'demo:test', 'key4','f1:sex','男/女'
put 'demo:test', 'key4','f1:age','22'

put 'demo:test', 'key5','f1:name','zhaoliu'
put 'demo:test', 'key5','f1:sex','男'
put 'demo:test', 'key5','f1:age','21'


snapshot 'demo:test','demo_test_2'


--------
hbase com.huidian.hadoop.hbase.dataExport.ExportSnapshot -snapshot demo_test_2 -copy-to hdfs:///hbasebackup/snapshot2-snapshot1/ -snapshot-old demo_test_1
\#sudo -u hdfs hadoop fs -rm -r /hbasebackup/snapshot2-snapshot1

\#删除原表及快照
hbase(main):031:0> disable 'demo:test'
hbase(main):032:0> drop 'demo:test'
delete_snapshot 'demo_test_1'
delete_snapshot 'demo_test_2'
\#查看hdfs目录下无数据后恢复快照及数据
sudo -u hbase hadoop fs -cp /hbasebackup/demo_test_1/.hbase-snapshot/demo_test_1 /hbase/.hbase-snapshot
sudo -u hbase hadoop fs -cp /hbasebackup/demo_test_1/archive/data/demo/test /hbase/archive/data/demo/
sudo -u hbase hadoop fs -cp /hbasebackup/snapshot2-snapshot1/.hbase-snapshot/demo_test_2 /hbase/.hbase-snapshot
sudo -u hbase hadoop fs -cp /hbasebackup/snapshot2-snapshot1/archive/data/demo/test /hbase/archive/data/demo/

\#sudo -u hdfs hadoop fs -rm -r /hbase/archive/data/demo/*
\#sudo -u hbase hadoop fs -mkdir /hbase/archive/data/demo
\#重新创建表
create 'demo:test', {NAME => 'f1',COMPRESSION => 'SNAPPY'}
\#先禁用表
disable 'demo:test'
restore_snapshot 'demo_test_2'
\#等待COMPLETE完成再启用
enable 'demo:test'
\#校验数据
count 'demo:test'
\--------------------------------------------oss改造---------------------------------------

