一、故障情况
```
【故障简述】：分析日志统计xxx导致磁盘被打满
```
【故障时间】：
```
1.故障发生时间 :2018-11-2 18:20
2.故障发现时间：2018-11-2 18:28 磁盘90%报警
3.止损操作开始时间：2018-11-2 18:33
4.止损操作完成时间：2018-11-2 18:39
5.服务恢复时间：
6.故障持续时长：
```
【故障发现】监控报警
```
【故障范围】：
【故障影响】：
【故障级别】（按照事故级别来写）
【故障原因】：
```
【故障分析】
```
追查重点：执行grep命令为什么会产生2T大小的log文件？
logs下总的日志大小
131G ./logs/
命令执行
```bash
 grep "sync whitelist" ./* | grep "true" > rds_v3+.log
 grep: write error
 前面测试命令时生成的rds_v3+.log的未删除，造成死循环，不断写入rds_v3+.log文件，又不断读取，最终造成海量文件的生成，磁盘占满
```
【处理过程】（精确到分钟级别）
```
17:30 在测试环境的日志试验并确保统计命令的准确性
17:36 开始扫描在logs下所有日志，并关注cpu.idle, io.utils, io.await等相关指标
18:10 观察期间未发现问题，外出吃饭
18:28 磁盘90%报警（实际已经达到95%），随即登录机器查看
18：30 磁盘打满
从18：29登录机器开始到18：32分，处理过程为
1.找到大文件，将rds_v3+.log mv 成 rds_v3+.log.bak
2.重启相关，发现没有任何效果
3.找不到rds_v3+.log.bak的inode，df -h查看已经恢复（是文龙停止了程序？待确认）
18:33 磁盘已经删除完成，重启服务
18:39 恢复正常
```
【暴露问题及改进措施】

| 暴露问题 | 改进措施 | 负责人 | 优先级 | 预期完成时间 | 任务连接|
| :--- | :----: | ----: | :--- | :----: | ----: |
| 磁盘满报警阈值过高，且只有1个阈值 | 优化报警 | xxx | 高 | xxx | xxx |
| 缺少磁盘增长率的报警    |   优化报警   | xxx    | xxx   | xxx      | xxx     |
