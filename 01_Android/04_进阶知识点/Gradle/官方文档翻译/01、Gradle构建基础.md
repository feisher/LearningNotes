基于 https://developer.android.com/studio/build/ 整理。

文中图片也来自于上述地址。


### 1、Grdle构建流程

![](https://images.gitee.com/uploads/images/2018/1217/213244_5ea72551_930142.png "屏幕截图.png")

### 2、可自定义的构建(编译)配置

#### （1）、构建类型

对应：buildTypes{}

[点击查看详细说明](https://developer.android.com/studio/build/build-variants#build-types)

* 定义 Gradle 在构建和打包应用时使用的某些属性
* **针对于开发周期中的不同阶段进行定义——开发阶段、测试阶段、生产阶段**
* 可配置 压缩、混淆、签名文件等内容
* 默认有 debug / release 两种


#### （2）、产品风味

对应 productFlavors{}

[点击查看详细说明](https://developer.android.com/studio/build/build-variants#product-flavors)

* 构建针对于不同用户群体的版本，如：免费版和付费版等
* 此处默认没有配置，如有需要，必须手动配置


#### （3）、构建变体
variant  ['veərɪənt] 变体，转化

[点击查看详细](https://developer.android.com/studio/build/build-variants)

* 是 buildTypes 和 productFlavors 的组合产物。
* 配置了不同的 type 和 flavor后，对应的就会产生不同的变体组合。如：开发阶段的付费版、开发阶段的免费版、生产阶段的付费版、生产阶段的免费版等。

#### （4）、清单条目

* 对应 AndroidManifest.xml 的属性
* 在构建变体时，可以指定清单文件中的一些属性值，如 applicationID、minSdkVersion 等
    - 基于这些配置，就可以在一套源码下编译出不同 id、不同名称的APP
* 配置了相关属性之后，Gradle 会自动合并这些属性值。[点击查看详细的合并说明](https://developer.android.com/studio/build/manifest-merge)


#### （5）、依赖项

对应 dependencies{}

[点击查看详细说明](https://developer.android.com/studio/build/build-variants#dependencies)

* 管理本地libs 或 远程的一些三方依赖库


#### （6）、签署/签名

对应 signingConfigs{}

[点击查看详细说明](https://developer.android.com/studio/publish/app-signing)
* 指定签名文件后，Gradle可以在编译过程中动态的加载签名文件。

#### （7）、ProGuard

[点击查看详细说明](https://developer.android.com/studio/build/shrink-code)
* 配置编译过程中压缩和混淆的相关规则

#### （8）、APK 拆分

对应 splits{}
* 自动构建针对于 **不同屏幕密度或特定ABI(应用二进制界面)** 的apk


### 3、与构建相关的配置文件

自定义构建配置相关属性时，需要在配置文件中进行修改。

这些配置文件包括：
* buidle.gradle(project)
* builde.gradle(module)
* setting.gradle
* gradle.properties

#### （1）、Gradle 设置文件

即  **Setting.gradle** 

用于指示Gradle在构建时需要包含哪些模块。

* 必然会包含当前项目 **include ‘:当前项目名’**
* 如果项目有依赖其他module,需要在后面追加，如  **include ':yourAppName', ':opencv', ":tess-two", ':ucrop'** 

#### (2)、顶级构建文件

即  **build.gradle(project)** 

用来定义项目中适用于所有模块（module）的相关配置.

* 默认情况下使用 buildscript{}代码块定义所有模块共用的gradle存储区和依赖项


**A:示例** 

```
/**
 * 该代码块定义了Gradle自己需要的repositories（仓库） 和 dependencie（依赖项）。此处不要定义module所使用的dependencie。
 * 比如说，该代码块中包含一些 Android 插件，这些插件作为Gradle的依赖项可以为Gradle在构建项目时提供额外的一些组件支持。
 */
buildscript {

    /**
     * 该代码块配置了Gradle用来搜索和下载 dependencies 的仓库（repositories）。
     * 这些仓库可以是一些远程仓库，比如：JCenter, Maven Central, Ivy 等。
     * 也可以使用本地仓库，或者是你自己的远程仓库。
     * 在示例中，包含了 jcenter() ，该仓库中包含了Gradle必须的依赖项（dependencies）.
     */
    repositories {
        jcenter()
        google()
    }

    /**
     * 该代码块定义了Gradle在构建项目时所需要的依赖项。
     * 在示例中，通过路径依赖的方式引用了 3.2.1 版本的Gradle组件。
     */
    dependencies {
        classpath 'com.android.tools.build:gradle:3.2.1'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

/**
 * 该代码块定义当前项目中被所有模块使用的仓库和依赖项。比如，三方的插件或仓库。
 * 非所有模块共用的依赖需要定义在各个模块的 build.grale(module)中。
 * 对于新的项目，会默认配置一个jcenter()仓库。
 */
allprojects {
    repositories {
        jcenter()
        google()
        maven { url 'https://jitpack.io' }
    }
}
```



#### (3)、模块级构建文件

对应  **build.gradle(module)** 

用于配置模块所需要的相关构建设置。

可以在此处自定义打包时的相关构建设置（如 type、flavor等）

还可以替换 AndroidMainfest.xml 或 顶级构建文件——build.gradle(project) 中的相关设置。


**A：示例：** 


```
/**
 * 作为文件的第一行，它定义了Gradle构建APP时所使用的Android插件。
 * 并且启用了 android {} 代码块中的各种自定义构建属性。
 * 如果是可运行的module，此处为：apply plugin: 'com.android.application'
 * 如果是被依赖的module，此处为：apply plugin: 'com.android.library'
 */
apply plugin: 'com.android.application'

/**
 * 该代码块中定义了构建APP时的各种个性化设置。
 */
android {

  /**
   * complieSdkVersion—— 指定Gradle编译APP时可以使用的Android API 版本。也就是说，我们在写代码时可以使用不高于该版本的API属性。如果高了就会报错。
   *
   * buildToolsVersion——指定Gradle构建APP时所使用的 SDK构建工具版本、命令行工具版本、编译器版本。 指定完成之后，我们需要使用SDK Manager 下载对应版本的SDK。
   */
  compileSdkVersion 28
  buildToolsVersion "28.0.3"

  /**
   * 该代码块中包含了针对所有变体的默认设置。
   * 通过调整这些设置，可以在构建APP时动态改变定义在清单文件中的一些属性。
   */
  defaultConfig {

    /**
     * 这是APP的唯一标识。默认情况下，它与定义在清单文件中的包名保持一致，但二者并没有必然关系
     */
    applicationId 'com.example.myapp'

    //定义APP的最低兼容版本
    minSdkVersion 15

    // 指定测试APP时所使用的API版本——Specifies the API level used to test the app.
    targetSdkVersion 28

    // 定义APP的版本号——Defines the version number of your app.
    versionCode 1

    // 定义APP的版本名称——Defines a user-friendly version name for your app.
    versionName "1.0"
  }

  /**
   * 在该代码块中可以定义多种基于APP不同开发阶段的构建类型
   * 默认情况下，会定义两种：debug(开发版本)、release(生产版本) 
   * debug(开发版本)默认不显示，其中包含了debug调试工具，并且使用debug版本的签名文件
   * release(生产版本)会启用代码混淆和资源压缩，但是并没有指定签名文件
   */
  buildTypes {

    /**
     * 默认启用资源压缩，并使用代码混淆——同时会指定混淆规则
     */
    release {
        //a 启用资源压缩
        minifyEnabled true
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
  }

  /**
   * 在该代码块中定义针对于不同用户群体/不同发布平台的相关内容。
   * 在此处可以重写 defaultConfig {} 中的一些属性,从而构建出不同的App
   * 该代码块是可选的，默认不会被创建 
   * 下列示例中，创建了一个免费版本和一个收费版本,它们各自拥有一个 application ID—— 这样它们就可以在应用市场上共存,也可以在同一个设备上共存
   */
  productFlavors {
    free {
      applicationId 'com.example.myapp.free'
    }

    paid {
      applicationId 'com.example.myapp.paid'
    }
  }

  /**
   * 在该代码块中可以定义针对于不同屏幕密度或者ABI的构建参数——也就是分包/APK 拆分。
   * You'll also need to configure your build so that each APK has a different versionCode.
   */
  splits {
    // 基于屏幕密度的拆分设置
    density {

      // 启用或禁用基于屏幕密度的拆分机制
      enable false

      // Exclude these densities from splits
      exclude "ldpi", "tvdpi", "xxxhdpi", "400dpi", "560dpi"
    }
  }
}

/**
 * 指定构建module时所需要的 dependencies
 */
dependencies {
    compile project(":lib")
    compile 'com.android.support:appcompat-v7:28.0.0'
    compile fileTree(dir: 'libs', include: ['*.jar'])
}
```


#### （4）、Gradle属性文件

用于指定适用于 Gradle 构建工具包本身的设置.

包括如下两个属性文件：

**A:gradle.properties** 

可以配置项目范围的Gradle设置，如 Gradle 后台进程的最大堆大小。

详细可参考：https://docs.gradle.org/current/userguide/build_environment.html

**B:local.properties** 

为构建系统配置本地环境属性，例如 SDK 安装路径。

该文件是Android Studio 自动生成的，所以不需要手动修改



#### （5）、将项目与Gradle文件同步

也就是 SyncProject.

修改过gradle相关内容之后需要执行该操作。


#### (6)、源集——SourceSet

 **A：基本源集** 

![](https://images.gitee.com/uploads/images/2018/1218/155145_3417867e_930142.png "屏幕截图.png")

如上图，AndroidStudio中每个模块对应的源代码和资源被称为源集——SourceSet。

**说的更直白一点，在AndroidStudio中，切换为 project 目录视图.此时，src 目录下包含的目录就被称为源集**
* 新建的项目通常会包含 src/androidTest、src/main 两个源集。
* src/main 是主源集，所有的构建变体都会引用其中的内容。


**B:其他源集**
 
如果我们的APP需要定义不同的buildType 或 productFlavor，那么对应的也可以有如下源集：

*  **src/<buildType>/** 

其中包含 **特定构建类型** 专用的代码和资源。

*  **src/<productFlavor>/** 

其中包含 **特定产品风味** 专用的代码和资源。

*  **src/<productFlavorBuildType>/** 

其中包含 **特定构建变体** 专用的代码和资源。


 **C:源集的优先级** 

当我们定义了多个源集后，Gradle在构建时会合并不同源集中的代码和资源。

如果不同源集中含有同名文件，那么它们的优先级和替代关系如下：

>  **构建变体 --> 构建类型 --> 构建风味 --> 主源集 --> 依赖库** 
 
* **左侧源集会替代右侧源集中的内容** 
* 如果多个源集中包含了AndroidMainfest.xml，它们的替代关系同上。

更多源集内容可参考：https://developer.android.com/studio/build/build-variants#sourcesets




