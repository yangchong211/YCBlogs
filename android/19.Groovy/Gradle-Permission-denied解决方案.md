今天在查看Android项目的依赖关系时，发现蜜汁好用的gradle命令权限被限制了：

```
qingmeideMac-mini:FireProtectionClient_Android qing.mei$ ./gradlew -q app:dependencies
//注意这行，被提示没有权限
-bash: ./gradlew: Permission denied
```
最后在 [stackoverflow-gradlew: Permission Denied](https://stackoverflow.com/questions/17668265/gradlew-permission-denied)找到了答案：

> 输入 chmod +x gradlew   

该命令的作用是是Linux下去除执行权限。
```
//输入该命令
qingmeideMac-mini:FireProtectionClient_Android qing.mei$ chmod +x gradlew

//检查权限，发现该命令可以用了
qingmeideMac-mini:FireProtectionClient_Android qing.mei$ ./gradlew

> Configure project :app
Configuration 'provided' in project ':app' is deprecated. Use 'compileOnly' instead.
app: 'androidProcessor' dependencies won't be recognized as kapt annotation processors. Please change the configuration name to 'kapt' for these artifacts: 'com.google.dagger:dagger-compiler:2.11', 'com.google.dagger:dagger-android-processor:2.11', 'com.github.bumptech.glide:compiler:4.2.0', 'org.projectlombok:lombok:1.16.18', 'com.android.databinding:compiler:3.0.1' and apply the kapt plugin: "apply plugin: 'kotlin-kapt'".

> Task :help

Welcome to Gradle 4.1.

...

BUILD SUCCESSFUL in 0s
1 actionable task: 1 executed
qingmeideMac-mini:FireProtectionClient_Android qing.mei$ 

```

通过这个问题，深深感觉到，只是单纯的懂得配置gradle是不够的，接下来更需要深入学习这门脚本语言。
