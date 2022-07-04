---
title: Evernote带来的计划任务神器Android-Job
date: 2017-03-23 15:56:55
categories:
    - 开源框架
tags:
    - Evernote
    - JobScheduler
    - GcmNetworkManager
    - AlarmManager
    - Android
    - 安卓
    - Android-Job
---

> 在新版本安卓版本(API23以上)中，谷歌把`android.net.conn.CONNECTIVITY_CHANGE` 这个广播废弃了。总的来说，应用不应该依赖这个广播来执行相关任务，而应该使用 `JobScheduler` 或者 ` GCMNetworkManager`。计划任务的执行方案还可以用 `AlarmManager`。 对于国内的开发者来说，这是一个悲剧。Android 现在提供这三种不同的API来执行计划任务.这三种API都有各自的优缺点 。 `android.net.conn.CONNECTIVITY_CHANGE` 在高版本的系统中不可用了，`JobScheduler` 在低版本的系统中又不可用，` GCMNetworkManager` 是什么？在国内不存在。而且这些API之间的不同语义和用法对开发者是一种额外的负担。这种负担体现在我们不仅仅要懂得什么时候使用合适的API,而且要为不同的环境执行同样任务提供不同分支的代码。APP需要在运行时候用合适的条件判断到底要使用哪种方式执行计划任务。幸运的是，开源的世界有伟大的公司为我们提供了灵活的轮子。

Evernote 开源的 `Android-Job`  为我们带来兼容这三种API的方案，高效，简单，灵活。`Android-Job` 在运行判断使用哪种API，它提供  `AlarmManager`, `JobScheduler`和 `GcmNetworkManager`功能的超集，比如说，我们可以定义计划任务在网络连通且在充电时候执行。

# 为兼容而生

上面提到的三种方案，兼容维护非常麻烦。为了彰显`Android-Job`的价值，我们有必要对这三种方案做一个分析。

## AlarmManager


`AlarmManager` 的API[一直在改变](https://plus.google.com/+AndroidDevelopers/posts/GdNrQciPwqo) ，所以我们需要非常注意我们APP目标运行系统是哪个版本。

** 优点 **

- 所有设备上可用
- 发送广播来启动一个服务执行计划很容易

** 缺点 **

- API在不同系统版本表现不一致
- 太多样板代码
- 忽略了设备的状态
 ![alarmManager](/images/alarmManager.png)

## JobScheduler 

[JobScheduler ](https://developer.android.com/reference/android/app/job/JobScheduler.html)只在Lollipop以上版本可用。

** 优点 **

- 流式API简洁易用
- 检查设备状态

**  缺点 **

- 只在API 21+ 可用 （有些功能要求API24+）
- 平台bug
- 也是太多样板代码

## GcmNetworkManager 

[GcmNetworkManager ](https://developers.google.com/cloud-messaging/network-manager)只有在安装有Google Play的设备上可用，而且，没有被墙。

** 优点 **

- 跟`JobScheduler`相似的API
- 要求minSdkVersion 9

** 缺点 **

- 是 GooglePlay ServicesSDK 的一部分
- 不能脱离 Play Services 运行
- API 有很多陷阱

## 例子

Android-Job项目作者在一篇PPT中详细的说明了这三种方案以及Android-Job的用法。由于篇幅有限且排版原因，强烈建议点击链接去看看。例子简单易懂，在这里给出PPT对应的PDF文件。还在犹豫是否要使用 Android-Job ，看了这个PDF，相信你心里就有肯定的答案了 [PDF传送门](https://speakerd.s3.amazonaws.com/presentations/c9e9289df7df48bab2bf52db0a087248/Schedule_background_jobs_at_the_right_time_-_MTC.pdf)

# 使用Android-Job(翻译自项目说明)

请保证点击以下链接检查你使用的是否是最新版本[Android-Job最新版链接 ](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.evernote%22%20AND%20a%3A%22android-job%22)
添加以下的Gradle配置到安卓项目：

```groovy
// 在app 项目的 build.gradle 文件:
dependencies {
    compile 'com.evernote:android-job:1.1.7'
}
```

如果没有在 Gradle build tools 中关闭 manifest merger，那就不用更多步骤来配置这个工具库了。否则需要手动添加权限生命和services，参考这个[AndroidManifest](https://github.com/evernote/android-job/blob/master/library/src/main/AndroidManifest.xml)

###  用法

 `JobManager` 类作为入口。我们自定义的Jobs 需要继承  `Job` 类。使用对应的Builder类创建一个  `JobRequest` 然后使用 `JobManager` 创建这个计划请求。

使用 `JobManager` 之前必须向初始化这个单例。我们需要提供一个 `Context` 然后添加一个 `JobCreator` 实现类.  `JobCreator` 把一个job tag 映射到一个特定的job 类。推荐在 `Application` 的 `onCreate()` 方法中 初始化 `JobManager` 。如果没有 `Application` 类的访问权限，这是[可选的方案](FAQ.md#i-cannot-override-the-application-class-how-can-i-add-my-jobcreator)。

```java
public class App extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        JobManager.create(this).addJobCreator(new DemoJobCreator());
    }
}
```

```java
public class DemoJobCreator implements JobCreator {

    @Override
    public Job create(String tag) {
        switch (tag) {
            case DemoSyncJob.TAG:
                return new DemoSyncJob();
            default:
                return null;
        }
    }
}
```

然后我们可以开始计划要执行的任务。

```java
public class DemoSyncJob extends Job {

    public static final String TAG = "job_demo_tag";

    @Override
    @NonNull
    protected Result onRunJob(Params params) {
        // run your job here
        return Result.SUCCESS;
    }

    public static void scheduleJob() {
        new JobRequest.Builder(DemoSyncJob.TAG)
                .setExecutionWindow(30_000L, 40_000L)
                .build()
                .schedule();
    }
}
```

### 进阶用法

`JobRequest.Builder` 类有很多额外的配置选项，例如，我们可以声明条件：需要网络连接。为了使任务周期性执行，用 bundle 传递额外的参数，使得任务在重启后或在特定的时间恢复。 
每一个Job 有一个唯一的 ID 。这个 ID 帮助我们辨别 job ，以便将来更新条件或者取消 job。

```java
private void scheduleAdvancedJob() {
    PersistableBundleCompat extras = new PersistableBundleCompat();
    extras.putString("key", "Hello world");

    int jobId = new JobRequest.Builder(DemoSyncJob.TAG)
            .setExecutionWindow(30_000L, 40_000L)
            .setBackoffCriteria(5_000L, JobRequest.BackoffPolicy.EXPONENTIAL)
            .setRequiresCharging(true)
            .setRequiresDeviceIdle(false)
            .setRequiredNetworkType(JobRequest.NetworkType.CONNECTED)
            .setExtras(extras)
            .setRequirementsEnforced(true)
            .setPersisted(true)
            .setUpdateCurrent(true)
            .build()
            .schedule();
}

private void schedulePeriodicJob() {
    int jobId = new JobRequest.Builder(DemoSyncJob.TAG)
            .setPeriodic(TimeUnit.MINUTES.toMillis(15), TimeUnit.MINUTES.toMillis(5))
            .setPersisted(true)
            .build()
            .schedule();
}

private void scheduleExactJob() {
    int jobId = new JobRequest.Builder(DemoSyncJob.class)
            .setExact(20_000L)
            .setPersisted(true)
            .build()
            .schedule();
}

private void cancelJob(int jobId) {
    JobManager.instance().cancel(jobId);
}
```

如果非周期性的  `Job` 执行失败,我们可以设定恢复标准重新计划这个 job

```java
public class RescheduleDemoJob extends Job {

    @Override
    @NonNull
    protected Result onRunJob(Params params) {
        // 一些异常情况发生，稍后重试这个Job
        return Result.RESCHEDULE;
    }

    @Override
    protected void onReschedule(int newJobId) {
        // 恢复的Job有一个新的 ID
    }
}
```

**提醒:**  谷歌在安卓 Marshmallow  版本引入自动备份功能，所有的 job 信息保存在一个文件名为 `evernote_jobs.xml`的 shared preference ，   以及文件名为`evernote_jobs.db` 的数据库中. 我们应该排除这些文件，避免他们被自动备份。
我们可以定义一个resource XML 文件 (比如, `res/xml/backup_config.xml`) 内容如下:

```xml
<?xml version="1.0" encoding="utf-8"?>
<full-backup-content>
    <exclude domain="sharedpref" path="evernote_jobs.xml" />
    <exclude domain="database" path="evernote_jobs.db" />
</full-backup-content>
``` 

然后在应用的 `AndroidManifest.xml`指向这个文件:

```xml
<application ...  android:fullBackupContent="@xml/backup_config">
```

### 代码混淆

这个库没有使用反射，但是它依赖两个 `Service` 以及两个 `BroadcastReceiver` 为了避免任何可能出现的问题，不应该混淆这四个类。这个库打包了它自己的混淆配置，所以我们不需要做任何额外的配置。以防万一，也可以在我们项目的混淆配置中添加[这些规则](library/proguard.txt) 


### FAQ

 [点击这里](FAQ.md)查看更多的疑问解答。

# 参考资料
[evernote的Blog](https://blog.evernote.com/tech/2015/10/26/unified-job-library-android/)
[作者Ralf的PPT](https://speakerdeck.com/vrallev/scheduling-background-job-on-android-at-the-right-time)




