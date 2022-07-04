---
title: 使用Github Actions  完成编译、发布release apk 
date: 2022-4-25 17:30:55
categories:
    - 构建

tags:
    - 构建
    - ci
    - Github

---


使用*Github Actions，*我们可以编译，运行测试，还可以**签署发布工件** 。 **但是**，如果我们将密钥库上载到公共存储库，这很***危险**。作为开发人员，我们必须采取一些预防措施，例如**不要提交API密钥或密钥库***（使用*Github Actions*作为CI并不是这样做的借口。）

那我们怎样才能在仓库中使用github actions 完成编译签署发布工作，同时又保证密钥和密钥口令的安全性呢？

跟着以下步骤，你应该就能明白了。

# 创建签名环境

创建签名环境，主要实现两个目标，一是保护签名文件不被泄露，公共存储库是所有人可读的。二是保证签名文件的密钥密码等信息不被泄露，包括在执行编译、发布apk整个过程的安全。



## 创建存储签名文件的私有仓库

首先，我们在github上创建一个**私有**仓库，为简单起见，我们将其称为` keystore`。得到的仓库地址是`git@github.com:Ernest-su/keystore.git`

然后，把仓库clone到本地。`git clone git@github.com:Ernest-su/keystore.git`

将签名文件放到keystore内.例如，我的签名文件名称是`keyfile`

```
git add .\keyfile
git commit -m "添加签名文件"
git push origin main
```

**注意**: 在github上新建仓库的时候，github已经把默认分支从以前的`master`改为了`main` ,所以上面的命令是`git push origin main`，而不是之前的`git push origin master`

## 添加签名文件仓库为git的子模块

切换到项目目录，使用以下命令将包含密钥文件的私有存储库添加为子模块，为简单起见，我们将其称为` keystore`。

`git submodule add git@github.com:Ernest-su/keystore.git keystore`

这样，我们便拥有了可以在本地进行签名的密钥库，并且因为该仓库是私有仓库，其他人**不会**获取到签名文件。

![image-20220426130338684](..\images\image-20220426130338684.png)

可以看到，项目已经多了一个`keystore`文件夹，里面有我们的签名文件。因为是作为git的子模块`submodule` 来添加的，所以提交的时候，项目只会引用子模块的提交信息，不会包含此文件夹。

![image-20220426130224297](..\images\image-20220426130224297.png)



## 加密的环境变量 Secrets

为了拥有更**安全的环境**来使用github actions签署apk,我们在项目的设置中做以下配置。

在项目 `Settings -> Secrets-> Actions`  中配置加密的环境变量，这里配置的环境变量很安全，有[特殊处理](https://docs.github.com/en/free-pro-team@latest/actions/reference/encrypted-secrets)，不会在日志中显示。同时也不会传递由 `Fork` 发起的 `Pull Request`。

以下配置，KEY*开头的是签名文件的密码和签名别名，根据上面添加为git子模块的签名文件填入正确的信息。

![](..\images\image-20220425174715402.png)



`TOKEN`配置的是github 的 `PAT`,也就是Personal access tokens，这里就不展开介绍了，可以查阅[官方的说明](https://docs.github.com/cn/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) 这里配置的`TOKEN`用于Github Actions执行时，拉取私有仓库子模块的签名文件。

![image-20220426130619781](..\images\image-20220426130619781.png)



## 修改app签名配置

在**app模块**的`build.gradle`中调整签名配置。`System.getenv`读取的是系统环境变量，在GitHub Actions执行时候，读取的是上面**加密的环境变量 Secrets** 中配置的信息。


```groovy
    signingConfigs {
        release {

            storeFile file(KEYSTORE)

            storePassword System.getenv("KEYSTORE_PASSWORD")

            keyAlias System.getenv("KEY_ALIAS")

            keyPassword System.getenv("KEY_PASSWORD")
        }
    }
```

在**项目根目录**的`gradle.properties`配置以下信息。我们把`KEYSTORE`直接在这里定义了，而不是通过获取系统环境变量的方式。当然，通过上面定义加密的环境变量Secrets,再用读取系统环境变量的方式，也是ok的。

```properties

KEYSTORE=../keystore/keyfile
#以下配置到环境变量里了,在github actions的脚本中会自动读取
# KEY_ALIAS=System.getenv("KEY_ALIAS")
# KEY_PASSWORD=[your key password]
# KEYSTORE_PASSWORD=[your store password]
```



# 配置Github Actions 

在项目新建`.github\workflows`目录，在`workflows`目录下新建`android.yml`文件,内容如下：

```yaml
name: Android CI

on:
  push:
    tags:
      - '*'

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
      with:
       submodules: recursive
       token: ${{ secrets.TOKEN }}
    - name: set up JDK 11
      uses: actions/setup-java@v3
      with:
        java-version: '11'
        distribution: 'temurin'
        cache: gradle

    - name: Grant execute permission for gradlew
      run: chmod +x gradlew
    - name: Build with Gradle
      env: #  as an environment variable
        KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
        KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
        KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
      run: ./gradlew assemble
    - name: Upload binaries to release
      uses: svenstaro/upload-release-action@v2
      with:
        repo_token: ${{ secrets.GITHUB_TOKEN }}
        file:  app/build/outputs/apk/release/*.apk
        tag: ${{ github.ref }}
        release_name: ${{ github.ref }}
        overwrite: true
        file_glob: true
        body: ""


```

## Github Actions 触发条件

简单说明下配置的含义。

```
on:
  push:
    tags:
      - '*'
```

意思是在推送tag到远程仓库时候，触发GitHub Actions。`'*'` 表示推送所有的tag都会触发，可以改为`'v*'`,表示推送了`v`开头的tag才触发。如果希望是在推送到特定分支时候触发，如`release`，则可以这样配置

```
on:
  push:
    branches: [ release ]
```

或需要推送到`master`分支时候触发
```
on:
  push:
    branches: [ master ]
```

但是需要注意,如果`master`分支是默认分支，且是私有仓库，会浪费掉GitHub Actions的运行时间额度。公共仓库和自托管运行器免费使用 GitHub Actions。 对于私有仓库，每个 GitHub 帐户可获得一定数量的免费记录和存储。[关于额度的说明](https://docs.github.com/cn/billing/managing-billing-for-github-actions/managing-your-spending-limit-for-github-actions)



## Github Actions 的子模块配置

    - uses: actions/checkout@v3
      with:
       submodules: recursive
       token: ${{ secrets.TOKEN }}

只有加了`  submodules: recursive`以及`   token: ${{ secrets.TOKEN }}`才会在构建时候拉取到私有仓库的子模块。



## 读取Actions 加密系统环境变量

    - name: Build with Gradle
      env: #  as an environment variable
        KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
        KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
        KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}

此步骤用于配置解密后的环境变量到系统环境变量中,供`gradle`脚本使用

## 上传apk文件到github release

    - name: Upload binaries to release
      uses: svenstaro/upload-release-action@v2
      with:
        repo_token: ${{ secrets.GITHUB_TOKEN }}
        file:  app/build/outputs/apk/release/*.apk
        tag: ${{ github.ref }}
        release_name: ${{ github.ref }}
        overwrite: true
        file_glob: true
        body: ""
