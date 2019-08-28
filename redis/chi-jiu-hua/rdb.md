**RDB持久化**是将当前进程中的数据生成快照保存到硬盘（因此也称作快照持久化），保存的文件后缀是rdb；当Redis重新启动时，可以读取快照文件恢复数据。

## **触发条件：**

### 手动触发

save命令和bgsave命令都可以生成RDB文件。

save命令会阻塞Redis服务器进程，直到RDB文件创建完毕为止，在Redis服务器阻塞期间，服务器不能处理任何命令请求。

而bgsave命令会创建一个子进程，由子进程来负责创建RDB文件，父进程(即Redis主进程)则继续处理请求。

### 自动触发

save m n

自动触发最常见的情况是在配置文件中通过save m n，指定当m秒内发生n次变化时，会触发bgsave。

#### save m n的实现原理

Redis的save m n，是通过serverCron函数、dirty计数器、和lastsave时间戳来实现的。

- serverCron是Redis服务器的周期性操作函数，默认每隔100ms执行一次；该函数对服务器的状态进行维护，其中一项工作就是检查 save m n 配置的条件是否满足，如果满足就执行bgsave。
- dirty计数器是Redis服务器维持的一个状态，记录了上一次执行bgsave/save命令后，服务器状态进行了多少次修改(包括增删改)；而当save/bgsave执行完成后，会将dirty重新置为0。
- astsave时间戳也是Redis服务器维持的一个状态，记录的是上一次成功执行save/bgsave的时间。

save m n的原理如下：每隔100ms，执行serverCron函数；在serverCron函数中，遍历save m n配置的保存条件，只要有一个条件满足，就进行bgsave。对于每一个save m n条件，只有下面两条同时满足时才算满足：

- 当前时间-lastsave > m
- dirty >= n

### 其它条件

- 在主从复制场景下，如果从节点执行**全量复制**操作，则主节点会执行bgsave命令，将rdb文件发送给从节点；
- 执行shutdown命令时，自动执行rdb持久化，

## 执行流程

![](D:\books\Import\java_base\assets\redis\rdb.png)

- Redis父进程首先判断：当前是否在执行save，或bgsave/bgrewriteaof（后面会详细介绍该命令）的子进程，如果在执行则bgsave命令直接返回。bgsave/bgrewriteaof  的子进程不能同时执行，主要是基于性能方面的考虑：两个并发的子进程同时执行大量的磁盘写操作，可能引起严重的性能问题。
- 父进程执行fork操作创建子进程，这个过程中父进程是阻塞的，Redis不能执行来自客户端的任何命令；
- 父进程fork后，bgsave命令返回”Background saving started”信息并不再阻塞父进程，并可以响应其他命令；
- 子进程创建RDB文件，根据父进程内存快照生成临时快照文件，完成后对原有文件进行原子替换；
- 子进程发送信号给父进程表示完成，父进程更新统计信息。

## 常用配置

- save m n：bgsave自动触发的条件；如果没有save m n配置，相当于自动的RDB持久化关闭，不过此时仍可以通过其他方式触发。
- stop-writes-on-bgsave-error   yes：当bgsave出现错误时，Redis是否停止执行写命令；设置为yes，则当硬盘出现问题时，可以及时发现，避免数据的大量丢失；设置为no，则Redis无视bgsave的错误继续执行写命令，当对Redis服务器的系统(尤其是硬盘)使用了监控时，该选项考虑设置为no。
- rdbcompression yes：是否开启RDB文件压缩。
- rdbchecksum yes：是否开启RDB文件的校验，在写入文件和读取文件时都起作用；关闭checksum在写入文件和启动文件时大约能带来10%的性能提升，但是数据损坏时无法发现。
- dbfilename dump.rdb：RDB文件名。
- dir ./：RDB文件和AOF文件所在目录。