# 服务器配置：
1. cpu：E5506（2.13GHz） x 2 = 8cores
2. 磁盘：600G的SAS（15000转）
3. 内存：16G

# Infobright的brighthouse.ini配置文件：
ServerMainHeapSize = 4000（MB）
ServerCompressedHeapSize = 500（MB）
LoaderMainHeapSize = 800（MB）
ClusterSize = 2000（MB）
KNFolder = BH_RSI_Repository
AllowMySQLQueryPath = 1

# 数据文件：
533M的csv文件

# 测试：
1. 不同版本之间本地导入
版本号所用时间3.3.155s4.0.745s4.0.7(fifo)44s
* fifo的方式只有4.0+版本才支持，实现方式：
     mkdir /tmp/test.pipe
     cat 数据文件 > /tmp/test.pipe
     然后直接load data infile 的文件为/tmp/test.pipe即可

2. 4.0.7远程导入
远程导入，在mysql命令添加“--local-infile=1”选项
导入方式所用时间csvfile132scsvfile+compress51sfifo133sfifo+compress51s
* compress是只在mysql命令添加“--compress”选项

# 结论：
1. 4.0.7版本在数据导入方面稍有提升
2. 采用fifo方式到mysql文件能够减少一次磁盘io，
     同时能够尝试进行多台机器导入成功才算成功：
          将同份日志导入多台ib时，带多台ib均接受完以后，再断开fifo，这样便成功导入；否则最后写入一条错误日志，则导致日志错误，从而进行回滚；
          能否用transaction来做？性能如何？
3. 远程导入数据需要采用compress方式，速度明显提升