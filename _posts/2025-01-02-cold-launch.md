---
layout: post
title: Android冷启动那些事
date: 2025-01-02 11:21 +0800
last_modified_at: 2025-1-2 11:4 +0800
tags: [性能, 冷启动]
toc: true
---

# Android冷启动那些事

## 冷启动流程
当用户点击 Android 桌面图标到应用启动，主要经过以下过程：
1. 点击图标后的系统响应
   - 1. Launcher（桌面应用）检测
	 	* 当用户点击桌面图标时，首先是 Launcher（安卓系统中的桌面应用）接收到这个触摸事件。Launcher 本身也是一个 Android 应用，它负责管理桌面图标和小部件等元素。
	 	* 它会根据被点击图标对应的应用信息（包名和类名）来确定要启动的目标应用。例如，如果用户点击的是微信图标，Launcher 就会获取微信应用的包名（如 com.tencent.mm）和启动主 Activity 的类名。
   - 2. 系统进程介入
		* Launcher 通过系统的 Binder IPC（进程间通信）机制向系统的 Activity Manager Service（AMS）发送一个启动应用的请求。AMS 是 Android 系统中一个非常重要的服务，它负责管理系统中所有的Activity（安卓应用的一个组件，用于实现用户界面）。
		* AMS 接收到请求后，会进行一系列的检查和准备工作。它会首先检查要启动的应用是否已经存在于内存中，如果存在，就会判断该应用是否处于可以直接启动的状态；如果不存在，就需要从存储设备（如闪存）中加载应用相关的代码和资源。
   - 3. 权限检查
		* AMS 会检查启动这个应用是否需要特定的权限。例如，有些应用可能需要访问用户的通讯录或者相机权限才能正常启动部分功能。如果缺少必要的权限，系统可能会提示用户授予权限，或者根据应用的设置和系统策略直接禁止应用启动。
2. 应用进程的创建和初始化（如果应用未运行）
   - 1. 进程创建
		* 如果应用没有在运行，AMS 会通过 Linux 的 fork 系统调用创建一个新的进程来运行这个应用。这个新进程会拥有自己独立的内存空间，用于加载应用的代码、数据和资源。
		* 新进程会加载 Android 运行时环境（ART），ART 是 Android 用于执行应用程序字节码的虚拟机。它会负责将应用的 dex 文件（安卓应用的字节码文件格式）编译成机器码，以便在设备的处理器上高效运行。
   - 2. 应用组件初始化
		* 应用进程会初始化一些重要的组件。首先是加载 Application 类，这是安卓应用的全局类，用于在应用启动时进行一些全局的初始化工作，如初始化全局变量、配置文件读取等。
		* 然后会根据 Launcher 传递过来的信息，找到并实例化要启动的主 Activity（安卓应用中用于展示用户界面的组件）。这个主 Activity 是应用的入口点，用户在应用中看到的第一个界面通常就是由这个主 Activity 创建的。
3. Activity 的启动和渲染
   - 1. Activity 生命周期回调启动阶段
		* 当主 Activity 被实例化后，系统会调用一系列的 Activity 生命周期方法。首先是 onCreate 方法，在这个方法中，开发者通常会进行一些界面初始化的操作，如设置布局文件（通过 setContentView 方法加载 XML 布局文件，这些布局文件定义了界面上的各种视图元素，如按钮、文本框等的位置和样式）、初始化视图组件等。
		* 接着会调用 onStart 和 onResume 方法。onStart 方法表示 Activity 开始对用户可见，onResume 方法表示 Activity 已经获取焦点，可以和用户进行交互。例如，在这两个阶段，可能会进行一些动画的启动或者传感器的注册等操作，以便为用户提供完整的交互体验。
   - 2. 视图渲染和显示
	 	* 在 Activity 的生命周期方法执行过程中，系统会根据布局文件中的定义，将各种视图元素（如按钮、文本框、图片等）加载到内存中，并进行测量（measure）、布局（layout）和绘制（draw）操作。
	 	* 测量操作会确定每个视图元素的大小，布局操作会确定它们在屏幕上的位置，绘制操作则会将这些视图元素的图形绘制到屏幕上的缓冲区。最后，系统会将缓冲区的内容显示到屏幕上，用户就可以看到应用的启动界面了。

其中如果进程已经被初始化过的话第二大步会被省略，这种情况又被称为温启动
![启动流程分类](https://github.com/Charles199310/Charles199310.github.io/blob/main/assets/images/cold-launch-1.png?raw=true)

第一阶段主要是系统工作，我们不太好介入。我们需要重点考虑的是第二三阶段
![Androidl冷启动流程](https://github.com/Charles199310/Charles199310.github.io/blob/main/assets/images/cold-launch-2.png?raw=true)

## 冷启动优化方法
Android 冷启动优化的手段包括以下几种：
1. **优化冷启动窗口**：优化冷启动窗口可以优化用户在视觉上的体验
	1. 确保`android:windowDisablePreview`为`true`。
	2. 定制自己的闪屏页，通过设置windowbackground的方法显示在启动页的背景上。
	3. 使用`SplashScreen`接口使页面启动更丝滑。
	4. 如果有可能合并闪屏页和Main页面。
2. **减少APK大小**: Android首次冷启动过程中包含了APK安装的过程。APK大小这一个过程越快。其中减少APK大小的手段包括并不限于：混淆，插件化等。
3. **简化视图结构**： 创建试图的时候，尽可能的使用简单的布局结构，使用`ViewStub`等标签优化布局，降低视图创建所需要的时间
4. **Application和闪屏Activity的onCreate中异步初始化某些代码**： 对于冷启动过程中任何代码，应该尽可能放在异步线程里（注意：要合理使用线程池，避免线程过多导致主线程得不到CPU资源）
5. **懒加载一些不必要的引擎**： 一些引擎并不需要在冷启动时启动，一些引擎可以放在idelhandler中, 这样可以避免大量引擎集中启动。
6. **系统调度优化**：避免在冷启动时启动子进程，避免在冷启动时跨进程通信（包括AMS等系统进程），避免同时启动多个四大组件（必要时使用startActivities代替多次startActivity()）
7. **IO优化**： 在启动过程中尽可能的避免IO操作，必要时使用磁盘IO操作代替网络IO操作。
8. **GC优化**： 池化冷启动过程中需要对象，使用缓存池中对象避免频繁发生GC。
9. **避免Verify Class**: Verify Class是是加载过程中的一个步骤，通过hook关闭这一步骤可以提高类加载的速度。
10. **资源重排**： 阿里巴巴通过定制化Android Rom, 获取到Android初始化过程中获取资源的顺序，给APK中的资源进行重排序，从而避免Android启动过程中频繁的磁盘IO。
11. **类重排**： 通过重写classloader，获取类读取的顺序，使用Facebook的Redex对dex文件中的字节码进行重排序，从而避免Android启动过程中频繁的磁盘IO。
12. **使用Jetpack Startup合并ContentProvider**： Startup是Jetpack提供的可以合并ContentProvider的组件，从而提升启动时间。
13. **合理梳理业务逻辑**: 通过梳理业务逻辑间的关系，合理的使不同的业务之间串并行，提高业务初始化效率。
14. **保活**： 保活可以节省fork应用进程的时间，也可以节省部分系统初始化的时间。
15. **合理使用加固技术**：加固技术会在第一次安装app的时候给APP进行解密。使用加固技术的时候要充分考虑其对冷启动的影响。
16. **合理使用第三方库**：使用第三方库的时候要仔细考察其对冷启动的影响。需要注意有一些第三方代码会通过ContentProvider无侵入性地启动自己的引擎。