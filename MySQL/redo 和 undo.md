### redo 和 undo

#### redo 日志
redo 日志的作用是把在事务过程中的所有修改操作全部记录下来，在之后系统崩溃重启后，把事务的所做的修改全部恢复。

组成：
1. 内存中的重做缓冲（redo log buffer），易丢失，在内存中。
2. 重做日志（redo log file），是持久的，保存在磁盘中。

![1](https://i.loli.net/2019/10/18/itFoZHSRGby4jIn.jpg)

步骤 ：
1. 将原始数据从磁盘中取出
2. 修改数据的内存拷贝
3. 生成一条重做日志，并写入 redo log buffer ，记录的数据是被修改的值。
4. 当事务 commit 时，将 redo log buffer 中的内容刷新到 redo log file，采用追加写的方式。
5. 定期将内存中修改的数据刷新到磁盘中。


在 InnoDB 中，redo log 都是以 512 个字节的块的形式进行存储的，同时块的大小和磁盘扇区的大小一致，所以保证了重做日志的原子性，不会因为机器断电导致重做日志只写了一半。


##### mini-transaction

![mini-transaction](https://i.loli.net/2019/10/18/xKOzaLAF7uqtrye.jpg)


![mini-transaction2](https://i.loli.net/2019/10/18/q4fwxLPDVmYNK8i.jpg)


上图为重做日志的过程，每个 mini—transaction （mtr）对应一条 DML 语句。对数据修改后，产生一条 redo1，首先将其写入 mtr 私有 buffer 中，当 DML 语句结束后，将 redo1 的私有 buffer 拷贝到共有的 log buffer 中。当外部整个事务提交时，再将 redo log buffer 刷入 redo log file 中。


#### undo 日志

为了保证事务的原子性，就需要在异常发生时，对已经执行的操作进行回滚。在 mysql 中，回滚操作是由 undo 日志实现的，所有事务的修改都会记录在 undo log 中，在发生错误时进行回滚。

![undo 日志](https://i.loli.net/2019/10/18/HXG2v7Fbo1uqrP3.jpg)


undo 日志只能「逻辑地」将数据恢复成之前的样子，其实它做的是与修改相反的工作，如一个 insert 修改，那么 undo 日志就会生成一个 delete ，一个 update 生成一条相反的 update。

![undo 日志 2](https://i.loli.net/2019/10/18/XHEmSbOU4QVDkL3.jpg)


作用：
1. 事务回滚
2. MVCC（多版本并发控制）


写入时机：
1. DML 操作修改聚簇索引之前，记录 undo 日志。
2. 二级索引记录的修改，不记录 undo 日志。

x
为了保证原子性，必须将数据在事务钱写到磁盘中，只要事务成功，数据必然持久化。
undo log 必须先于数据持久化到磁盘中，可以用来回滚事务。




先回滚 redo 再回滚 undo。
redo 日志从前往后恢复。
undo 日志从后往前恢复。



#### 参考
[浅析MySQL事务中的redo与undo - 简书](https://www.jianshu.com/p/20e10ed721d0)
[『浅入深出』MySQL 中事务的实现](https://draveness.me/mysql-transaction)