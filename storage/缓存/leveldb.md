##leveldb 安装及使用

//##leveldb 安装及使用
https://blog.csdn.net/ronon77/article/details/84908778

leveldb 简介
leveldb 是 Google 用 C++ 开发的一个快速的键值对存储数据库，提供从字符串键到字符串值的有序映射。

leveldb 安装
下载 leveldb
git clone https://github.com/google/leveldb.git
编译 leveldb
cd leveldb/
make
编译的动态库和静态库分别在 out-shared，out-static 下：

ls leveldb/out-shared/libleveldb.so.1.20
ls leveldb/out-static/libleveldb.a
安装 leveldb
只有动态库需要安装，静态库在你编译的时候直接链接即可

//# cp leveldb header file
sudo cp -r /leveldb/include/ /usr/include/
 
//# cp lib to /usr/lib/
sudo cp /leveldb/out-shared/libleveldb.so.1.20 /usr/lib/
 
//# create link
sudo ln -s /usr/lib/libleveldb.so.1.20 /usr/lib/libleveldb.so.1
sudo ln -s /usr/lib/libleveldb.so.1.20 /usr/lib/libleveldb.so
 
//# update lib cache
sudo ldconfig
查看安装是否成功

ls /usr/lib/libleveldb.so*
//# 显示下面 3 个文件即安装成功
/usr/lib/libleveldb.so.1.20
/usr/lib/libleveldb.so.1
/usr/lib/libleveldb.so
leveldb 使用
我们来编写一个 hello_leveldb.cc 来测试我们的 leveldb 。

//#include <iostream>
//#include <cassert>
//#include <cstdlib>
//#include <string>
// 包含必要的头文件
//#include <leveldb/db.h>
 
using namespace std;
 
int main(void)
{
    leveldb::DB *db = nullptr;
    leveldb::Options options;
    // 如果数据库不存在就创建
    options.create_if_missing = true;
    // 创建的数据库在 /tmp/testdb
    leveldb::Status status = leveldb::DB::Open(options, "/tmp/testdb", &db);
    assert(status.ok());
    std::string key = "A";
    std::string value = "a";
    std::string get_value;
    // 写入 key1 -> value1
    leveldb::Status s = db->Put(leveldb::WriteOptions(), key, value);
    // 写入成功，就读取 key:people 对应的 value
    if (s.ok())
        s = db->Get(leveldb::ReadOptions(), "A", &get_value);
    // 读取成功就输出
    if (s.ok())
        cout << get_value << endl;
    else
        cout << s.ToString() << endl;
    delete db;
    return 0;
}
编译 - 静态链接
cp leveldb/out-static/libleveldb.a ./
g++ hello_leveldb.cc -o hello_leveldb ./libleveldb.a -lpthread
编译 - 动态链接
g++ hello_leveldb.cc -o hello_leveldb -lpthread -lleveldb
运行结果
./hello_leveldb
//# 输出值为 a，说明成功存储和获取
a
 
//# 查看数据库
ls /tmp/testdb


https://blog.csdn.net/chdhust/article/details/49402989
https://blog.csdn.net/salyty/article/details/83106237
https://blog.csdn.net/chenriwei2/article/details/45178249
https://blog.csdn.net/weixin_42663840/article/details/82253556
https://blog.csdn.net/xiongwenwu/article/details/53262804

##leveldb源码
https://baijiahao.baidu.com/s?id=1634577516618476849&wfr=spider&for=pc
https://www.cnblogs.com/haippy/archive/2011/12/04/2276064.html
http://blog.itpub.net/31561269/viewspace-2375371/
https://yq.aliyun.com/articles/684218?utm_content=g_1000035187
https://github.com/google/leveldb/blob/master/benchmarks/db_bench.cc

##leveldb api
https://blog.csdn.net/qq_32293345/article/details/85063531

LevelDB是K-V数据库。

LevelDB在存储数据时，是根据记录的key值有序存储的，就是说相邻的key值在存储文件中是依次顺序存储的，而应用可以自定义key大小比较函数，LevleDb会按照用户定义的比较函数依序存储这些记录。

例如：int类型的键，levelDB默认按照自然顺序(升序)进行有序存储，其他类型可以自定义比较函数。

注：可以利用其有序存储这一特性进行范围查找。

LevelDB API的使用
1.导包命令：

import org.iq80.leveldb.*;
import static org.fusesource.leveldbjni.JniDBFactory.*;
import java.io.*;
2.关闭开启数据库

Options options = new Options();
options.createIfMissing(true);
DB db = factory.open(new File("example"), options);
try {
  // Use the db in here....
} finally {
  // Make sure you close the db to shutdown the 
  // database and avoid resource leaks.
  db.close();
}
3.增、查、删数据

db.put(bytes("Tampa"), bytes("rocks"));
String value = asString(db.get(bytes("Tampa")));
db.delete(bytes("Tampa"));
4.批量操作，Performing Batch/Bulk/Atomic Updates.

WriteBatch batch = db.createWriteBatch();
try {
  batch.delete(bytes("Denver"));
  batch.put(bytes("Tampa"), bytes("green"));
  batch.put(bytes("London"), bytes("red"));
 
  db.write(batch);
} finally {
  // Make sure you close the batch to avoid resource leaks.
  batch.close();
}
5. 遍历 Iterating key/values.

DBIterator iterator = db.iterator();
try {
  for(iterator.seekToFirst(); iterator.hasNext(); iterator.next()) {
    String key = asString(iterator.peekNext().getKey());
    String value = asString(iterator.peekNext().getValue());
    System.out.println(key+" = "+value);
  }
} finally {
  // Make sure you close the iterator to avoid resource leaks.
  iterator.close();
}
6. 快照 Working against a Snapshot view of the Database.

ReadOptions ro = new ReadOptions();
ro.snapshot(db.getSnapshot());
try {
  
  // All read operations will now use the same 
  // consistent view of the data.
  ... = db.iterator(ro);
  ... = db.get(bytes("Tampa"), ro);
 
} finally {
  // Make sure you close the snapshot to avoid resource leaks.
  ro.snapshot().close();
}
7. 自定义比较器 Using a custom Comparator.

DBComparator comparator = new DBComparator(){
    public int compare(byte[] key1, byte[] key2) {
        return new String(key1).compareTo(new String(key2));
    }
    public String name() {
        return "simple";
    }
    public byte[] findShortestSeparator(byte[] start, byte[] limit) {
        return start;
    }
    public byte[] findShortSuccessor(byte[] key) {
        return key;
    }
};
Options options = new Options();
options.comparator(comparator);
DB db = factory.open(new File("example"), options);
8. 禁用压缩 Disabling Compression

Options options = new Options();
options.compressionType(CompressionType.NONE);
DB db = factory.open(new File("example"), options);
9.配置缓存 Configuring the Cache

Options options = new Options();
options.cacheSize(100 * 1048576); // 100MB cache
DB db = factory.open(new File("example"), options);
9.获得近似尺寸 Getting approximate sizes.

long[] sizes = db.getApproximateSizes(new Range(bytes("a"), bytes("k")), new Range(bytes("k"), bytes("z")));
System.out.println("Size: "+sizes[0]+", "+sizes[1]);
10.获取数据库状态 Getting database status.

String stats = db.getProperty("leveldb.stats");
System.out.println(stats);
11.获取日志信息 Getting informational log messages.

Logger logger = new Logger() {
  public void log(String message) {
    System.out.println(message);
  }
};
Options options = new Options();
options.logger(logger);
DB db = factory.open(new File("example"), options);
12.销毁数据库 Destroying a database.

Options options = new Options();
factory.destroy(new File("example"), options);
13.修复数据库 Repairing a database.

Options options = new Options();
factory.repair(new File("example"), options);
14使用内存池可以提高本机内存分配的效率： Using a memory pool to make native memory allocations more efficient:

JniDBFactory.pushMemoryPool(1024 * 512);
try {
    // .. work with the DB in here, 
} finally {
    JniDBFactory.popMemoryPool();
}
 参考地址：

https://github.com/fusesource/leveldbjni


