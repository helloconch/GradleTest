# GradleTest
##基本配置
###app/build
```
apply plugin: 'com.android.application'

android {
    compileSdkVersion 24
    buildToolsVersion "24.0.0"

    defaultConfig {
        applicationId "com.gradle.testing"
        minSdkVersion 14
        targetSdkVersion 24
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:24.0.0'
}
```

```
apply plugin: ‘com.android.application’，表示该module是一个app module，应用了com.android.application插件，
如果是一个android library，那么这里写apply plugin: ‘com.android.library’

compileSdkVersion：基于哪个SDK编译，这里是API LEVEL

buildToolsVersion：基于哪个构建工具版本进行构建的。

defaultConfig：默认配置，如果没有其他的配置覆盖，就会使用这里的。
applicationId：配置包名的
versionCode：版本号
versionName：版本名称

buildTypes是构建类型，常用的有release和debug两种，可以在这里面启用混淆，启用zipAlign以及配置签名信息等。

```
###gradle-wrapper.properties
```
声明了gradle的目录与下载路径以及当前项目使用的gradle版本，这些默认的路径我们一般不会更改的.
```

###根目录的build.gradle
```
buildscript {
    repositories {
        jcenter()//使用jcenter库
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.1.2'// 依赖android提供的1.5.0的gradle build

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}
//为所有的工程的repositories配置为jcenters
allprojects {
    repositories {
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

###setting.gradle
包含哪些模块，比如有app和library


##依赖管理
###本地依赖
```
jar
默认情况下，新建的Android项目会有一个lib文件夹
dependencies {
       compile fileTree(dir: 'libs', include: ['*.jar'])//即添加所有在libs文件夹中的jar
       //compile files('libs/WuXiaolong.jar')//不需要这样一个个去写了
}

so包
用c或者c++写的library会被叫做so包，Android插件默认情况下支持native包，你需要把.so文件放在对应的文件夹中
app
   ├── AndroidManifest.xml
   └── jni
       ├── armeabi
       │   └── WuXiaolong.so
       ├── armeabi-v7a
       │   └── WuXiaolong.so
       ├── mips
       │   └── WuXiaolong.so
       └── x86
           └── WuXiaolong.so

```

###aar文件
```
library库输出文件是.aar文件，包含了Android 资源文件，在library工程build/output/aar/下

直接依赖library库
dependencies {
       compile project(':library名字')
       compile project(':libraries:library名字')//多个library，libraries是文件夹名字
  }
  

依赖.aar文件
创建一个aars文件夹，然后把.aar文件拷贝到该文件夹里面，然后添加该文件夹作为依赖库：
app/bulid.gradle
repositories {
    flatDir {
        dirs 'aars' 
    }
}
dependencies {
       compile(name:'libraryname', ext:'aar')
}

注意：如果你的library依赖了第三方库，须app再次依赖。

```

###远程仓库
```
dependencies {
		compile 'com.wuxiaolong.pullloadmorerecyclerview:library:1.0.4'
}
```


##全局设置
```
如果有很多项目，可以设置全局来统一管理版本号或依赖库，根目录下build.gradle下：
ext {
    compileSdkVersion = 23
    buildToolsVersion = "23.0.2"
    minSdkVersion = 14
    targetSdkVersion = 23
}

app/build.gradle
android {
    compileSdkVersion rootProject.ext.compileSdkVersion
    buildToolsVersion rootProject.ext.buildToolsVersion

    defaultConfig {
        applicationId "com.wuxiaolong.gradle4android"
        minSdkVersion rootProject.ext.minSdkVersion
        targetSdkVersion rootProject.ext.targetSdkVersion
        versionCode 1
        versionName "1.0"
    }
    

也可以在根目录下建个config.gradle，然后只需在根目录下build.gradle最顶部加上下面一行代码，
然后同步下，意思就是所有的子项目或者所有的modules都可以从这个配置文件里读取内容。
apply from: "config.gradle"
config.gradle
ext {

    android = [
            compileSdkVersion: 23,
            buildToolsVersion: "23.0.2",
            minSdkVersion    : 14,
            targetSdkVersion : 22,

    ]

    dependencies = [
            appcompatV7': 'com.android.support:appcompat-v7:23.2.1',
            design      : 'com.android.support:design:23.2.1'

    ]
}


app/build.gradle
android {
    compileSdkVersion rootProject.ext.android.compileSdkVersion
    buildToolsVersion rootProject.ext.buildToolsVersion

    defaultConfig {
        applicationId "com.wuxiaolong.gradle4android"
        minSdkVersion rootProject.ext.android.minSdkVersion
        targetSdkVersion rootProject.ext.android.targetSdkVersion
        versionCode 1
        versionName "1.0"
    }
  
...

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile rootProject.ext.dependencies.appcompatV7
    compile rootProject.ext.dependencies.design
}

```


##自定义BuildConfig
```
实际开发中服务器可能有正式环境和测试环境，gradle可以通过buildConfigField来配置。
defaultConfig {
       buildConfigField 'String','API_SERVER_URL','"http://wuxiaolong.me/"'
   }
   
buildConfigField 一共有3个参数，第一个是数据类型，和Java的类型是对等的；
第二个参数是常量名，这里是API_SERVER_URL；第三个参数就是你要配置的值。

如图路径下就有个常量API_SERVER_URL，如何在代码取得这个常量值：
Log.d("wxl", "API_SERVER_URL=" + BuildConfig.API_SERVER_URL);

```


##启用proguard混淆
```
一般release发布版本是需要启用混淆的，这样别人反编译之后就很难分析你的代码，
而我们自己开发调试的时候是不需要混淆的，所以debug不启用混淆。对release启用混淆的配置如下：

android {

    buildTypes {
        release {
            minifyEnabled true//是否启动混淆
			shrinkResources true //是否移除无用资源文件，shrinkResources依赖于minifyEnabled，必须和minifyEnabled一起用
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
   }
}
minifyEnabled为true表示启用混淆，proguardFile是混淆使用的配置文件，
这里是module根目录下的proguard-rules.pro文件

```
##多渠道打包
```
国内有太多Android App市场，每次发版几十个渠道包。
还好Android Gradle给我们提供了productFlavors，我们可以对生成的APK包进行定制。
productFlavors {//多渠道打包
    xiaomi {
        applicationId 'com.gradle.testing1'
    }

    googlepaly {
        applicationId 'com.gradle.testing2'
    }
}
```
##定制生成的apk文件名
```
applicationVariants.all { variant ->
             if (variant.buildType.name.equals('release')) {
                 variant.outputs.each { output ->
                     def outputFile = output.outputFile
                     if (outputFile != null && outputFile.name.endsWith('.apk')) {
                         def fileName = "gradletest_v${defaultConfig.versionName}_${releaseTime()}_${variant.flavorName}.apk"
                         output.outputFile = new File(outputFile.parent, fileName)
                     }
                 }
             }
         }
         
         
 输出apk名字：gradletest_v1.0_2016-03-23_xiaomi.apk
```
##占位符
```
多渠道打包，还会遇到一个问题，比如友盟统计的渠道号，Gradle处理办法：manifestPlaceholders，
它允许我们动态替换我们在AndroidManifest文件里定义的占位符。
AndroidManifest.xml：
<meta-data
           android:name="UMENG_CHANNEL"
           android:value="${UMENG_CHANNEL_VALUE}" />

如下，${UMENG_CHANNEL_VALUE}占位符会被dev替换。
defaultConfig {
       manifestPlaceholders = [UMENG_CHANNEL_VALUE: 'dev']
   }
   
 如果渠道太多，不用这样一个个去写，可以循环
 productFlavors.all { flavor ->
                manifestPlaceholders.put("UMENG_CHANNEL_VALUE",name)
            }


渠道打包完整代码：
android {
//省略部分代码
     buildTypes {
        release {
            minifyEnabled false//是否启动混淆
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            applicationVariants.all { variant ->
                if (variant.buildType.name.equals('release')) {
                    variant.outputs.each { output ->
                        def outputFile = output.outputFile
                        if (outputFile != null && outputFile.name.endsWith('.apk')) {
                            def fileName = "gradleTest_v${defaultConfig.versionName}_${releaseTime()}_${variant.flavorName}.apk"
                            output.outputFile = new File(outputFile.parent, fileName)
                        }
                    }
                }
            }
            //针对很多渠道
            //productFlavors.all { flavor ->
            //   manifestPlaceholders.put("UMENG_CHANNEL_VALUE",name)
            // }
        }
    }
    productFlavors {//多渠道打包，命令行打包：gradlew assembleRelease
        xiaomi {
            applicationId 'com.wuxiaolong.gradle4android1'
            manifestPlaceholders.put("UMENG_CHANNEL_VALUE", 'xiaomi')
        }
        googlepaly {
            applicationId 'com.wuxiaolong.gradle4android2'
            manifestPlaceholders.put("UMENG_CHANNEL_VALUE", 'googlepaly')
        }
    }
 //省略部分代码

def releaseTime() {
    return new Date().format("yyyy-MM-dd", TimeZone.getTimeZone("UTC"))
}
```

##配置签名信息

```
Android Studio设置默认的签名文件
新浪微博SSO登录，微信分享这些都需要签名打包，才能看到效果，设置默认签名文件为自己的签名jks，
这样就不需要打包了直接运行起来就是正式的签名。
在android.signingConfigs{}下定义一个或者多个签名信息，然后在buildTypes{}配置使用即可。
在app目录下添加你的.jks，然后app的build.gradle文件中的增加以下内容：
android {  
    signingConfigs {  
        release {  
            storeFile file("xxx.jks")
            storePassword 'android'
            keyAlias 'android'
            keyPassword 'android'
        }          
    }  
	buildTypes {
        debug {
            signingConfig signingConfigs.release
        }        
    }
}
```

##签名打包
```
通过Android Studio签名
通过命令行签名
如上那样配置签名信息
android {  
    signingConfigs {  
        release {  
            storeFile file("xxx.jks")
            storePassword 'android'
            keyAlias 'android'
            keyPassword 'android'
        }          
    }  
	buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release
        }      
    }
}

先”build-clean Project”，然后Terminal输入命名行：
gradlew assembleRelease
打印信息如下：
E:\AndroidStudioProjects\Gradle4Android>gradlew assembleRelease
:app:preBuild UP-TO-DATE                                                             
:app:preReleaseBuild UP-TO-DATE     
:app:checkReleaseManifest                  
//省略部分               
:app:packageRelease                 
:app:zipalignRelease                 
:app:assembleRelease                 
               
BUILD SUCCESSFUL
OK，打包成功的apk路径如：E:\AndroidStudioProjects\Gradle4Android\app\build\outputs\apk\app-release.apk
```