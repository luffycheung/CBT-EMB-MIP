# 实验一. Android NDK之Hello World开发入门实验




---
[toc]



原文链接：[Add C and C++ Code to Your  

Project](https://developer.android.com/studio/projects/add-native-code.html#existing-project)




## **术语解释**

###NDK



> NDK（英语：native development kit，简称NDK）是一种基于原生程序接口的软件开发工具。通过此工具开发的程序直接以本地语言运行，而非虚拟机。因此只有java等基于虚拟机运行的语言的程序才会有原生开发工具包。

 - NDK是一系列工具的集合

> NDK提供了一系列的工具，帮助开发者快速开发C（或C++）的动态库，并能自动将so和java应用一起打包成apk。这些工具对开发者的帮助是巨大的。

 - NDK集成了交叉编译器，并提供了相应的mk文件隔离CPU、平台、ABI等差异，开发人员只需要简单修改mk文件（指出“哪些文件需要编译”、“编译特性要求”等），就可以创建出so。

> NDK可以自动地将so和Java应用一起打包，极大地减轻了开发人员的打包工作。  

 - 那我们为什么要使用呢?   

1. 代码的保护。由于apk的java层代码很容易被反编译，而C/C++库反汇难度较大。  
2. 可以方便地使用现存的开源库。大部分现存的开源库都是用C/C++代码编写的。  
3. 提高程序的执行效率。将要求高性能的应用逻辑使用C开发，从而提高应用程序的执行效率。  
4. 便于移植。用C/C++写得库可以方便在其他的嵌入式平台上再次使用。   

NDK提供了一份稳定、功能有限的 API 头文件声明

Google 明确声明该API是稳定的，在后续所有版本中都稳定支持当前发布的API。从该版本的NDK中看出，这些API支持的功能非常有限，包含有：C 标准库（libc）、标准数学库（libm）、压缩库（libz）、Log 库（liblog）。

### JNI
> JNI（Java NativenInterface）是java与C/C++进行通信的一种技术，使用JNI技术，可以java调用C/C++的函数对象等等，Android中的Framework层与Native层就是采用的JNI技术。

我们知道，Android系统是基于linux开发，采用的是linux内核 ，Android APP开发大部分也要和系统打交道，只是Android FrameWork 帮我们处理了和系统相关的操作， 我们从Android 系统的分成结构可以看出，Android FrameWork是通过JNI与底层的C/C++库交互，例如：FreeType，OpenGL，SQLite，音视频等等。
![Android系统框图][1]   

如果我们程序也需要调用自己的C/C++函数库，就必须用到JNI/NDK开发。

###Cmake
> CMake 是个开源的跨平台自动化建构系统，它用配置文件控制建构过程（build process）的方式和Unix的Make相似，只是CMake的配置文件取名为CMakeLists.txt。Cmake并不直接建构出最终的软件，而是产生标准的建构档（如Unix的Makefile或Windows Visual C++的projects/workspaces），然后再依一般的建构方式使用。这使得熟悉某个集成开发环境（IDE）的开发者可以用标准的方式建构他的软件，这种可以使用各平台的原生建构系统的能力是CMake和SCons等其他类似系统的区别之处。CMake可以编译源代码、制做程序库、产生适配器（wrapper）、还可以用任意的顺序建构可执行文件。CMake支持in-place建构（二进档和源代码在同一个目录树中）和out-of-place建构（二进档在别的目录里），因此可以很容易从同一个源代码目录树中建构出多个二进档。CMake也支持静态与动态程序库的建构。
“CMake”这个名字是"cross platform make"的缩写。虽然名字中含有"make"，但是CMake和Unix上常见的“make”系统是分开的，而且更为高级。

 - Cmake的优势
1. 可以直接的在C/C++代码中加入断点，进行调试
2. java引用的C/C++中的方法，可以直接ctrl+左键进入
3. 对于include的头文件，或者库，也可以直接的进入
4. 不需要配置命令行操作,手动的生成头文件,不需要配置android.useDeprecatedNdk=true 属性




使用 Android studio，你可以将 C 和 C++ 代码编译成 native library（即 .so  
文件），然后打包到你的 APK 中。你的 Java 代码可以通过 Java Native  
Interface（JNI）调用 native library 中的方法。


Android Studio 默认使用 CMake 编译原生库。由于已经有大量的代码使用了ndk-build 来编译 native code，所以Android Studio同样也支持 ndk build。如果你想导入一个 ndk-build 库到你的 Android Studio项目中，请参阅后文的 **关联本地库与 Gradle**。然而，如果你创建了一个新的
native 库工程，你应该使用 CMake。




本篇文章将会说明如何使用 Android Studio 来创建、配置 Android  

项目，以支持 native code，以及将其运行到你的 app 中。




> 注意：要在 Android Studio 中使用 CMake 或者 ndk-build，你需要使用  

> Android Studio 2.2 或更高的版本，同时需要配合使用 Android Plugin for  

> Gradle 2.2.0 及以上的版本。




## 下载 NDK 和构建工具 {#下载-NDK-和构建工具}




要编译和调试本地代码（native code），你需要下面的组件：




* _`The Android Native Development Kit (NDK)`_: 让你能在 Android

  上面使用 C 和 C++ 代码的工具集。

* _`CMake`_: 外部构建工具。如果你准备只使用 ndk-build

  的话，可以不使用它。

* _`LLDB`_: Android Studio 上面调试本地代码的工具。




你可以使用 SDK Manager 来安装上述组件：




1. 打开一个项目，从菜单栏中选择 **Tools &gt; Android &gt; SDK Manager**。

2. 点击 **SDK Tools** 选项卡。

3. 勾选 **LLDB，CMake** 和 **NDK**。




![Figure-1.png-34.8kB](http://static.zybuluo.com/wl9739/58b11an8k32rylerlh291f7b/Figure-1.png)




1. 点击 **Apply**，然后点击 **OK**。

2. 当安装完成后，点击 **Finish**，然后点击 **OK**。




## 创建支持 C/C++ 的新项目 {#创建支持-C-C-的新项目}




创建一个支持 native code 的项目和创建普通的 Android studio  

工程很像。但是有几点需要留意的地方：




1. 在 **Configure your new project** 选项中，勾选 \*\*Include C++

   Support\*\* 选项。

2. 点击 Next，后面的流程和创建普通的 Android studio 工程一样。

3. 在 **Customize C++ Support**  

   选项卡中。你有下面几种方式来自定义你的项目：




   * **C++ Standard**：点击下拉框，可以选择标准 C++，或者选择默认

     CMake 设置的 **Toolchain Default** 选项。

   * **Exceptions Support**：如果你想使用有关 C++

     异常处理的支持，就勾选它。勾选之后，Android Studio 会在 module

     层的 build.gradle 文件中的 **cppFlags** 中添加 **-fexcetions**

     标志。

   * **Runtime Type Information Support**：如果你想支持

     RTTI，那么就勾选它。勾选之后，Android Studio 会在 module 层的

     build.gradle 文件中的 **cppFlags** 中添加 **-frtti** 标志。




4. 点击 “Finish”。







当 Android Studio 完成新项目创建后，打开 **Project** 面板，选择  

**Android** 视图。Android Studio 会添加 **cpp** 和 **External Build  

Files** 目录。




![Figure-2.png-41.4kB](http://static.zybuluo.com/wl9739/ltzzvosh6y1uulszsk6l7z5y/Figure-2.png)




1. **cpp** 目录存放你所有 native code

   的地方，包括源码，头文件，预编译项目等。对于新项目，Android Studio

   创建了一个 C++ 模板文件：**native-lib.cpp**，并且将该文件放到了你的

   app 模块的 **src/main/cpp/** 目录下。这份模板代码提供了一个简答的

   C++ 函数：`stringFromJNI()，该函数返回一个字符串：”Hello from`

   C++”。

2. **External Build Files** 目录是存放 CMake 或 ndk-build

   构建脚本的地方。有点类似于 build.gradle 文件告诉 Gradle 如何编译你的

   APP 一样，CMake 和 ndk-build 也需要一个脚本来告知如何编译你的 native

   library。对于一个新的项目，Android Studio 创建了一个 CMake

   脚本：**CMakeLists.txt**，并且将其放到了你的 module 的根目录下。




### 编译运行示例 APP {#编译运行示例-APP}




当你点击 **Run** 按钮，Android Studio 会编译并启动一个 APP ，然后在 APP  

中显示一段文字”Hello from C++”。从编译到运行示例 APP  

的流程简单归纳如下：




1. Gradle 调用外部构建脚本，也就是 **CMakeLists.txt**。

2. CMake 会根据构建脚本的指令去编译一个 C++ 源文件，也就是

   `native-lib.cpp，并将编译后的产物扔进共享对象库中，并将其命名为`

   **libnative-lib.so**，然后 Gradle 将其打包到 APK 中。

3. 在运行期间，APP 的 MainActivity 会调用 `System.loadLibrary()`

   方法，加载 native

   library。而这个库的原生函数，`stringFromJNI()，就可以为 APP`

   所用了。

4. MainActivity.onCreate\(\) 方法会调用 `stringFromJNI()，然后返回`

   “Hello from C++”，并更新 TextView 的显示。




> 注意：**Instant Run** 并不兼容使用了 native code 的项目。Android  

> Studio 会自动禁止 **Instant Run** 功能。




如果你想验证一下 Gradle 是否将 native library 打包进了 APK，你可以使用  

APK Analyzer:




1. 选择 **Build &gt; Analyze APK**。

2. 从 `app/build/outputs/apk/ 路径中选择 APK，并点击`**`OK`**`。`

3. 如下图，在 APK Analyzer 窗口中，选择 _**lib/&lt;ABI&gt;/**_，你就可以看见

   `libnative-lib.so 。`




![Figer-3.png-58.5kB](http://static.zybuluo.com/wl9739/ei2c0h7dftdi8tp9nuafvt30/Figer-3.png)




## 将 C/C++ 代码添加到现有的项目中 {#将-C-C-代码添加到现有的项目中}




如果你想将 native code 添加到一个现有的项目中，请按照下面的步骤操作：




1. 创建新的 native source 文件，并将其添加到你的 Android Studio

   项目中。如果你已经有了 native code，也可以跳过这一步。

2. 创建一个 CMake 构建脚本。如果你已经有了一个 CMakeLists.txt

   构建脚本，或者你想使用 ndk-build 然后有一个 Android.mk

   构建脚本，也可以跳过这一步。

3. 将你的 native library 与 Gradle 关联起来。Gradle

   使用构建脚本将源码导入到你的 Android Studio 项目中，并且将你的

   native library （也就是 .so 文件）打包到 APK 中。




一旦你配置好了项目，你就可以在 Java 代码中，使用 JNI  

框架开调用原生函数（native functions）。只需要点击 **Run**  

按钮，就可以编译运行你的 APP 了。




### 创建新的 native source 文件




请按照下面的方法来创建一个 `cpp/ 目录和源文件（native source files）：`




1. 打开IDE左边的 **Project** 面板，选择 **Project** 视图。

2. 找到你项目的 **module &gt; src**目录，右键点击 **main** 目录，选择

   **New &gt; Directory**。

3. 输入目录的名字（比如 cpp），然后点击 **OK**。

4. 右键点击刚才创建好的目录，选择 **New &gt; C/C++ Source File**。

5. 输入文件名，比如 `native-lib。`

6. 在 **Type** 菜单下拉选项中，选择源文件的扩展后缀名，比如 `.cpp。`

7. 如果你也想创建一个头文件，点击 **Create an associated header**

   选项框。

8. 点击 **OK**。




### 创建 CMake 构建脚本 {#创建-CMake-构建脚本}




如果没有一个 CMake 构建脚本，你需要自己手动创建一个，并添加一些合适的  

CMake 命令。CMake 构建脚本是一个空白的文本文档（后缀为 .txt  

的文件），名字必须为 `CMakeLists.txt。`




> 注意：如果你的项目使用了 ndk-build，你就不需要创建 CMake  

> 构建脚本，只需要提供一个路径链，将你的 Android.mk 文件链接到 Gradle  

> 中即可。




将一个空白的文本文档变成一个 CMake 构建脚本，你需要这么做：




1. 打开 IDE 左边的 **Project** 面板，选择 **Project** 视图。

2. 在你的 module 根目录下，右键，选择 **New &gt; File**。

3. 输入 “CMakeLists.txt” 作为文件名，并点击 **OK**。




现在，你可以添加 CMake 命令来配置你的构建脚本了。为了让 CMake  

将源代码（native source code）编译成 native  

library。需要在编译文件中添加 `cmake_minimum_required() 和      

add_library() 命令：`




```

# Sets the minimum version of CMake required to build your native library.

# This ensures that a certain set of CMake features is available to

# your build.




cmake_minimum_required(VERSION 3.4.1)




# Specifies a library name, specifies whether the library is STATIC or

# SHARED, and provides relative paths to the source code. You can

# define multiple libraries by adding multiple add.library() commands,

# and CMake builds them for you. When you build your app, Gradle

# automatically packages shared libraries with your APK.




add_library( # Specifies the name of the library.

             native-lib




            # Sets the library as a shared library.

             SHARED




            # Provides a relative path to your source file(s).

            src/main/cpp/native-lib.cpp )

```




当你使用 `add_library()，将一个源文件（source file）或库添加到你的      

CMake 构建脚本，同步你的项目，然后你会发现 Android studio      

将关联的头文件也显示了处理。然而，为了让 CMake      

在编译时期能定位到你的头文件，你需要在 CMake 构建脚本中添加      

include_directories() 命令，并指定头文件路径：`




```

add_library(...)




# Specifies a path to native header files.

include_directories(src/main/cpp/include/)

```




然后，按照约定，CMake 会将生成的 library 命名为下面的形式：




`lib*library-name*.so`




比如，如果你在构建脚本中，将 library 命名为 “native-lib”，那么 CMake  

会创建叫 `libnative-lib.so 的文件。但是，当你将 library 加载到 Java      

代码中的时候， 你需要使用在 CMake 中指定的名称：`




```

static {

        System.loadLibrary(“native-lib”);

}

```




> 注意：如果你将 CMake 脚本里面的 library  

> 重命名了，或者移除了。你需要清理一下你的工程。在 IDE 的菜单栏中选择  

> **Build &gt; Clean Project**。




Android Studio 会在 Project 面板中的 cpp  

目录中自动添加源文件和头文件。你可以多次使用 `add_library()      

命令，来添加额外的 library。`




### 添加 NDK APIs {#添加-NDK-APIs}




Android NDK 提供了一些有用的 native APIs。将 NDK librarys 添加到  

CMakeLists.txt 脚本文件中，就可以使用这些 API 了。




预编译的 NDK librarys 已经存在在 Android  

平台中了，所以你不需要编译它们，或者是将其打包到你的 APK 中。因为这些  

NDK librarys 已经是 CMake 搜索路径的一部分，你甚至不需要提供你本地安装的  

NDK 路径。你只需要向 CMake 提供你想使用的 library 名字。




将 `find_library() 命令添加到你的 CMake 构建脚本中，这样就可以定位 NDK      

library      

的位置，并将其位置存储在一个变量之中。你可以在构建脚本的其他地方使用这个变量，来代指      

NDK library。下面的示例代码将 Android-specific log support library      

的位置存储到变量 log-lib 中：`




```

find_library( # Defines the name of the path variable that stores the

              # location of the NDK library.

              log-lib




              # Specifies the name of the NDK library that

              # CMake needs to locate.

              log )

```




NDK 同样也包含一些只包含源码的  

library，这些就需要你去编译，然后链接到你的本地库（native  

library）。你可以在 CMake 构建脚本中使用 `add_library()      

命令将源码编译进本地库。这时就需要提供你的本地 NDK      

安装路径，通常将该路径保存在 ANDROID_NDK 变量中，这样 Android Studio      

可以自动为你识别。`




下面的命令告诉 CMake 去构建  

`android_native_app_glue.c，这个命令可以管理 NativeActivity      

的生命周期以及点击输入，并将其导入静态库中，然后将其链接至      

native-lib：`




```

add_library( app-glue

             STATIC

             ${ANDROID_NDK}/sources/android/native_app_glue/android_native_app_glue.c )




# You need to link static libraries against your shared native library.

target_link_libraries( native-lib app-glue ${log-lib} )

```




### 添加其他的预编译库




添加预编译库和添加本地库（native  

library）类似。由于预编译库是已经构建好的，你想就要使用 `IMPORTED      

标志去告诉 CMake ，你只需要将其导入到你的项目中即可：`




```

add_library( imported-lib

             SHARED

             IMPORTED )

```




然后你需要使用 `set_target_properties()      

命令去指定库的路径，就像下面的代码那样。`




一些库会根据不同的 CPU 使用不同的包，或者是  

`Application Binary Interfaces(ABI)，并且将他们归类到不同的目录中。这样做的好处是，可以充分发挥特定的      

CPU 架构。你可以使用 ANDROID_ABI 路径变量，将多个 ABI      

版本的库添加到你的 CMake 构建脚本中。这个变量使用了一些 NDK 默认支持的      

ABI，以及一些需要手动配置到 Gradle 的 ABI，比如：`




```

add_library(...)

set_target_properties( # Specifies the target library.

                       imported-lib




                       # Specifies the parameter you want to define.

                       PROPERTIES IMPORTED_LOCATION




                       # Provides the path to the library you want to import.

                       imported-lib/src/${ANDROID_ABI}/libimported-lib.so )

```




为了让 CMake 在编译时期能找到你的头文件，你需要使用  

`include_directories() 命令，并且将你的头文件地址传进去：`




```

include_directories( imported-lib/include/ )

```




在 CMake 构建脚本中使用 `target_link_libraries()      

命令，将预构建库与你本地库相关联：`




```

target_link_libraries( native-lib imported-lib app-glue ${log-lib} )

```




当你构建你的 APP 的时候，Gradle 会自动将导入的库打包到你的 APK  

中。你可以使用 APK Analyzer 来检查。




## 关联本地库与 Gradle {#关联本地库与-Gradle}




为了将本地库与 Gradle 相关联，你需要在 CMake 或 ndk-build  

构建脚本中提供一个路径地址。当你构建你的 APP 时，Gradle 会将 CMake 或  

ndk-build 作为一个依赖运行，然后将共享库（.so 文件）打包到你的 APK  

中。Gradle 同样使用构建脚本来识别哪些文件需要导入到 Android Studio  

项目，你可以从 **Project** 窗口面板中看到相应的文件。如果你还没有一个为  

native sources 准备的构建脚本，你需要先创建一个。




### 使用 Android Studio 图形化界面 {#使用-Android-Studio-图形化界面}




你可以使用 Android Studio 的图形化界面来将 Gradle 与外部 CMake 或者  

ndk-build 项目关联起来。




1. 打开 IDE 左边的 **Project** 面板，选择 **Android** 视图。

2. 右键点击你想链接本地库的 module，比如 **app**

   module，然后从菜单中选择 \*\*Link C++ Project with

   Gradle\*\*。你应该能看见一个和下图很像的对话框。

3. 在下拉菜单中，选择 **CMake** 或者 **ndk-build**。\  

    a. 如果你选择 **CMake**，需要在 **Project Path** 中指定  

   `CMakeLists.txt 脚本文件的路径。\      

    b. 如果你选择`**`ndk-build`**`，你需要在`**`Project Path`**`中指定      

   Android.mk 脚本文件的路径。`




   ![Figure
   3.jpg-86.4kB](http://static.zybuluo.com/wl9739/q80ozydc9jw0ktk8quobcmq3/Figure%203.jpg)




4. 点击 **OK**。







### 手动配置 Gradle {#手动配置-Gradle}




如果要手动将 Gradle 与你的本地库相关联，你需要在 module 层级的  

build.gradle 文件中添加 `externalNativeBuild {}      

代码块，并且在该代码块中配置 cmake {} 或 ndkBuild {}：`




```

android {

  ...

  defaultConfig {...}

  buildTypes {...}




  // Encapsulates your external native build configurations.

  externalNativeBuild {




    // Encapsulates your CMake build configurations.

    cmake {




      // Provides a relative path to your CMake build script.

      path "CMakeLists.txt"

    }

  }

}

```




#### 可选配置




你可以在你的 module 层级的 build.gradle 文件中的 `defaultConfig {}      

代码块中，添加 externalNativeBuild {} 代码块，为 CMake 或 ndk-build      

配置一些额外参数。当然，你也可以在你的构建配置中的其他每一个生产渠道重写这些属性。`




比如，如果你的 CMake 或者 ndk-build  

项目中定义了多个本地库，你想在某个生产渠道使用这些本地库中的几个，你就可以使用  

`targets 属性来构建和打包。下面的代码展示了一些你可能会用到的属性：`




```

android {

  ...

  defaultConfig {

    ...

    // This block is different from the one you use to link Gradle

    // to your CMake or ndk-build script.

    externalNativeBuild {




      // For ndk-build, instead use ndkBuild {}

      cmake {




        // Passes optional arguments to CMake.

        arguments "-DCMAKE_VERBOSE_MAKEFILE=TRUE"




        // Sets optional flags for the C compiler.

        cFlags "-D_EXAMPLE_C_FLAG1", "-D_EXAMPLE_C_FLAG2"




        // Sets a flag to enable format macro constants for the C++ compiler.

        cppFlags "-D__STDC_FORMAT_MACROS"

      }

    }

  }




  buildTypes {...}




  productFlavors {

    ...

    demo {

      ...

      externalNativeBuild {

        cmake {

          ...

          // Specifies which native libraries to build and package for this

          // product flavor. If you don't configure this property, Gradle

          // builds and packages all shared object libraries that you define

          // in your CMake or ndk-build project.

          targets "native-lib-demo"

        }

      }

    }




    paid {

      ...

      externalNativeBuild {

        cmake {

          ...

          targets "native-lib-paid"

        }

      }

    }

  }




  // You use this block to link Gradle to your CMake or ndk-build script.

  externalNativeBuild {

    cmake {...}

    // or ndkBuild {...}

  }

}

```




#### 指定 ABI {#指定-ABI}




一般情况下，Gradle 会将你的本地库构建成 `.so 文件，然后将其打包到你的      

APK 中。如果你想 Gradle 构建并打包某个特定的 ABI 。你可以在你的 module      

层级的 build.gradle 文件中使用 ndk.abiFilters 标签来指定他们：`




```

android {

  ...

  defaultConfig {

    ...

    externalNativeBuild {

      cmake {...}

      // or ndkBuild {...}

    }




    ndk {

      // Specifies the ABI configurations of your native

      // libraries Gradle should build and package with your APK.

      abiFilters 'x86', 'x86_64', 'armeabi', 'armeabi-v7a',

                   'arm64-v8a'

    }

  }

  buildTypes {...}

  externalNativeBuild {...}

}

```




大多数情况，你只需要像上面的代码那样，在 `ndk {} 代码块中指定      

abiFilters 即可。如果你想控制 Gradle      

构建、依赖你希望的东西，你就需要在      

defaultConfig.externalNativeBuild.cmake {} 代码块或      

defaultConfig.externalNativeBuild.ndkBuild {} 代码块中，配置其他的      

abiFilters 标签。Gradle 会构建这些 ABI 配置，但是只会将      

defaultConfig.ndk {} 代码块中指定的东西打包到 APk 中。`


  [1]: http://upload-images.jianshu.io/upload_images/1132780-b929e473b22bba72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240