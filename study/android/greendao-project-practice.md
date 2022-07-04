---
title: greenDAO项目实战
date: 2017-03-16 17:13:27
categories:
    - 开发效率
tags:
    - 安卓
    - GreenDAO
    - 数据库
    - 开发效率
---
>   在安卓移动APP开发过程中，一般使用安卓默认的sqlite数据库做数据持久化存储。虽然android提供了ContentProvider等框架支持，但是写SQL语句依然是一件不少程序猿的头疼的事。无论是简单的SQL语句，还是复杂的多表查询，都是影响开发效率的苦力活。幸运的是，我们有类似于Hibernate/MyBatis的ORM框架用于安卓开发中。今天我们介绍gitHub上热门的一个专注于Android开发的ORM框架——greenDAO.

## greenDAO简介 

greenDAO是一个轻量&快速的安卓ORM框架,把对象和Sqlite数据库关联映射起来。它专门为Android做了高度的优化，高性能，消耗极少的内存。
**greenDAO首页, 开发文档以及技术支持链接: http://greenrobot.org/greendao/**
![greendao图解](http://greenrobot.org/wordpress/wp-content/uploads/greenDAO-orm-320.png)
**GreenDAO特色功能**：
* 稳健：GreenDAO从2011开始，就被无数著名的应用程序使用 
* 超级简单：简洁直截了当的API，支持注释（V3以上版本） 
* 小巧：框架小于150K,并且只是普通的java jar（没有CPU相关的本地部分）减少构建时间避免超出 65k方法数限制 
* 快速：可能是全球最快的Android平台ORM框架，得益于gradle插件，框架能够自动智能执行代码生成 
* 安全且人性化的查询API：QueryBuilder使用属性常量，避免拼写错误（这是SQL语句拼接查询常见的错误） 
* 强大的连接：跨实体的查询，甚至嵌套复杂的多表链接和多条件关系查询 
* 灵活的属性类型：在实体中使用自定义的类或枚举表示数据 
* 加密：支持SQLCipher加密数据库
* 活跃的社区:超过7000 GitHub stars
## 在项目中使用GreenDAO
greenDAO 在 Maven 仓库可用. 请保证点击以下链接检查你使用的是否是最新版本[greenDAO最新版链接 ](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22org.greenrobot%22%20AND%20a%3A%22greendao%22)| [greenDAO-generator最新版链接](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22org.greenrobot%22%20AND%20a%3A%22greendao-generator%22)

添加以下的Gradle配置到安卓项目：
```groovy
// 在项目根目录的 build.gradle 文件:
buildscript {
    repositories {
        jcenter()
        mavenCentral() <-- 添加这个仓库
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.3.0'
        classpath 'org.greenrobot:greendao-gradle-plugin:3.2.1' <-- 添加greendao用到的gradle插件，用于生成GreenDao用到的代码（注意使用最新版本）
    }
}
 
// 在app 项目的 build.gradle 文件:
apply plugin: 'com.android.application'
apply plugin: 'org.greenrobot.greendao' <-- 应用greendao的gradle插件(用于生成GreenDao用到的代码)
//------------
//(可选)配置数据库版本，生成的Dao包路径(无特殊要求除了schemaVersion，daoPackage和targetGenDir一般不用配置，使用默认的)
greendao {
    schemaVersion 1
    daoPackage 'com.example.greendao'
    targetGenDir 'src/main/java'
}

//------------

dependencies {
    compile 'org.greenrobot:greendao:3.2.0' <-- 添加greenDao框架依赖
//如果需要数据库加密，要引入一下sqlcipher加密库
//compile 'net.zetetic:android-database-sqlcipher:3.5.4'
}
```
**注意** 这些配置使得GreenDao插件在Gradle构建过程中触发，当项目构建时.插件生成 DaoMaster, DaoSession 以及实体类对应的DAO.这些类是GreenDao框架的核心类

### 声明实体类
> 首先，我们声明一个实体类Note,一个自定义的枚举类型NoteType。

```java

public enum NoteType {
    TEXT, LIST, PICTURE
}

```

```java

@Entity
public class Note {

    @Id(autoincrement = true)
    private Long id;

    @NotNull
    private String text;
    private String comment;
    private java.util.Date date;
    
    @Convert(converter = NoteTypeConverter.class, columnType = String.class)
    private NoteType type;

    @Generated(hash = 1272611929)
    public Note() {
    }

    public Note(Long id) {
        this.id = id;
    }

    @Generated(hash = 1686394253)
    public Note(Long id, @NotNull String text, String comment, java.util.Date date, NoteType type) {
        this.id = id;
        this.text = text;
        this.comment = comment;
        this.date = date;
        this.type = type;
    }


    @NotNull
    public String getText() {
        return text;
    }

    /** 非空值声明，保证这个只在保存到数据之前是非空的**/
    public void setText(@NotNull String text) {
        this.text = text;
    }

    //省略了其他的geter,setter

}

```

上面的实体类代码中：

`@Entity`注解表明这是一个GreenDao映射到数据的实体类，括号内的配置是可选的。

`@Id`注解表明这是主键,`autoincrement = true`表示自增，**注意**GreenDao主键需要是Long类型，不能是long或者int之类的其他类型。（这也是安卓推荐的主键类型）

`@NotNull` 注解表明这是一个非空值，在它的setter参数也注明了非空，保证这个只在保存到数据之前是非空的。

`@Convert(converter = NoteTypeConverter.class, columnType = String.class)`这个注解实现自定义类型到数据库支持的类型的转换。`converter = NoteTypeConverter.class`指定转换器是NoteTypeConverter这个类（下面马上介绍），`columnType = String.class`指定数据库对应的列类型是文本类型。通过Convert我们可以用自然的方式表达各种属性。

`@Transient`上面的例子没有标出。这个注解表明此字段不存储到数据库中，用于不用持久化的字段

值得说明的是，实体类的构造函数和getter/setter是greendao在项目build的时候自动生成的，是不是很方便呢？

接下来我们再看看转换器的写法。转换器听起来很高级有没有？但是写起来出乎意料的简单。

```java

public class NoteTypeConverter implements PropertyConverter<NoteType, String> {
    @Override
    public NoteType convertToEntityProperty(String databaseValue) {
        return NoteType.valueOf(databaseValue);
    }

    @Override
    public String convertToDatabaseValue(NoteType entityProperty) {
        return entityProperty.name();
    }
}

```
每个转换器都实现了`PropertyConverter<P, D>` 这个接口。P是实体类中自定义的类型，D是数据库支持的类型。

接口方法`convertToEntityProperty`接受一个数据库支持类型的参数，返回实体类中自定义的类型。

接口方法`convertToDatabaseValue`接受一个实体类中自定义的类型的参数，返回数据库支持类型。

通过实现这两个方法，我们就完成了实体类的自定义类型到数据库支持类型的映射。在本例中，`NoteType`这个自定义的枚举类型得以用文本方式存储到数据库。


### 在Application中配置DaoSession
> 接着，我们在App中配置DaoSession，作为数据统一操作的对象，注意DaoSession所在的包如果未配置，默认跟声明的实体类包名相同

```java
public class App extends Application {
    /**是否加密标志。通过这个例子展示给我们greendao从用SQLCipher加密数据库到标准的sqlite切换时多么简单.打开数据库加密需要引入SQLCipher相关的库*/
    public static final boolean ENCRYPTED = false;
    /**全局公用的DaoSession**/
    private DaoSession daoSession;

    @Override
    public void onCreate() {
        super.onCreate();
        //DevOpenHelper简单的在数据库需要升级的时候重建数据表，在实际项目中，如果我们需要保存旧版本的数据，需要自己实现一个OpenHelper,把旧版本数据迁移到新版本中。
        
        DevOpenHelper helper = new DevOpenHelper(this, ENCRYPTED ? "notes-db-encrypted" : "notes-db");
        Database db = ENCRYPTED ? helper.getEncryptedWritableDb("super-secret") : helper.getWritableDb();
        daoSession = new DaoMaster(db).newSession();
    }

    public DaoSession getDaoSession() {
        return daoSession;
    }
}

```
** 提醒 **
记得在`AndroidManifest.xml`中配置App.
```xml
    <application
        android:name=".App">
    </application>
```
到这里，我们从实体类的声明，数据库操作入口都已经建立好了。没有写一行SQL，是不是有一种畅快淋漓的感觉？下面我们用几个代码片段展示一下如何通过greendao进行数据的增删改查。

### 查询所有的note列表
    
```java

        // 获得note对应的DAO。Dao在build的时候自动生成
        DaoSession daoSession = ((App) getApplication()).getDaoSession();
        //我们通过这个noteDao进行增删改查的操作
        NoteDao noteDao = daoSession.getNoteDao();
        
        //查询所有的note数据
        List<Note> notes = noteDao.loadAll();
        // 查询所有的note数据, 根据note的文本内容根据a-z升序排序.这里的NoteDao.Properties.Text是greendao自动生成的，上面提到的属性常量。 
        notes = noteDao.queryBuilder().orderAsc(NoteDao.Properties.Text).build().list();

```

### 根据id来查询一个note记录,修改后再保存到数据库里
    
```java

        Long id = 1L;
        Note note = noteDao.load(id);
        
        note.setText("这是修改后的内容文本");
        note.setComment("这是修改后的comment");
        noteDao.save(note);
```

### 删除所有note记录
```java
        noteDao.deleteAll();
```
### 根据id删除一条note记录
```java
        Long id = 1;
        noteDao.deleteByKey(id);
```
### 添加一条note记录

```java
        Note note = new Note();
        note.setText("这是note 内容文本");
        note.setComment("这是note的comment");
        note.setDate(new Date());
        note.setType(NoteType.TEXT);
        noteDao.insert(note);
```

## 后记

greenDao的简单介绍就到这里。后续介绍GreenDao的进阶使用，包括`@ToOne` 一对一， `@ToMany` 一对多等使用介绍。同时介绍如何在现有项目中迁移到GreenDao，包括用统一的数据库管理类来对GreenDao进行轻度封装等。