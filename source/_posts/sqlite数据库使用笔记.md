---
title: Sqlite数据库使用笔记
date: 2018-10-26 16:41:43
categories: SQL
tags: [SQL]
toc: true
description:  Everyone has talent. What is rare is the courage to follow the talent to the dark place where it leads.
---
# sqlite的特点
[sqlite](https://www.sqlite.org/index.html)只支持库级锁，同时只能允许一个写操作。但SQLite尽量延迟申请X锁，直到数据块真正写盘时才申请X锁，非常巧妙而有效。
>注: 读锁(S锁）、写锁(X锁）

[Is SQLite threadsafe?](https://www.sqlite.org/faq.html#q6)  SQLite官网上的最权威的解答,答案是sqlite是线程安全的。

# sqlite的线程模式
SQLite支持[3种线程模式](https://www.sqlite.org/threadsafe.html)

1. 单线程：禁用所有的mutex锁，并发使用时会出错。当SQLite编译时加了SQLITE_THREADSAFE=0参数，或者在初始化SQLite前调用sqlite3_config(SQLITE_CONFIG_SINGLETHREAD)时启用。

2. 多线程：只要一个数据库连接不被多个线程同时使用就是安全的。源码中是启用bCoreMutex，禁用bFullMutex。实际上就是禁用数据库连接和prepared statement（准备好的语句）上的锁，因此不能在多个线程中并发使用同一个数据库连接或prepared statement。当SQLite编译时加了SQLITE_THREADSAFE=2参数时默认启用。若SQLITE_THREADSAFE不为0，可以在初始化SQLite前，调用sqlite3_config(SQLITE_CONFIG_MULTITHREAD)启用；或者在创建数据库连接时，设置SQLITE_OPEN_NOMUTEX flag。

3. 串行：启用所有的锁，包括bCoreMutex和bFullMutex。因为数据库连接和prepared statement都已加锁，所以多线程使用这些对象时没法并发，也就变成串行了。当SQLite编译时加了SQLITE_THREADSAFE=1参数时默认启用。若SQLITE_THREADSAFE不为0，可以在初始化SQLite前，调用sqlite3_config(SQLITE_CONFIG_SERIALIZED)启用；或者在创建数据库连接时，设置SQLITE_OPEN_FULLMUTEX flag。

# sqlite的事务
[事务](https://www.sqlite.org/lang_transaction.html)是和数据库连接相关的，每个数据库连接（使用pager来）维护自己的事务，且同时只能有一个事务（但是可以用SAVEPOINT来实现内嵌事务）。数据库只有在[事务](https://www.sqlite.org/lang_transaction.html)中才能被更改。所有更改数据库的命令（除SELECT以外的所有SQL命令）都会自动开启一个新事务，并且当最后一个查询完成时自动提交。

下面用Android/Java来演示一下:
```
Database.beginTransaction();  //手动设置开始事务
dosomething();
userDatabase.setTransactionSuccessful();        //设置事务处理成功，不设置会自动回滚不提交
userDatabase.endTransaction();        //处理完成
```
事务在改写数据库文件时，会先生成一个rollback journal（回滚日志），记录初始状态（其实就是备份），所有改动都是在数据库文件上进行的。当事务需要回滚时，可以将备份文件的内容还原到数据库文件；提交成功时，默认的delete模式下会直接删除这个日志。这个日志也可以帮助解决事务执行过程中断电，导致数据库文件损坏的问题。但如果操作系统或文件系统有bug，或是磁盘损坏，则仍有可能无法恢复。
# sqlite的锁机制
SQLite基于锁来实现并发控制。SQLite的锁是粗粒度的，并不拥有PostgreSQL那样细粒度的行锁，这也使得SQLite较为轻量级。当一个连接要写数据库时，所有其它的连接都被锁住，直到写连接结束它的事务。

sqlite的数据库连接有5种锁状态:

| 状态  | 对应的锁  | 说明   |
| :--:      |  :--:    | :--: |
| 未加锁（unlock）  | —          | 未和数据库建立连接、已建立连接但是还没访问数据库、已用BEGIN开始了一个事务但未开始读写数据库，处于这些情形时是未加锁状态。|
| 共享（shared）|  共享锁 | 连接需要从数据库中读取数据时，需要申请获得一个共享锁，如果获得成功，则进入共享状态。|
| 预留（reserved）|  预留锁 | 连接需要写数据库时，首先申请一个预留锁，一个数据库同时只能有一个预留锁，预留锁可以与共享锁共存。获得预留锁后进入预留状态，这时会先在缓冲区中进行需要的修改、更新操作，操作后的结果依然保存在缓冲区中，未真正写入数据库。|
| 未决（pending）|  未决锁 | 连接从预留升为排它前，需要先升为未决，这时其它连接就不能获得共享锁了，但已经拥有共享锁的连接仍然可以继续正常读数据库，此时，拥有未决锁的连接等待其它拥有共享锁的连接完成工作并释放其共享锁后，提成到排它锁。|
| 排它（exclusive）|  排它锁| 连接需要提交修改时，需要将预留锁升为排它锁，这时其它连接都无法获得任何锁，直到当前连接的排它状态结束。|

- UNLOCKED：表示数据库此时并未被读写。
- SHARED：表示数据库可以被读取。SHARED锁可以同时被多个线程拥有。一旦某个线程持有SHARED锁，就没有任何线程可以进行写操作。
- RESERVED：表示准备写入数据库。RESERVED锁最多只能被一个线程拥有，此后它可以进入PENDING状态。
- PENDING：表示即将写入数据库，正在等待其他读线程释放SHARED锁。一旦某个线程持有PENDING锁，其他线程就不能获取SHARED锁。这样一来，只要等所有读线程完成，释放SHARED锁后，它就可以进入EXCLUSIVE状态了。
- EXCLUSIVE：表示它可以写入数据库了。进入这个状态后，其他任何线程都不能访问数据库文件。因此为了并发性，它的持有时间越短越好。

一个线程只有在拥有低级别的锁的时候，才能获取更高一级的锁。SQLite就是靠这5种类型的锁，巧妙地实现了读写线程的互斥。同时也可看出，写操作必须进入EXCLUSIVE状态，此时并发数被降到1，这也是SQLite被认为并发插入性能不好的原因。
另外，read-uncommitted和WAL模式会影响这个锁的机制。在这2种模式下，读线程不会被写线程阻塞，即使写线程持有PENDING或EXCLUSIVE锁。
# sqlite的死锁
在使用事务的情况下，SQLite的锁机制存在死锁的可能性。

举例说明死锁:
```
连接1：BEGIN （UNLOCKED）
连接1：SELECT ... （SHARED）
连接1：INSERT ... （RESERVED）
连接2：BEGIN （UNLOCKED）
连接2：SELECT ... （SHARED）
连接1：COMMIT （PENDING，尝试获取EXCLUSIVE锁，但还有SHARED锁未释放，返回SQLITE_BUSY）
连接2：INSERT ... （尝试获取RESERVED锁，但已有PENDING锁未释放，返回SQLITE_BUSY）
```
现在2个连接都在等待对方释放锁，于是就死锁了。当然，实际情况并没那么糟糕，任何一方选择不继续等待，回滚事务就行了。

不过要更好地解决这个问题，就必须更深入地了解事务了。
实际上BEGIN语句可以有3种起始状态：
- DEFERRED：默认值，开始事务时不获取任何锁。进行第一次读操作时获取。
- SHARED锁，进行第一次写操作时获取RESERVED锁。
- IMMEDIATE：开始事务时获取RESERVED锁。
EXCLUSIVE：开始事务时获取EXCLUSIVE锁。

现在考虑2个事务在开始时都使用IMMEDIATE方式：
```
连接1：BEGIN IMMEDIATE （RESERVED）
连接1：SELECT ... （RESERVED）
连接1：INSERT ... （RESERVED）
连接2：BEGIN IMMEDIATE （尝试获取RESERVED锁，但已有RESERVED锁未释放，因此事务开始失败，返回SQLITE_BUSY，等待用户重试）
连接1：COMMIT （EXCLUSIVE，写入完成后释放）
连接2：BEGIN IMMEDIATE （RESERVED）
连接2：SELECT ... （RESERVED）
连接2：INSERT ... （RESERVED）
连接2：COMMIT （EXCLUSIVE，写入完成后释放）
```
这样死锁就被避免了。

而EXCLUSIVE方式则更为严苛，即使其他连接以DEFERRED方式开启事务也不会死锁：
```
连接1：BEGIN EXCLUSIVE （EXCLUSIVE）
连接1：SELECT ... （EXCLUSIVE）
连接1：INSERT ... （EXCLUSIVE）
连接2：BEGIN （UNLOCKED）
连接2：SELECT ... （尝试获取SHARED锁，但已有EXCLUSIVE锁未释放，返回SQLITE_BUSY，等待用户重试）
连接1：COMMIT （EXCLUSIVE，写入完成后释放）
连接2：SELECT ... （SHARED）
连接2：INSERT ... （RESERVED）
连接2：COMMIT （EXCLUSIVE，写入完成后释放）
```
不过在并非很高的情况下，直接获取EXCLUSIVE锁的难度比较大；而且为了避免EXCLUSIVE状态长期阻塞其他请求，最好的方式还是让所有写事务都以IMMEDIATE方式开始。
顺带一提，要实现重试的话，可以使用sqlite3_busy_timeout()或sqlite3_busy_handler()函数。

由此可见，要想保证线程安全的话，可以有这4种方式：

1. SQLite使用单线程模式，用一个专门的线程访问数据库。
2. SQLite使用单线程模式，用一个线程队列来访问数据库，队列一次只允许一个线程执行，队列里的线程共用一个数据库连接。
3. SQLite使用多线程模式，每个线程创建自己的数据库连接。
4. SQLite使用串行模式，所有线程共用全局的数据库连接。

第一种方式太过麻烦，需要线程间通信。
第二种方式可以用dispatch_queue_create()来创建一个serial queue，或者用一个maxConcurrentOperationCount为1的NSOperationQueue来实现。
这种方式的缺点就是事务必须在一个block或operation里完成，否则会乱序；而耗时较长的事务会阻塞队列。另外，没法利用多核CPU的优势。

# sqlite的WAL模式
WAL的全称Write Ahead Log,修改并不直接写入到数据库文件中，而是写入到另外一个称为WAL的文件中；如果事务失败，WAL中的记录会被忽略，撤销修改；如果事务成功，它将在随后的某个时间被写回到数据库文件中，提交修改。

WAL使用检查点将修改写回数据库，默认情况下，当WAL文件发现有1000页修改时，将自动调用检查点。这个页数大小可以自行配置。

使用WAL的优势:
- 读写操作不再互相阻塞，一定程度上解决了SQLite在处理高并发上的性能瓶颈
- 大多数场景中，带来很大的性能提升
- 磁盘I/O行为更容易被预测

缺点:
除非数据达到GB级时，性能才会降低。

android中如何开启WAL模式
```
SQLiteDatabase db = SQLiteDatabase.openDatabase("db_filename", 
cursorFactory,CREATE_IF_NECESSARY, myDatabaseErrorHandler);
db.enableWriteAheadLogging();
```
来看看SQLiteDatabase开启WAL的核心方法源码:
```
public boolean enableWriteAheadLogging() {
        // make sure the database is not READONLY. WAL doesn't make sense for readonly-databases.
        if (isReadOnly()) {
            return false;
        }
        // acquire lock - no that no other thread is enabling WAL at the same time
        lock();
        try {
            if (mConnectionPool != null) {
                // already enabled
                return true;
            }
            if (mPath.equalsIgnoreCase(MEMORY_DB_PATH)) {
                Log.i(TAG, "can't enable WAL for memory databases.");
                return false;
            }
            // make sure this database has NO attached databases because sqlite's write-ahead-logging
            // doesn't work for databases with attached databases
            if (mHasAttachedDbs) {
                if (Log.isLoggable(TAG, Log.DEBUG)) {
                    Log.d(TAG,
                            "this database: " + mPath + " has attached databases. can't  enable WAL.");
                }
                return false;
            }
            mConnectionPool = new DatabaseConnectionPool(this);  // 创建数据库连接池，由于要支持并发访问所以需要连接池的支持
            setJournalMode(mPath, "WAL"); // 调用setJournalMode设置模式为WAL
            return true;
        } finally {
            unlock();
        }
    }
```
开启了WAL模式之后，事务的开始需要注意，在源码的注释是这样写到
```
Writers should use {@link #beginTransactionNonExclusive()} or
     * {@link #beginTransactionWithListenerNonExclusive(SQLiteTransactionListener)}
```
调用者需要使用beginTransactionNonExclusive或者beginTransactionWithListenerNonExclusive来开始事务，也就是执行：BEGIN IMMEDIATE; 支持多并发。

开启了WAL模式磁盘中是这样的文件格式:
```
xxx.db
xxx.db-shm
xxx.db-wal
```
>-shm文件包含-wal文件的数据索引，-shm文件提升-wal文件的读性能
如果-shm文件被删除，下次数据库连接时会自动新建一个-shm文件
如果执行了checkpoint命令，-war文件可以删除。