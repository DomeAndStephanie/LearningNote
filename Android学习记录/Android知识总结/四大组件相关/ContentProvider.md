[TOC]

#### ContentProvider知识总结

---

##### 一. 概述

​		ContentProvider是Android四大组件之一，用来对数据进行封装，向外部提供统一的数据访问接口。可用于在不同的应用程序之间实现数据共享。

##### 二. 使用方法

1. 继承ContentProvider

   ```java
   public class MyProvider extends ContentProvider {
   
       public MyProvider() {
       }
   
       @Override
       public int delete(Uri uri, String selection, String[] selectionArgs) {
           // Implement this to handle requests to delete one or more rows.
           throw new UnsupportedOperationException("Not yet implemented");
       }
   
       @Override
       public String getType(Uri uri) {
           // TODO: Implement this to handle requests for the MIME type of the data
           // at the given URI.
           throw new UnsupportedOperationException("Not yet implemented");
       }
   
       @Override
       public Uri insert(Uri uri, ContentValues values) {
           // TODO: Implement this to handle requests to insert a new row.
           throw new UnsupportedOperationException("Not yet implemented");
       }
   
       @Override
       public boolean onCreate() {
           // TODO: Implement this to initialize your content provider on startup.
           return false;
       }
   
       @Override
       public Cursor query(Uri uri, String[] projection, String selection,
                           String[] selectionArgs, String sortOrder) {
           // TODO: Implement this to handle query requests from clients.
           throw new UnsupportedOperationException("Not yet implemented");
       }
   
       @Override
       public int update(Uri uri, ContentValues values, String selection,
                         String[] selectionArgs) {
           // TODO: Implement this to handle requests to update one or more rows.
           throw new UnsupportedOperationException("Not yet implemented");
       }
   
   }
   
   ```

2. 在AndroidManifest中进行注册

   ```xml
   <provider
             android:name=".MyProvider"
             android:authorities="com.wx.test.MyProvider"
             android:exported="true" />
   ```

   name：自定义ContentProvider类名

   authorities：唯一标识，外部通过该值访问该ContentProvider

   exported：表示是否允许其它应用访问。如果包含`intent-filter`则默认为true,否则为false

3. 实现六个方法

   | 方法签名                                                | 含义             |
   | ------------------------------------------------------- | ---------------- |
   | `Uri insert(Uri, ContentValues)`                        | 插入新数据       |
   | `Cursor query(Uri,projection,selection,selectionArgs)`  | 查询数据         |
   | `boolean onCreate()`                                    | 初始化           |
   | `int update(Uri,ContentValues,selection,selectionArgs)` | 更新数据         |
   | `int delete(Uri,selection,selectionArgs)`               | 删除已有数据     |
   | `String getType(Uri)`                                   | 获取数据MIME类型 |

   **Uri**:资源访问路径

   **projection**:查询列名

   **selection**: 查询条件

   **selectionArgs**: 查询条件占位符

4. 使用ContentResolver进行增删改查操作

   ```java
   context.getContentResolver().insert(uri, contentValues);
   ```

   通过ContentResolver调用增删改查时，会回调该uri对应的ContentProvider中对应的增删改查方法。

##### 三. Uri相关

​		URI为系统的每一个资源都赋予了名称。由前缀，标识，路径，记录ID四部分组成

- 前缀 ：默认的固定开头格式，值为`content://`

- 标识 ：或者是授权，唯一标识Provider,一般为包名

- 路径 ：要操作的数据库表名

- 记录ID ：如果URI中包含表示获取的记录ID,则返回该ID对应数据，否则返回全部

  ```java
  Uri uri = Uri.parse("content://com.wx.test/task")
  ```

##### 四. 数据监听

##### 五. 工作流程

见[Android开发艺术探索第九章#Activity](../../Android开发艺术探索/Android开发艺术探索第九章笔记.md#ContentProvider的工作过程)

##### 六. 原理机制

##### 七. 补充



