- [开始使用gstreamer](#开始使用gstreamer)
  - [分析](#分析)


# 开始使用gstreamer

要学习一个多媒体库，没有什么比播放一段视频更能快速了解一个软件库的了！  
不要被下面代码的数量吓到：真正起作用的只有4行。其余部分都是清理代码，在C语言中，这些代码通常会显得有些冗长。  
废话少说，准备开始你的第一个GStreamer应用程序吧……

```C
#include <gst/gst.h>

#ifdef __APPLE__
#include <TargetConditionals.h>
#endif

int
tutorial_main (int argc, char *argv[])
{
  GstElement *pipeline;
  GstBus *bus;
  GstMessage *msg;

  /* Initialize GStreamer */
  gst_init (&argc, &argv);

  /* Build the pipeline */
  pipeline =
      gst_parse_launch
      ("playbin uri=https://gstreamer.freedesktop.org/data/media/sintel_trailer-480p.webm",
      NULL);

  /* Start playing */
  gst_element_set_state (pipeline, GST_STATE_PLAYING);

  /* Wait until error or EOS */
  bus = gst_element_get_bus (pipeline);
  msg =
      gst_bus_timed_pop_filtered (bus, GST_CLOCK_TIME_NONE,
      GST_MESSAGE_ERROR | GST_MESSAGE_EOS);

  /* See next tutorial for proper error message handling/parsing */
  if (GST_MESSAGE_TYPE (msg) == GST_MESSAGE_ERROR) {
    g_printerr ("An error occurred! Re-run with the GST_DEBUG=*:WARN "
        "environment variable set for more details.\n");
  }

  /* Free resources */
  gst_message_unref (msg);
  gst_object_unref (bus);
  gst_element_set_state (pipeline, GST_STATE_NULL);
  gst_object_unref (pipeline);
  return 0;
}

int
main (int argc, char *argv[])
{
#if defined(__APPLE__) && TARGET_OS_MAC && !TARGET_OS_IPHONE
  return gst_macos_main ((GstMainFunc) tutorial_main, argc, argv, NULL);
#else
  return tutorial_main (argc, argv);
#endif
}
```
> 注意 编译时候需要使用pkg-config 找出gstreamer的依赖
> ```gcc basic-tutorial-1.c -o basic-tutorial-1 `pkg-config --cflags --libs gstreamer-1.0```


## 分析
```
gst_init (&argc, &argv);
```
用于初始化gstreamer内部环境，检查插件，**凡是使用gstreamer 都必须先调用此API**

```C
pipeline =
      gst_parse_launch
      ("playbin uri=https://gstreamer.freedesktop.org/data/media/sintel_trailer-480p.webm",
      NULL);
```
这个代码是这整个代码的核心，这里涉及2个关键点：`gst_parse_launch`和 playbin

`gst_parse_launch`：

gstreamer 作为一个多媒体框架，对于各个视频源，播放窗口，以及中间各种scale, rotate等都被是为 `element`,
那么我们用gstreamer播放一段视频，就要涉及视频输入，分辨率转换，输出等几个步骤，把这几个步骤组合在一起，
这个组合就是 `pipeline`。

可想而知，要是利用gstreamer播放一段视频，如果都要自己手动组合这些`element`，会非常复杂，而如果我们就只想播放一段视频，
其他什么都不做，那么`gst_parse_launch`就可以帮你一行代码播放视频

`playbin`：

playbin是一个特殊的pipeline，它里面就已经包含了播放视频所需要必要的`element`。
playbin这么方便，但是它也有缺点：我们无法精确控制整个`pipeline`中的各个`element`


```C
gst_element_set_state (pipeline, GST_STATE_PLAYING);
```
顾名思义，用于设置`pipeline`的状态，跟播放器的播放/暂停一样

```C
bus = gst_element_get_bus (pipeline);
  msg =
      gst_bus_timed_pop_filtered (bus, GST_CLOCK_TIME_NONE,
      GST_MESSAGE_ERROR | GST_MESSAGE_EOS);
```
这里作用是阻塞代码流程，具体功能后续再详细写

```C
gst_message_unref (msg);
gst_object_unref (bus);
gst_element_set_state (pipeline, GST_STATE_NULL);
gst_object_unref (pipeline);
```

这些都是些销毁函数，无需多讲