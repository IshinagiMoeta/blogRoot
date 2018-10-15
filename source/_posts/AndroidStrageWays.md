---
title: Android四种数据存储方式
date: 2018-09-30 15:31:55
tags: [Android]
categories: Android

---
android的四种数据存储方式   

- SharedPreference
- Sqlite
- 文件存储
- 网络存储

# 一、SharedPreference #
默认存储路径：/data/data/<PackageName>/shared_prefs  
操作模式：

- **MODE_PRIVATE**（默认）：只有当前的应用程序才能对文件进行读写。  
- **MODE_MULTI_PROCESS**：用于多个进程对同一个SharedPreferences进行读写。  

存储数据格式：键值对  
## 获取SharedPreferences对象的方法##
- Context的getSharedPreferences()方法，参数一是文件名，参数二是操作模式
- Activity的getPreferences()方法，参数为操作模式，使用当前应用程序包名为文件名
- PreferenceManager的getDefaultSharedPreferences()静态方法，接收Context参数，使用当前应用程序包名为文件名  

## 存储数据 ##
- 调用SharedPreferences对象的edit()方法获取一个SharedPreferences.Editor对象
- 向Editor对象中添加数据putBoolean、putString等
- 调用commit()方法提交数据

```java
SharedPreferences.Editor editor = getSharedPreferences("data",MODE_PRIVATE).edit();
editor.putString("name","ZhangSan");
editor.putInt("age",12);
editor.putBoolean("isMarried",false);
editor.commit();
```
## 从SharedPreferences文件中读取数据 ##
```java
SharedPreferences pref = getSharedPreferences("data",MODE_PRIVATE);
String name = pref.getString("name");
int age = pref.getInt("age");
boolean isMarried = pref.getBoolean("isMarried");
```

# 二、sqlite #

默认存储路径：/data/data/<PackageName>/databases
数据类型：

- integer 整型
- real 浮点型
- text 文本类型
- blob 二进制类型  

```java
public class MyDatabaseHelper extends SQLiteOpenHelper{  
    public static final String CREATE_BOOK = "create table book ( "
               + " id integer primary key autoincrement,"
               + " author text,"
               + "price real,"
               + "pages integer,"
               + "name text)"; 
    private Context context;
    public MyDatabaseHelper (Context context, String name, CursorFactory factory, int version) {  
        super(context, name, factory, version);  
        this.context = context;
    }  

    @Override  
    public void onCreate(SQLiteDatabase db) {  
        db.execSQL(CREATE_BOOK);          
    }  
      
    //当打开数据库时传入的版本号与当前的版本号不同时会调用该方法  
    @Override  
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {    
    
    }    
}  
```

调用
```java
MyDatabaseHelper helper = new MyDatabaseHelper(this,"BookStore.db",null,1);
 //检测到没有BookStore这个数据库，会创建该数据库并调用MyDatabaseHelper中的onCreated方法。
helper.getWritableDatabase(); 
```

## 升级数据库 ##
```java
public class MyDatabaseHelper extends SQLiteOpenHelper{  
    ......
    //当打开数据库时传入的版本号与当前的版本号不同时会调用该方法  
    @Override  
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {   
          db.execSQL("drop table if exists Book");
          onCreate(db):
    }    
} 
```

只需将version改为大于原来版本号即可。
```java
MyDatabaseHelper helper = new MyDatabaseHelper(this,"BookStore.db",null,2);
helper.getWritableDatabase(); 
```

## 向数据库添加数据 ##
insert()方法，参数一表名，参数二是在未指定添加数据的情况下给某些可为空的列自动赋值为NULL，设置为null即可，参数三是ContentValues对象。
```java
SQLiteDatabase db = helper.getWritableDatabase();
ContentValues values = new ContentValues();
values.put("name","The Book Name");
values.put("author","chen");
values.put("pages",100);
values.put("price",200);
db.insert("Book",null,values);
```

## 更新数据库中的数据 ##
update()方法，参数一是表名，参数二是ContentValues对象，参数三、四是去约束更新某一行或某几行的数据，不指定默认更新所有。
```java
SQLiteDatabase db = helper.getWritableDatabase();
ContentValues values = new ContentValues();
values.put("price",120);
db.update("Book",values,"name= ?",new String[]{"The Book Name"});
```

## 从数据库中删除数据 ##
delete()方法，参数一是表名，参数二、三是去约束删除某一行或某几行的数据，不指定默认删除所有。
```java
SQLiteDatabase db = helper.getWritableDatabase();
db.delete("Book","pages> ?",new String[]{"100"});
```

## 查询数据库中的数据 ##
query()方法，参数一是表名，参数二是指定查询哪几列，默认全部，参数三、四是去约束查询某一行或某几行的数据，不指定默认查询所有，参数五是用于指定需要去group by的列，参数六是对group by的数据进一步的过滤，参数七是查询结果的排序方式
```
SQLiteDatabase db = helper.getWritableDatabase();
Cursor cursor = db.query("Book",null,null,null,null,null,null);
if(cursor.moveToFirst()){
      do{
            String name = cursor.getString(cursor.getColumnIndex("name");
            String author = cursor.getString(cursor.getColumnIndex("author");
            int pages = cursor.getString(cursor.getColumnIndex("pages");
            double price = cursor.getString(cursor.getColumnIndex("price");
       }while(cursor.moveToNext());
}
cursor.close():
```

## 使用SQL语句操作数据库 ##
```java
//添加数据
db.execSQL("insert into Book(name,author,pages,price) values(?,?,?,?) "
            ,new String[]{"The Book Name","chen",100,20});
//更新数据
db.execSQL("update Book set price = ? where name = ?",new String[]
            {"10","The Book Name"});
//删除数据
db.execSQL("delete from Book where pages > ?",new String[]{"100"});
//查询数据
db.execSQL("select * from Book",null);
```

## 使用事务操作 ##
```java
SQLiteDatabase db = helper.getWritableDatabase();
db.beginTransaction();  //开启事务
try{
      ......
      db.insert("Book",null,values);
      db.setTransactionSuccessful();  //事务成功执行
}catch(SQLException e){
      e.printStackTrace();
}finally{
      db.endTransaction();  //结束事务
}
```

# 三、文件存储 #  
默认存储路径：/data/data/<PackageName>/files  
文件操作模式：MODE_PRIVATE（默认）：覆盖、MODE_APPEND：追加  

- 写入文件  

```java
public void save(){
      String data = "save something here";
      FileOutputStream out = null;
      ButteredWriter writer = null;
      try{
            out = openFileOutput("data",Context.MODE_PRIVATE);
            writer = new ButteredWriter(new OutputSreamWriter(out));
            writer.write(data);
      }catch(IOException e){
            e.printStackTrace();
      }finally{
            try{
                  if(writer!=null){
                        writer.close();
                  }
            }catch(IOException e){
                  e.printStackTrace();
       }
}
```

- 读取数据  

```java
public String load(){
      FileInputStream in = null;
      ButteredReader reader = null;
      StringBuilder builder = new StringBuilder();
      try{
            in = openFileInput("data");
            reader = new ButteredReader(new InputStreamReader(in));
            String line= "";
            while((line = reader.readline()) != null){
                   builder.append();
            }
      }catch(IOException e){
            e.printStackTrace();
      }finally{
            if(reader != null){
                    try{
                          reader.close();
                    }catch(IOException e){
                          e.printStackTrace();
                    }
             }
      }
}
```

# 四、网络存储 #
网络存储的本质和文件存储一样，是将需要存储的数据以文件的形式保存，不过网络储存存放的位置是服务端。由于网络存储又涉及到网络请求，所以在这里不详细说明，只做一个介绍。

# P.S. #
还有种说法是ContentProvider也是Android的存储方式之一。不过我们先来看看Google的官方介绍  

![](https://ws1.sinaimg.cn/large/6bbf23f6gy1fvrob98dclj20o00br75q.jpg)  
 
在谷歌文档中，ContentProvider是一个内容提供者，仅当你需要在多个应用之间共享数据的时候，才需要用到这个组件。  
所以博主认为ContentProvider与其说是存储方式，更接近一个提供不同应用间共享数据的接口。  
不过如果有同学在面试中遇到的话，可以根据面试官的说法来回答，如果面试官和你持有不同意见，你甚至可以拿出本文中的理论依据和他解释，甚至可以加分。


注:本文部分摘抄自简书，无意冒犯，侵权必删  
作者：cy_why  
链接：[https://www.jianshu.com/p/536ca489a7f4](https://www.jianshu.com/p/536ca489a7f4 "https://www.jianshu.com/p/536ca489a7f4")