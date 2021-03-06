## Introduction

对于监控系统来讲，历史数据的存储，高效率查询，快速展现，是个很重要且困难的问题。这主要表现在下面三个方面：

1. 数据量大：目前我们的监控系统，每个周期，大概有2500万次数据采集项上报（上报周期为1分钟和5分钟两种，各占50%），一天24小时里，从来不会有低峰，不管是白天和黑夜，每个周期，总会有那么多的数据要更新。
2. 写操作多：一般的业务系统，通常都是读多写少，可以方便的使用各种缓存技术，再者各类数据库，对于查询操作的处理效率远远高于写操作。而监控系统恰恰相反，写操作远远高于读。每个周期几千万次的更新操作，对于常用数据库（MySQL、PostgreSQL、MongoDB）都不是最合适和擅长的。
3. 高效率的查：我们说监控系统读操作少，是说相对写入来讲。监控系统本身对于读的要求很高，用户经常会有查询上百个metric，在过去一天、一周、一月、一年的数据。如何在秒级返回给用户并在前端展现，这是一个不小的挑战。

graph所做的事情，就是把用户每次push上来的数据，进行采样存储，并提供查询接口。

我们参考RRDtool的理念，在数据每次存入的时候，会自动进行采样、归档。在默认的归档策略，一分钟push一次的频率下，历史数据保存5年。同时为了不丢失信息量，数据归档的时候，会按照平均值采样、最大值采样、最小值采样存三份。用户在查询某个metric，在过去一年的历史数据时，Graph会选择最合适的采样频率，返回采样过后的数据，提高了数据查询速度。

## Installation

```bash
# set $GOPATH and $GOROOT

mkdir -p $GOPATH/src/github.com/my-open-falcon
cd $GOPATH/src/github.com/my-open-falcon
git clone https://github.com/my-open-falcon/graph.git

cd graph
go get ./...
./control build

./control start
```

## Configuration

    pid: 绝对路径，进程启动后的pid文件，这对于graph的平滑重启很重要（如有必要，请修改）

    log: error/warn/info/debug/trace, 默认为info

    debug: true/false, 如果为true，配合debugChecksum一起使用

    debugChecksum: 如果debug为true，那么符合该checksum的counter，整个处理过程会在日志文件中详细打印，主要用于debug和排错

    http
        - enable: true/false, 表示是否开启该http端口，该端口为控制端口，主要用来对graph发送控制命令、统计命令、debug命令等
        - listen: 表示监听的http端口

    rpc
        - enable: true/false, 表示是否开启该rpc端口，该端口为数据接收端口
        - listen: 表示监听的http端口

    rrd
        - storage: 绝对路径，历史数据的文件存储路径（如有必要，请修改为合适的路径）

    db
        - dsn: MySQL的连接信息，默认用户名是root，密码为空，host为127.0.0.1，database为graph（如有必要，请修改）
        - maxIdle: MySQL连接池配置，连接池允许的最大连接数，保持默认即可
