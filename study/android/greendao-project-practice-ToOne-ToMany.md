---
title: greenDAO项目实战-ToOne-ToMany实现一对一，一对多关联映射
date: 2017-03-20 11:14:27
categories:
    - 开发效率
tags:
    - 安卓
    - GreenDAO
    - 数据库
    - 开发效率
---
>   在上一篇文章[greenDAO项目实战](http://www.suqishuo.cn/greendao-project-practice/)中，我们对GreenDao有了初步的认识。今天我们熟悉一下GreenDao中使用`ToOne`和`ToMany`实现一对一，一对多关联映射。本文在上篇的基础上进行。


## 添加Category类

我们现在对上一篇的Note进行扩展，添加类别。我们假设一个Note属于一个类别Category，一个类别Category可以拥有多个Note.

```java
@Entity
public class Category {
    @Id
    private Long id;
    private String name;
    @ToMany(referencedJoinProperty = "categoryId")
    private List<Note> notes;
```
## 修改Note类，Category信息
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

    private Long categoryId;

    @ToOne(joinProperty = "categoryId")
    private Category category;
    //省略了其他的geter,setter
}

```
## 注解分析

上面的Category类代码中：
`@ToMany(referencedJoinProperty = "categoryId")`这个注解表明，一个`Category`可以拥有多个`Note`（一对多）.`referencedJoinProperty= "categoryId"`表示使用`Note`类中的`categoryId`字段进行关联。注意这个字段应该是Long类型.在Category中用` List<Note> notes`表示拥有的Note.

上面的Note类代码中：
`@ToOne(joinProperty = "categoryId")`这个注解表明，一个`Note`属于一个`Category`。注意我们添加了`private Long categoryId`这字段，然后在`@ToOne`注解中使用`joinProperty = "categoryId"`把`Category`用`categoryId`关联起来。

经过在一的端（Category）使用`@ToMany`，在多的端（Note）使用`@ToOne`,我们就把一对多和多对一关联关系建立起来了。

## 使用
```java
       //添加一个类别:技术分享.
        Category category = new Category(1L,"技术分享");
        categoryDao.insert(category);

        //添加一个类别:心灵鸡汤.
        category = new Category(2L,"心灵鸡汤");
        categoryDao.insert(category);
        
        //添加一条技术分享note记录
        Note note = new Note();
        note.setText("这是note内容文本，技术分享");
        note.setComment("这是note的comment");
        note.setDate(new Date());
        note.setType(NoteType.TEXT);
        note.setCategoryId(1L);//设置分类为技术分类
        noteDao.insert(note);
        
        //再添加一条技术分享note记录
        note = new Note();
        note.setText("这是note内容文本，也是技术分享");
        note.setComment("这是note的comment");
        note.setDate(new Date());
        note.setType(NoteType.TEXT);
        note.setCategoryId(1L);//设置分类为技术分类
        noteDao.insert(note);

        //接着添加一条属于心灵鸡汤Note记录
        note = new Note();
        note.setText("这是note内容文本，心灵鸡汤");
        note.setComment("这是note的comment");
        note.setDate(new Date());
        note.setType(NoteType.TEXT);
        note.setCategoryId(2L);//设置分类为心灵鸡汤
        noteDao.insert(note);

        //查询所有的分类为技术分享（id=1）的note数据
        List<Note> notes = categoryDao.load(1L).getNotes();
        System.out.println(notes);
        //通过Note反查Category
        noteDao.load(1L).getCategory().getName();     
        
        //顺便演示一下通过对文本内容的模糊查询匹配返回包含"心灵鸡汤"文本的Note。
        notes = noteDao.queryBuilder().where(NoteDao.Properties.Text.like("%心灵鸡汤%")).list();
        System.out.println(notes);
```

