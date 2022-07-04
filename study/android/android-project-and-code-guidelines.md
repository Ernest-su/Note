---
title: GitHub上star超过2k的安卓项目和代码风格指南（绝对干货）
date: 2017-03-28 19:06:55
categories:
    - 最佳实践
tags:
    - 代码风格
    - 命名规范
    - Android
    - 安卓
---
> 近日在搜MVP模式作为安卓项目架构时候，发现GitHub上一篇介绍MVP架构的文章。一看star已经超过2K了。作者同项目的这篇文章讲安卓项目和代码风格指南，也非常有参考价值。为了方便大家查阅，把原文翻译了一下。点击查看作者介绍安卓MVP架构指南的[翻译文章](http://www.suqishuo.cn/android-architecture-guideline-for-mvp/)。

# 1. 项目指南

## 1.1 项目结构

新项目应该遵循安卓Gradle项目结构，定义在这里 ：[安卓Gradle插件用户指南](http://tools.android.com/tech-docs/new-build-system/user-guide#TOC-Project-Structure)。项目[ribot Boilerplate](https://github.com/ribot/android-boilerplate) 是一个很好的参考。

## 1.2 文件命名

### 1.2.1 类文件
类名书写方式是 [大写的驼峰命名](http://en.wikipedia.org/wiki/CamelCase).

继承于安卓组件的类，名称应该以继承的组件命名，比如：`SignInActivity`, `SignInFragment`, `ImageUploaderService`, `ChangePasswordDialog`.

### 1.2.2 资源文件

资源文件书写方式：小写_下划线。如：ic_launcher.png
#### 1.2.2.1 Drawable 文件

drawable类型资源命名：

| 资源类别   | 前缀            |		例子               |
|--------------| ------------------|-----------------------------|
| Action bar   | `ab_`             | `ab_stacked.9.png`          |
| Button       | `btn_`	            | `btn_send_pressed.9.png`    |
| Dialog       | `dialog_`         | `dialog_top.9.png`          |
| Divider      | `divider_`        | `divider_horizontal.9.png`  |
| Icon         | `ic_`	            | `ic_star.png`               |
| Menu         | `menu_	`           | `menu_submenu_bg.9.png`     |
| Notification | `notification_`	| `notification_bg.9.png`     |
| Tabs         | `tab_`            | `tab_pressed.9.png`         |

图标（icons）命名方式 (取自 [安卓图标官方指南](http://developer.android.com/design/style/iconography.html)):

| 资源类别                     | 前缀             | 例子                      |
| --------------------------------| ----------------   | ---------------------------- |
| Icons                           | `ic_`              | `ic_star.png`                |
| Launcher icons                  | `ic_launcher`      | `ic_launcher_calendar.png`   |
| Menu icons and Action Bar icons | `ic_menu`          | `ic_menu_archive.png`        |
| Status bar icons               | `ic_stat_notify`   | `ic_stat_notify_msg.png`     |
| Tab icons                       | `ic_tab`           | `ic_tab_recent.png`          |
| Dialog icons                    | `ic_dialog`        | `ic_dialog_info.png`         |

selector 状态命名方式:

| 状态	       | 后缀          | 例子                     |
|--------------|-----------------|-----------------------------|
| Normal (正常)      | `_normal`       | `btn_order_normal.9.png`    |
| Pressed（按下）      | `_pressed`      | `btn_order_pressed.9.png`   |
| Focused （获得焦点）     | `_focused`      | `btn_order_focused.9.png`   |
| Disabled    （不可用） | `_disabled`     | `btn_order_disabled.9.png`  |
| Selected    （选中） | `_selected`     | `btn_order_selected.9.png`  |


#### 1.2.2.2 布局文件


布局文件应该跟对应的安卓组件名称匹配，但是把最顶级的组件名称放在前面。比如，当我们创建一个用于 `SignInActivity`的布局，那布局文件名称应该是`activity_sign_in.xml`.

| 组件        | 类名             | 布局文件名                  |
| ---------------- | ---------------------- | ----------------------------- |
| Activity         | `UserProfileActivity`  | `activity_user_profile.xml`   |
| Fragment         | `SignUpFragment`       | `fragment_sign_up.xml`        |
| Dialog           | `ChangePasswordDialog` | `dialog_change_password.xml`  |
| AdapterView item | ---                    | `item_person.xml`             |
| Partial layout   | ---                    | `partial_stats_bar.xml`       |

一个稍微不同的地方是当我们创建用于`Adapter`渲染的布局,如填充一个 `ListView`， 在这种情况下，布局文件命名应该是用 `item_`开始。
注意有些地方这些规则没法应用，如，当我们创建一个布局文件，用于渲染另一个布局的一部分，这样的情况下，我们应该使用`partial_`前缀。

#### 1.2.2.3 菜单文件
跟布局文件相似，按钮文件应该跟对应的组件匹配。比如：当我们定义一个菜单，用于 `UserActivity`,那么文件名应该是 `activity_user.xml`

文件名不含有`menu`是个良好的习惯做法，因为这些文件已经位于 `menu` 目录了。

#### 1.2.2.4 取值文件(Values files)
在values文件夹的资源文件应该是复数形式（plural），比如，`strings.xml`, `styles.xml`, `colors.xml`, `dimens.xml`, `attrs.xml`

# 2 代码指南

## 2.1 Java语言规范

### 2.1.1 不要忽略了异常

你万不该这样做:

```java
void setServerPort(String value) {
    try {
        serverPort = Integer.parseInt(value);
    } catch (NumberFormatException e) { }
}
```
虽然你可能觉得你的代码永远不会抛出这个异常或者这异常不重要，不需要处理它，像上面那样忽略异常会在你的代码中留下雷区，将来其他人会踩到。你必须在你的代码中以某种原则的方式处理每个异常。具体的处理视情况而定。._ - ([安卓官方代码指南](https://source.android.com/source/code-style.html))

指南中关于不要忽略异常的说明在 [这里](https://source.android.com/source/code-style.html#dont-ignore-exceptions).

### 2.1.2 不要捕获一般的异常。

你不应该这样做:

```java
try {
    someComplicatedIOFunction();        // 可能抛出 IOException
    someComplicatedParsingFunction();   // 可能抛出 ParsingException
    someComplicatedSecurityFunction();  // 可能抛出 SecurityException
    //啊哈，统一处理掉
} catch (Exception e) {                 //我将捕获所有的异常
    handleError();                      // 使用一个通用的处理手段!
}
```

查看不这样做的原因以及要怎么做： [猛戳安卓官方指南之不要捕获一般的异常](https://source.android.com/source/code-style.html#dont-catch-generic-exception)

### 2.1.3 不要用finalizer

我们不使用finalizer。它什么时候被调用是没有保证的，甚至它不被调用。大部分情况下，你可以使用良好的异常处理来满足需要在finalizer中做的工作。如果你确实需要它，定义一个 `close()` 方法（或类似的）然后在文档中明确的指出什么时候这个方法需要被调用。可以查看 `InputStream` 做为例子.这种情况下，在finalizer中打印一个简短的log信息是恰当的但不是必须的，因为我们不希望log泛滥 ._ - ([猛戳安卓官方指南之不要用finalizer](https://source.android.com/source/code-style.html#dont-use-finalizers))


### 2.1.4 完全的import语句

反例: `import foo.*;`

正解: `import foo.Bar;`

点击 [这里](https://source.android.com/source/code-style.html#fully-qualify-imports) 查看更多说明

## 2.2 Java风格规范

### 2.2.1 全局变量定义及命名

全局变量应该定义在**文件的头部**,遵循下面的命名规则。

* 私有的非静态全局变量以`m`开头
* 私有的静态全局变量以`s`开头
* 其他的全局变量以小写单词开头
* 常量使用全部大写，单词间用下划线间隔（ALL_CAPS_WITH_UNDERSCORES）.
> 译者注：这个规则取自Android Open Source Project代码贡献规范。在程序员的圈子里对于要不要前缀这个问题已经吵翻天了。我在使用greendao的时候使用前缀的话，自动生成的getter/setter会是getMxxx/setMxxx这样难看的方法。很多人也觉得这个命名规则没有意义。[stackOverFlow](http://stackoverflow.com/questions/2092098/why-do-most-fields-class-members-in-android-tutorial-start-with-m) 还有 [这里](http://stackoverflow.com/questions/4237469/why-do-variable-names-often-start-with-the-letter-m) ，以及一本教你怎么写出整洁代码的书中如是说：
 "我觉得如今这些前缀没有意义，尤其是在你的APP中!你的类和方法应该尽量的小，而且你应该使用代码高亮的编辑环境，使得成员变量易于分辨。再者，人们快速适应忽略了前缀或后缀来看名字的有意义部分。我们阅读代码越多，看到前后缀越少。渐渐地，前缀就变成了旧代码看不到的线索和标记。——代码整洁之道(Clean.Code).Robert.C.Martin"

例子:

```java
public class MyClass {
    public static final int SOME_CONSTANT = 42;
    public int publicField;
    private static MyClass sSingleton;
    int mPackagePrivate;
    private int mPrivate;
    protected int mProtected;
}
```

### 2.2.3把首字母缩写当做一个单词来看

| 好的例子           | 不好的例子            |
| -------------- | -------------- |
| `XmlHttpRequest` | `XMLHTTPRequest` |
| `getCustomerId`  | `getCustomerID`  |
| `String url`     | `String URL`     |
| `long id`        | `long ID`        |

### 2.2.4 使用空格缩进

使用四个空格做代码块缩进:

```java
if (x == 1) {
    x++;
}
```

换行时使用8个空格缩进:

```java
Instrument i =
        someLongExpression(that, wouldNotFit, on, one, line);
```

### 2.2.5 使用标准的大括号风格

大括号的开始跟代码同一行
```java
class MyClass {
    int func() {
        if (something) {
            // ...
        } else if (somethingElse) {
            // ...
        } else {
            // ...
        }
    }
}
```


条件语句使用大括号包住，除非条件体只有一行。
如果条件及只有一行条件体，而且没被换行，大括号是不必的。
```java
if (condition) body();
```

这是**不好的**:

```java
if (condition)
    body();  // 不好的，没有大括号包围!
```

### 2.2.6 注解

#### 2.2.6.1 注解实践规范

根据安卓代码风格指南，一些Java内置的注解标准的实践规范有：
* `@Override`:  无论一个方法是重载父类还是实现某个接口的都**必须使用**@Override 注解。比如，当你使用 @inheritdocs Javadoc 标签,然后 从一个类(不是一个接口)中导出，你也必须声明那个方法 @Overrides 父类的方法。

* `@SuppressWarnings`: @SuppressWarnings 注解应该仅当不可能消除一个警告的时候才使用。如果一个警告 通过这个 "不可能消除" 的测试,  @SuppressWarnings 注解必不可少, 以便保证所有的警告在代码中反映实际的问题。

关于注解的更多指南可以查看[这里](http://source.android.com/source/code-style.html#use-standard-java-annotations).

#### 2.2.6.2 注解风格

**类，方法及构造器**

当注解被应用到一个类，方法或者构造函数时候，注解在代码注释之后，**一行一个**注解
```java
/* 这是这个类的注解 */
@AnnotationA
@AnnotationB
public class MyAnnotatedClass { }
```

**全局变量**

全局变量的注解应该在**同一行**列出。除非这一行超过了最大字数了。

```java
@Nullable @Mock DataManager mDataManager;
```

### 2.2.7 限制变量范围

变量的使用范围应该限制到最小(Effective Java 条目 29)。这样做，你可以为你的代码增加可读性和可维护性，减少可能的错误。每个变量应该在最内层的使用到的它代码块中定义


局部变量应该在第一次用到他们的时候定义。几乎每个局部变量都应该初始化。如果你没有足够的信息去初始化一个变量，你应该延迟声明这个变量，直到你有足够的初始化信息。._ - ([安卓代码风格指南](https://source.android.com/source/code-style.html#limit-variable-scope))

### 2.2.8 排序import语句

如果你使用IDE,如Android Studio，你不必担心这些规则，这些IDE已经遵循这些规则。如果你不用IDE，往下看这些规则:

1.  import 安卓框架的
2. Import 来自第三方的(com, junit, net, org)
3. java 以及  javax
4. 同一个项目的 imports

为了保持跟IDE的设置一致，这些imports应该是这样的：
* 首字母分组排序，大写的字母在小写的字母前面（如Z在a前面）。
* 每个主要分组 (android, com, junit, net, org, java, javax)间应该有一行空行。

更多信息查看[这里](https://source.android.com/source/code-style.html#limit-variable-scope)

### 2.2.9 日志(Loging)指南


使用 `Log`  类提供的日志方法来打印错误信息或对开发者分辨问题有用的其他信息：

* `Log.v(String tag, String msg)` (verbose)
* `Log.d(String tag, String msg)` (debug)
* `Log.i(String tag, String msg)` (information)
* `Log.w(String tag, String msg)` (warning)
* `Log.e(String tag, String msg)` (error)


通常来说，我们使用一个类的名称作为一个TAG，在文件的开头定义成一个 `static final` 变量，如：

```java
public class MyClass {
    private static final String TAG = MyClass.class.getSimpleName();

    public myMethod() {
        Log.e(TAG, "My error message");
    }
}
```
VERBOSE 和 DEBUG 日志 **必须**在release版本中禁用。同时也建议禁用INFORMATION, WARNING 和 ERROR 日志，但是你可能觉得它们在release版本中定位问题很有用而保持启用。如果你决定让他们保持启用，你要保证这些日志信息不会泄漏email地址，用户id之类的隐私信息。
仅限debug版本显示Log的配置：
```java
if (BuildConfig.DEBUG) Log.d(TAG, "The value of x is " + x);
```

### 2.2.10 类成员排序

这虽然没有简单正确的方案，但是使用一个 **逻辑的** 及 ** 一致的 ** 顺序可以提高代码的可读性和可学习性。下面是推荐的顺序：

1. 常量
2. 全局变量
3. 构造函数
4. 重载的函数和回调 (public 或 private)
5. Public 方法
6. Private 方法
7. 内部类或内部接口

例子:

```java
public class MainActivity extends Activity {

	private String mTitle;
    private TextView mTextViewTitle;

    public void setTitle(String title) {
    	mTitle = title;
    }

    @Override
    public void onCreate() {
        ...
    }

    private void setUpView() {
        ...
    }

    static class AnInnerClass {

    }

}
```

如果你的类继承一个**安卓组件**如Activity或Fragment，对重载的方法进行排序以便**跟组件的生命周期匹配**是一个最佳实践。比如，当你的Activity实现`onCreate()`, `onDestroy()`, `onPause()` 和 `onResume()`，正确的顺序是：
```java
public class MainActivity extends Activity {

	//顺序跟Activity的生命周期匹配
    @Override
    public void onCreate() {}

    @Override
    public void onResume() {}

    @Override
    public void onPause() {}

    @Override
    public void onDestroy() {}

}
```

### 2.2.11 方法的参数顺序

在安卓编码时候，定义一个方法拥有一个`Context`参数是非常常见的，如果你写一个这样的方法，**Context**应该是第一个参数。

相反的例子是**回调**接口，它应该是**最后**一个参数
例子:

```java
// Context 总是在第一个位
public User loadUser(Context context, int userId);

// 回调总是在最后一位
public void loadUserAsync(Context context, int userId, UserCallback callback);
```

### 2.2.13 字符串常量，命名和取值

很多安卓SDK的元素如`SharedPreferences`, `Bundle`, 或 `Intent`使用键值对实现，所以即使是一个小应用，使用一堆字符串常量也是很常见的。

当使用这些组件，你**必须**定义这些键是 `static final` 变量，并且它们应该像下面这样使用前缀

| 元素            | 变量名前缀|
| -----------------  | ----------------- |
| SharedPreferences  | `PREF_`             |
| Bundle             | `BUNDLE_`           |
| Fragment Arguments | `ARGUMENT_`         |
| Intent Extra       | `EXTRA_`            |
| Intent Action      | `ACTION_`           |


注意Fragment的参数——`Fragment.getArguments()`也是一个Bundle。然而，因为这是Bundle非常常见的用法，我们为它定义一个不同的前缀。

例子:

```java
//注意这些变量的值应该和名称一致来避免问题
static final String PREF_EMAIL = "PREF_EMAIL";
static final String BUNDLE_AGE = "BUNDLE_AGE";
static final String ARGUMENT_USER_ID = "ARGUMENT_USER_ID";

//Intent相关的使用完全包名作为值
static final String EXTRA_SURNAME = "com.myapp.extras.EXTRA_SURNAME";
static final String ACTION_OPEN_USER = "com.myapp.action.ACTION_OPEN_USER";
```

### 2.2.14  Fragment和Activity的参数

当数据通过`Intent` 或`Bundle`传递到一个`Activity `或 `Fragment`，这些不同值的键**必须** 遵循上面描述的规则。

当一个 `Activity` 或 `Fragment`  想要参数，它应该提供一个`public static`方法来简化对应的`Intent` 或 `Fragment`创建
在Activity中这个方法通常叫 `getStartIntent()`:

```java
public static Intent getStartIntent(Context context, User user) {
	Intent intent = new Intent(context, ThisActivity.class);
	intent.putParcelableExtra(EXTRA_USER, user);
	return intent;
}
```

对于Fragment它的名称是`newInstance()`，处理使用合适的参数创建Fragment。

```java
public static UserFragment newInstance(User user) {
	UserFragment fragment = new UserFragment;
	Bundle args = new Bundle();
	args.putParcelable(ARGUMENT_USER, user);
	fragment.setArguments(args)
	return fragment;
}
```

**注意1**: 这些方法应该在类的 `onCreate()`前面

**注意2**: 如果我们提供上面说的方法，这些extras对应的键和参数应该是`private` 的，因为他们不需要暴露到类以外。

### 2.2.15 代码行长度限制

代码行不应该超过**100 个字符**。如果代码行超过这个限制长度，通常有两种方式来降低长度：

* 提取一个局部变量或方法（推荐方式）.
* 把单行换行成多行。

有两种**例外**可以让一行超过100字符：
* 这行不可分割，如长URL。
* `package` 和 `import` 语句.

#### 2.2.15.1 换行策略


没有精确的公式解释怎么换行，很多不同的方案是有效的。然而，这里有几个规则可以应用到常见情况。
**在操作符前断开**

当一行被操作符断开，断开处应该在操作符**之前**，例子如下：

```java
int longName = anotherVeryLongVariable + anEvenLongerOne - thisRidiculousLongOne
        + theFinalOne;
```

**赋值符号（=）例外**

在操作符前断开有一个例外，那就是赋值符号  `=`，应该在赋值符号**后面**断开。

```java
int longName =
        anotherVeryLongVariable + anEvenLongerOne - thisRidiculousLongOne + theFinalOne;
```

**方法链的情况**

当多个方法被链接在同一行的时候——如使用Builder时，每一个方法的调用应该在独立一行，在 `.`之前断开。

```java
Picasso.with(context).load("http://ribot.co.uk/images/sexyjoe.jpg").into(imageView);
```

```java
Picasso.with(context)
        .load("http://ribot.co.uk/images/sexyjoe.jpg")
        .into(imageView);
```

**长参数的情况**

当一个方法有很多参数或它的参数非常长，我们应该在每个逗号 `,`后面断开

```java
loadPicture(context, "http://ribot.co.uk/images/sexyjoe.jpg", mImageViewProfilePicture, clickListener, "Title of the picture");
```

```java
loadPicture(context,
        "http://ribot.co.uk/images/sexyjoe.jpg",
        mImageViewProfilePicture,
        clickListener,
        "Title of the picture");
```

### 2.2.16 RxJava 链式风格

Rx链式操作符要求换行。每一个操作符必须在新的一行，断行应该在`.`之前

```java
public Observable<Location> syncLocations() {
    return mDatabaseHelper.getAllLocations()
            .concatMap(new Func1<Location, Observable<? extends Location>>() {
                @Override
                 public Observable<? extends Location> call(Location location) {
                     return mRetrofitService.getLocation(location.id);
                 }
            })
            .retry(new Func2<Integer, Throwable, Boolean>() {
                 @Override
                 public Boolean call(Integer numRetries, Throwable throwable) {
                     return throwable instanceof RetrofitError;
                 }
            });
}
```

## 2.3 XML 风格规定

### 2.3.1 使用自关闭的标签

当一个XML元素没有任何内容时，你**必须**使用自关闭标签。
这是好例子:

```xml
<TextView
	android:id="@+id/text_view_profile"
	android:layout_width="wrap_content"
	android:layout_height="wrap_content" />
```

这是**不好的**例子 :

```xml
<!-- 不要这样做! -->
<TextView
    android:id="@+id/text_view_profile"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content" >
</TextView>
```


### 2.3.2 资源命名

资源 ID 和名称使用小写加下划线方式 **lowercase_underscore**.

#### 2.3.2.1 ID 命名

ID应该使用小写加下划线命名，加上元素名称为前缀，如：


| 元素            | 前缀            |
| -----------------  | ----------------- |
| `TextView`           | `text_`             |
| `ImageView`          | `image_`            |
| `Button`             | `button_`           |
| `Menu`               | `menu_`             |

Image view 例子:

```xml
<ImageView
    android:id="@+id/image_profile"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content" />
```

Menu 例子:

```xml
<menu>
	<item
        android:id="@+id/menu_done"
        android:title="Done" />
</menu>
```

#### 2.3.2.2 字符串

字符串名称以它们所属的部分为前缀命名，比如`registration_email_hint` 或 `registration_name_hint`。如果一个字符串**不属于**任何部分，那么你应该遵循下面的规则：

| 前缀             | 描述                           |
| -----------------  | --------------------------------------|
| `error_`             | An error message                      |
| `msg_`               | A regular information message         |
| `title_`             | A title, i.e. a dialog title          |
| `action_`            | An action such as "Save" or "Create"  |



#### 2.3.2.3 Styles 和 Themes

不像其他的资源，style命名方式是大写的驼峰是命名**UpperCamelCase**

### 2.3.3 属性顺序

作为通用的规则，你应该把相似的属性放在一起分组，一个对常见属性良好的排序方式是

1. View Id
2. Style
3. 布局的宽和高
4. 其他的布局属性，按字母排序
5. 剩下的属性，按字母排序

## 2.4 测试风格规定

### 2.4.1单元测试

测试类应该跟需要测试的类名字对应，以`Test`做后缀。比如我们创建一个测试类包含对`DatabaseHelper`的测试，我们应该把它命名为 `DatabaseHelperTest`。

测试方法用`@Test`注解，通常以要测试的方法名为命名开始，以测试的前置条件及/或期望结果做后缀。
* 模板: `@Test void methodNamePreconditionExpectedBehaviour()`
* 例子: `@Test void signInWithEmptyEmailFails()`

如果没有它们测试也能够清楚的表达，前置条件和/或期望结果不总是必须的。

有时候一个类可能包含一大堆方法，同时每个方法要求多个测试，在这种情况，推荐把测试类分割成几个。比如，如果 `DataManager`包含一大堆方法，我们可能想把它分到 `DataManagerSignInTest`, `DataManagerLoadUsersTest`等等。通常你能够知道哪些测试应该在一起，因为它们有共同的[test fixtures](https://en.wikipedia.org/wiki/Test_fixture).
### 2.4.2 Espresso 测试

每一个 Espresso 测试类目标通常是一个Activity，所以它的名称应该跟目标Activity的名称匹配，以`Test`结束，如 `SignInActivityTest`

当使用 Espresso API ，把链式方法调用放新行是一个最佳实践。

```java
onView(withId(R.id.view))
        .perform(scrollTo())
        .check(matches(isDisplayed()))
```

[原文链接](https://github.com/ribot/android-guidelines/blob/master/project_and_code_guidelines.md)
[译文链接](http://www.suqishuo.cn/android-project-and-code-guidelines/)