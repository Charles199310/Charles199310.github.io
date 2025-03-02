---
# layout: post
title: Android 帧率优化那些事
date: 2025-02-23 24:58 +0800
last_modified_at: 2025-03-02 24:58 +0800
tags: [性能, 帧率]
toc: true
---
# Android 帧率优化那些事
帧率，即FPS（每秒帧数），在Android中通常希望维持在60fps，这意味着每帧大约有16毫秒的处理时间。如果超过这个时间，就会导致掉帧，用户会觉得卡顿。所以维护帧率的关键在于确保每帧的处理时间不超过16ms。
## FrameMetricsListener
`FrameMetricsListener` 是 Android 提供的一个用于监控应用帧率性能的工具，它可以帮助开发者精确测量每一帧的渲染时间，从而定位性能瓶颈。以下是示例代码，
```Java
public class MainActivity extends AppCompatActivity {
    private Window.OnFrameMetricsAvailableListener listener;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        listener = new Window.OnFrameMetricsAvailableListener() {
            @Override
            public void onFrameMetricsAvailable(
                Window window, FrameMetrics frameMetrics, int dropCountSinceLastInvocation) {
                long totalDuration = frameMetrics.getMetric(FrameMetrics.TOTAL_DURATION);
                double fps = 1_000_000_000.0 / totalDuration;
                Log.d("FrameMetrics", "FPS: " + fps);
            }
        };

        // 注册监听器
        getWindow().addOnFrameMetricsAvailableListener(listener, new Handler());
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        // 移除监听器
        getWindow().removeOnFrameMetricsAvailableListener(listener);
    }
}
```
## Android帧率优化方式
1. 异步处理耗时任务，减小主线程压力
2. 布局优化
   1. 减少视图层级，例如`ConstraintLayout`代替多层嵌套的`LinearLayout`
   2. 使用`<include>`标签复用公共布局, 使用`<merge>`标签减少根视图冗余
3. 按需加载
   1. 使用`ViewStub`延迟加载不立即显示的视图。
   2. 使用`RecyclerView`代替`ListView`和`ScrollView`，动态加载列表项。
4. 减少过度绘制
5. 自定义View优化
   1. 避免在`onDraw()`中创建对象或执行耗时操作。
   2. 使用高效的绘制方法例如避免使用`drawPath()`
   3. 使用`canvas.clipRect()`限制绘制区域，避免不可见部分绘制。
   4. 考虑硬件加速（`setLayerType(LAYER_TYPE_HARDWARE)`）
6. 使用硬件加速
   1. 在`AndroidManifest.xml`中启用硬件加速：
  ```xml
  <application android:hardwareAccelerated="true" />
  ```
   2. 对于特定视图，使用`setLayerType(LAYER_TYPE_HARDWARE)`。
7. 优化`RecycleView`
   1. 使用`DiffUtil`高效更新数据集，避免`notifyDataSetChanged()`。
   2. 启用预加载（`setItemViewCacheSize()`）和固定大小（`setHasFixedSize(true)`）。
   3. 简化`ViewHolder`的绑定逻辑。
8. 优化动画
   1. 优先使用`ValueAnimator`和`ObjectAnimator`，避免补间动画。
   2. 在`onPause()`或`onStop()`中及时暂停动画
   3. 减少同时运行的动画数量。
   4. 避免在动画期间触发重布局。
9. 做好内存管理
10. 使用Jetpack Compose高效渲染视图
    1. 采用声明式UI框架，通过`@Composable`函数和`remember`减少重复计算。
    2. 使用`Modifier`链式调用优化布局性能。
11. 使用SurfaceView或TextureView播放视频或游戏