# **Android文件与数据库**



#### **SharedPreference使用规范**

**1.定义**：

>SharedPreferences是Android平台上一个轻量级的存储辅助类，用来保存应用的一些常用配置，最终数据是以xml形式,在应用中通常做一些简单数据的持久化缓存。

**2.内容存储**

> SharedPreference 中只能存储简单数据类型（int、boolean、String 等），复杂数据类型建议使用文件、数据库等其他方式存储。

**3.提交**：

> SharedPreference 提 交 数 据 时 ， 尽 量 使 用 Editor#apply() ，而非 Editor#commit()。一般来讲，仅当需要确定提交结果，并据此有后续操作时，才使 用 Editor#commit()。 说明： SharedPreference 相关修改使用 apply 方法进行提交会先写入内存，然后异步写入 磁盘，commit 方法是直接写入磁盘。如果频繁操作的话 apply 的性能会优于 commit， apply 会将最后修改内容写入磁盘。但是如果希望立刻获取存储操作的结果，并据此 做相应的其他操作，应当使用 commit。

**4.跨进程通讯**：

推荐使用ContentProvider+SharedPreference结合使用。

#### **File文件使用规范**

**1.路径访问**

避免使用硬编码路径，使用Android文件系统的API访问。

- 不要硬编码全局的路径”/sdcard”，使用Environment.getExternalStorageDirectory() 或者相关的方法替代
- 不要硬编码应用路径： “/data/data/myapp/databases”, 使用 Context.getDatabasePath(), Context.getFilesDir()或者相关的方法替代。

外部存储APIs:

| APIs                                                         | 参考目录                    |
| ------------------------------------------------------------ | --------------------------- |
| `Environment.getDataDirectory()`                             | /data                       |
| `Environment.getDownloadCacheDirectory()`                    | /cache                      |
| `Environment.getExternalStorageDirectory()`                  | /storage/emulated/0         |
| `Environment.getExternalStoragePublicDirectory(String type)` | /storage/emulated/0/[@type] |
| `Environment.getRootDirectory()`                             | /system                     |

内部存储APIs：

| APIs                    | 参考目录                           |
| ----------------------- | ---------------------------------- |
| `Context#getDataDir()`  | `/data/user/0/[PackageName]`       |
| `Context#getFilesDir()` | `/data/user/0/[PackageName]/files` |
| `Context#getCacheDir()` | `/data/user/0/[PackageName]/cache` |



**2.访问外部存储时，检查外部存储的可用性**

检查是否安装SD卡

```java
if(Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState()))
```

检查读写权限： android 6.0以上动态申请权限

```java
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```

**3.使用FileProvider实现文件跨进程访问**

#### **数据库使用规范**

**1. 数据库的创建**

- 避免使用太多的数据库，不要为了每个一个表单独创建一个数据库，大多数应用应该只有一个数据库DB。
- 查询频繁的字段，考虑添加索引。若是确保表中数据唯一，添加唯一索引。

**2. 数据库 Cursor 关闭**。

- 无论Cursor操作是否正常结束，最后必须确保 Cursor 正常关闭。
- 一般在finally子句中关闭Cursor。

```java
 Cursor cursor=null;
        try {
             // 查询过程
        }catch (Exception e){
                e.printStackTrace();
        }finally {
                if (cursor!=null){
                    cursor.close();
                }
         }
```

**3.多线程访问,使用事务，避免同步问题**

**使用单例的数据库对象**：

通过SQLiteOpenHelper 获取数据库SQLiteDatabase 实例，Helper 中会自动缓存已经打开的SQLiteDatabase 实例，单个App 中应使用SQLiteOpenHelper 的单例模式确保数据库连接唯一。由于SQLite 自身是数据库级锁(等同于文件锁问题)，单个数据库操作是保证线程安全的（不能同时写入），transaction 是一次原子操作，因此处于事务中的操作是线程安全的。

推荐原子计数+单例SQLiteOpenHelper

```java
public class DatabaseManager {

    private AtomicInteger mOpenCounter = new AtomicInteger();

    private static DatabaseManager instance;
    private static SQLiteOpenHelper mDatabaseHelper;
    private SQLiteDatabase mDatabase;

    public static synchronized void initializeInstance(SQLiteOpenHelper helper) {
        if (instance == null) {
            instance = new DatabaseManager();
            mDatabaseHelper = helper;
        }
    }

    public static synchronized DatabaseManager getInstance() {
        if (instance == null) {
            throw new IllegalStateException(DatabaseManager.class.getSimpleName() +
                    " is not initialized, call initializeInstance(..) method first.");
        }

        return instance;
    }

    public synchronized SQLiteDatabase openDatabase() {
        if(mOpenCounter.incrementAndGet() == 1) {
            // Opening new database
            mDatabase = mDatabaseHelper.getWritableDatabase();
        }
        return mDatabase;
    }

    public synchronized void closeDatabase() {
        if(mOpenCounter.decrementAndGet() == 0) {
            // Closing database
            mDatabase.close();

        }
    }
}
```



**多线程写入数据库时，使用事务**：

```java
public void insert(SQLiteDatabase db, String userId, String content) {
    ContentValues cv = new ContentValues();
    cv.put("userId", userId);
    cv.put("content", content);
    db.beginTransaction();
    try {
        db.insert(TUserPhoto, null, cv);
        // 其他操作
        db.setTransactionSuccessful();
    } catch (Exception e) {
        // TODO
    } finally {
        db.endTransaction();
    }
}
```

**4.使用大量数据更新或者执行多个Sql语句使用事务**

对于批量插入或者多个sql执行的操作，请使用事务或其他能够提高I/O 效率的机制，保证执行速度。

```java
public void insertBulk(SQLiteDatabase db, ArrayList<UserInfo> users) {
    db.beginTransaction();
    try {
        for (int i = 0; i < users.size; i++) {
            ContentValues cv = new ContentValues();
            cv.put("userId", users[i].userId);
            cv.put("content", users[i].content);
            db.insert(TUserPhoto, null, cv);
        }
        // 其他操作
        db.setTransactionSuccessful();
    } catch (Exception e) {
        // TODO
    } finally {
        db.endTransaction();
    }
}
```

**5.防止SQL注入**

执行SQL 语句时，应使用`SQLiteDatabase#insert()`、`update()`、`delete()`，不要使用`SQLiteDatabase#execSQL()`，以免SQL 注入风险。

反例:

```java
public void update(SQLiteDatabase db, String userId, String content) {
  String sqlStmt = String.format("UPDATE %s SET content=%s WHERE userId=%s",
    TUserPhoto, userId, content);
    //请提高安全意识，不要直接执行字符串作为SQL 语句
    db.execSQL(sqlStmt);
}
```



**6.使用ContentProvider实现数据库的跨进程访问**：

ContentProvider 管理的数据存储在SQL 数据库中，应该避免将不受信任的外部数据直接拼接在原始SQL 语句中:

正例：

```java
// 使用一个可替换参数
String mSelectionClause = "var = ?";
String[] selectionArgs = {""};
selectionArgs[0] = mUserInput;
```

反例：

```java
String mSelectionClause = "var = " + mUserInput;
```



