---
layout: post
title: Android包大小优化那些事
date: 2024-12-28 11:21 +0800
last_modified_at: 2025-1-2 11:4 +0800
tags: [性能, 包大小]
toc: true
---
# Android包大小优化那些事

Android包大小是性能的一个重要指标。在我们开发Android应用中，包体及其优化具有重要意义，主要体现在以下几个方面：

1. **应用安装和下载速度**   
   
   - **更小的包体**: 减小 APK 文件的大小可以降低用户下载和安装应用时所需的时间和数据费用。这对于在网络条件较差或用户流量有限的情况下尤其重要。
   - **提升用户体验**: 快速下载和安装能够提升用户的第一印象，从而提高应用的安装率。

2. **存储空间占用**
   
   - **减少存储占用**: 较小的 APK 文件可以减少设备存储的占用，尤其是在存储空间有限的设备上，高效的资源管理可以帮助用户更好地管理存储。
   - **保留更多空间**: 用户更愿意安装体积较小的应用，以便于在其设备上保留更多的照片、视频等其他数据。

3. **启动和运行性能**
   
   - **优化启动时间**: 减少 APK 大小可能影响首次启动的性能，因为较小的资源可以更快地加载。合适的优化策略能够显著提升应用的启动速度。
   - **流畅性和响应速度**: 包内资源的优化（如图片和其他存档）可以加强应用运行时的性能，使应用运行得更加流畅和迅速。

4. **提高安全性**
   
   - **资源保护**: 通过减少包体的冗余资源，降低了潜在的攻击面。应用不必要的资源可以被安全漏洞利用，因此优化资源可以提高安全性。
   - **加密与混淆**: 在整体优化过程中，资源的加密和代码混淆不仅可以减小 APK 尺寸，还能提高逆向工程的难度，从而保护应用的知识产权和敏感信息。

5. **更好的兼容性**
   
   - **分包**: 使用 APK 分割（如使用 Android App Bundles）可以根据设备硬件选择性地打包适合用户设备的资源和代码，减少应用的实际安装大小，提升兼容性。
   - **支持多种设备**: 为不同设备定制 APK，比如不同的 CPU 架构或屏幕密度，有助于避免冗余资源的打包，并且让应用在不同设备上运行更好。

6. **应用更新和维护**
   
   - **降低更新负担**: 从性能优化和资源管理的角度考虑，较小的应用包可以减少后续版本更新时的负担。这使当前和未来的用户更愿意持续更新应用。
   - **简化维护**: 精简的代码和资源能够简化代码维护过程，提高开发效率。

总结一下，优化 Android 应用的包体不仅是提升用户体验和应用性能的重要措施，还是促使合规、提高安全性、增强兼容性及降低存储压力的重要步骤。在竞争激烈的应用市场中，良好的包体优化策略能够显著提升应用的成功机会。

## 包体构成

![Android包体勾陈](https://github.com/Charles199310/Charles199310.github.io/blob/main/assets/images/package-size-1.png?raw=true)

要想优化Android包体积，我们需要先分析Android包体里面有哪些内容：

1. **assets/**：一个可选的目录，允许你在 APK 中包含原始文件（如 HTML、文本文件、字体等），这些文件可以在应用运行时读取

2. **lib/**: 可选的目录，包含本机库（即使用 C/C++ 编写的共享库），这些库通常以 `.so` 扩展名存放，针对不同的处理器架构分类（如 `armeabi-v7a`, `arm64-v8a`, `x86`, `x86_64` 等）。

3. **META-INF/**: 包含有关 APK 签名和元数据的信息，包括一个或多个数字签名文件。这些文件用于验证 APK 文件的完整性和来源。

4. **res/**: 包括应用中使用的所有资源（图片、布局文件、字符串、样式等）。资源文件通常放置在 `res` 目录下，并根据类型分类，例如：
   
   * `res/layout/`: 应用的布局文件（XML）
   
   * `res/drawable/`: 应用的图像资源
   
   * `res/values/`: 包含字符串和其他值资源（如 dimens.xml、strings.xml、colors.xml 等）
   
   需要注意`res/raw/`文件夹像 **asset/** 文件夹一样装的是原始文件

5. **resources.arsc**: 编译后的资源文件，包含所有应用可用的资源 ID 和对应的资源数据，以便在运行时能够快速访问。

6. **仿真和格式的文件**：一些 APK 可能包含特定于 Android 的格式或文件，如 ProGuard 生成的文件，这些文件在应用构建时生成。

7. **AndroidManifest.xml**: APK 的核心文件，定义了应用的基本信息，如应用的包名、组件（Activity、Service、BroadcastReceiver、ContentProvider）、权限、版本号以及其他应用的配置信息

8. **classes.dex**: 编译后的字节码文件，包含了应用的 Java 代码。APK 中的所有 Java 类都会被编译成 DEX 格式，以便在 Android 的 Dalvik 或 ART 虚拟机中运行。

其中， **classes.dex**, **res/**, **lib/**, **assets/**, 是我们做包体积优化的重点。**仿真和格式的文件**可能根据自身业务特点也有文章可以做。

## 包大小优化的手段

1. ### Classes.dex
   
   Android项目中的Java和Kotlin代码在编译阶段会被J**ava编译器**（javac）和**kotlin编译器**编译为`.class`文件，在处理完`.class`文件文件之间的依赖关系后，**DX**(Dalvik Executable）工具来将 `.class` 文件转换为 `.dex` 文件。  
   
   优化Classes.dex优化的思路有两个：
   
   1. #### 减少**Kotlin**和**Java**代码数量
      
      1. **复用代码**。对于相同的功能，相同的代码吗，如果出现超过3次，我们就需要考虑是否可以抽象出一个接口，实现代码复用了。
      
      2. **谨慎引用第三方库**。当需要应用第三方库时，我们要尽可能的在满足业务需求，保证稳定性的情况下，考虑使用大小较小的包。尽可能避免引入相似工呢的包。如果可以的话可以通过自己改造三方库，减少第三方库的大小（比如自己重构开源库，或者通过groovy剔除不需要的依赖）。
         
         ```groovy
         dependencies {
             implementation('some-important-but-large-library') {
                 exclude group: 'com.example.imgtools', module: 'native'
             }
         }
         ```
      
      3. **移除不需要的代码**。在我们使用混淆代码功能时，可以自动移除掉无用的class文件，但是由于四大组件会在`AndroidManifest.xml`文件中声明，所以对于无用的四大组件，需要我们手动去删除。另外如果代码中有大量不会用到的代码也不利于我们后续的维护，

2. #### 减少**Kotlin**和**Java**里面的信息，使最终生成的`.class`和`.dex`文件变小
   
   1. **使用代码混淆**。使用代码混淆可以将我们的类名、包名、方法名等信息隐藏掉，从而减小最终生成的`.dex`文件大小。并且增加了反编译难度，增强了代码安全性。具体做法需要在`app/build.gradle`文件中打开代码混淆功能
      
      ```groovy
      android {
          buildTypes {
              release {
                  minifyEnabled true
                  proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.p'
              }
          }
      }               
      ```
      
      如果有像反射，反序列化等情况，可以通过Proguard文件尽可能的缩小keep代码的范围。
   
   2. **缩小注解的声明周期**。 在 Kotlin 中，`AnnotationRetention` 是一个用于指定注解保留策略的枚举类。
      
      ```java
      /**
       * Contains the list of possible annotation's retentions.
       *
       * Determines how an annotation is stored in binary output.
       */
      public enum class AnnotationRetention {
          /** Annotation isn't stored in binary output */
          SOURCE,
          /** Annotation is stored in binary output, but invisible for reflection */
          BINARY,
          /** Annotation is stored in binary output and visible for reflection (default retention) */
          RUNTIME
      }
      ```
      
      其中`SOURCE`可以避免注解被打包到`.dex`文件中
   
   3. **避免使用枚举**。单个枚举会使应用的 `classes.dex` 文件增加大约 1.0 到 1.4KB。这些增加的大小会快速累积，产生复杂的系统或共享库。如果可能，请考虑使用 `@IntDef` 注解和[代码缩减](https://developer.android.com/studio/build/shrink-code?hl=zh-cn)移除枚举并将它们转换为整数。

2. ### **`lib/`**
   
   `lib/`目录下主要用于存放不同的`.so`文件（二进制文件）。不同的`.so`文件用来支持不同的Android设备ABI架构（Application Binary Interface）。Android设备支持以下七种ABI：**armeabi、armeabi-v7a、arm64-v8a、x86、x86_64、mips、mips64**。
   
   | CPU（纵向）/ABI（横向） | armeabi | armeabi-v7a | arm64-v8a | x86   | x86_64 | mips  | mips64 |
   | --------------- | ------- | ----------- | --------- | ----- | ------ | ----- | ------ |
   | ARMv5           | 支持(1)   |             |           |       |        |       |        |
   | ARMv7           | 支持(2)   | 支持(1)       |           |       |        |       |        |
   | ARMv8           | 支持(3)   | 支持(2)       | 支持(1)     |       |        |       |        |
   | x86             | 支持(3)   | 支持(2)       |           | 支持(1) |        |       |        |
   | x86_64          | 支持(4)   | 支持(3)       |           | 支持(2) | 支持(1)  |       |        |
   | MIPS            |         |             |           |       |        | 支持(1) |        |
   | MIPS64          |         |             |           |       |        | 支持(2) | 支持(1)  |
   
   其中**MIPS、MIPS64**设备很少可以不考虑，**x86、x86_64**设备主要是虚拟机设备，**ARMv5**设备较老，目前市场上**ARMV7、ARMV8**设备占比较大可以结合自身业务服务对象，发布的渠道， 以及ABI支持的设备选择最少的ABI。需要注意的是，虽然有些ABI虽然除了可以支持自己的CPU架构还可以支持其他CPU架构，但是需要以损失性能为代价。
   
   另外如果是使用google的aab进行发布，由于其自身有根据设备选择ABI的能力所有不太需要考虑ABI架构选择
   
   除了选择合适的ABI架构外，针对于自己开发的`.so`文件，在发布前还需要提取符号表。提取符号表的作用类似Java和Kotlin混淆，可以缩小`.so`的大小，而且会使代码更加难以反编译

3. **`res/`**
   
   res文件中保存着大量程序运行时需要的资源，也是我们包大小优化的重点对象之一
   
   1. **手动移除无用的资源**。您的应用可能不再使用已导入到项目中的部分资源。为帮助移除这些资源，Android Studio 针对这种特定情况提供了 Lint 检查。要使用该工具，请完成以下步骤：
      
      1. 按 **Control+Alt+Shift+I**（在 Mac OS 上按 **Command+Alt+Shift+I**）。
      
      2. 在显示的对话框中，输入 `"unused resources"`。
      
      3. 选择 **Unused resources** 选项，启动资源使用情况检查流程。
   
   2. **将资源放在服务端**。如果应用中仍存在大型资源，请考虑是否可以从应用软件包中移走这些资源，并在用户开始与应用互动之后将其作为独立文件下载下来。这种图片加载延迟通常需要更改代码，但由于这种方式只会下载用户明确请求的资源，因此可以大幅缩减免安装应用的文件大小。
   
   3. **选择合适的图片格式**。优先使用矢量图`SVG`转化成的`drawable`文件作为图片格式，其次是`webp`格式的图片，再其次才是`png`图片。使用`webp`和`png`格式的文件时，可以使用`tinypng`等工具压缩一下
   
   4. **选择合适的dpi文件夹**。由于市面上`mdpi`, `hdpi`, 以及`xxxhdpi`设备并不是太多，而且系统在没有对于`dpi`的资源时会使用相近`dpi`资源。可以考虑舍弃对这些市场上不是太多的设备的支持。
   
   5. **使用着色、翻转，组合现有图片等方式，避免添加新图片**
   
   6. 打开 Crunchpngs
      
      ```groovy
          buildTypes.all { isCrunchPngs = false }
      ```
   
   ```
   7. **打开shrinkResources**。打开shrinkResource可以在打包时移除没有使用的资源文件。
   
   ```groovy
   android {
       buildTypes {
           release {
               minifyEnabled true
               proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.p'
               shrinkResources true
           }
       }
   }       
   ```

4. `**assets、res/raw/`**
   
   这两个文件夹里面主要存放原始文件。针对这些文件我们一是需要注意删除不需要的文件，二是可以考虑使用更小的文件达到同样的效果，比如mp4文件我们优先使用h.265格式而不是h.264等格式，再比如音频文件我们可以考虑使用较小的码率等。

5. **使用插件化，组件化等技术**。
   
   使用插件化，组件化的的技术可以将扩展功能放到扩展包里，等用户需要或者空闲的时候再下载，从而实现减小初始包的大小。这方面海外有google的**DynamicFeature**，国内有**DroidPlugin**、**Atlas**等。