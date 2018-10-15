---
title: 常用View与优化
date: 2018-10-10 09:42:16
tags: [Android,view]
categories: Android

---
在Android APP中，所有的用户界面元素都是由View和ViewGroup的对象构成的。View是绘制在屏幕上的用户能与之交互的一个对象。而ViewGroup则是一个用于存放其他View（和ViewGroup）对象的布局容器。Android为我们提供了一个View和ViewGroup子类的集合，集合中提供了一些常用的输入控件(比如按钮和文本域)和各种各样的布局模式（比如线性或相对布局） 
# 一、Android View基础知识 #
你的APP的用户界面上的每一个组件都是使用View和ViewGroup对象的层次结构来构成的，如下图所示。  
每个ViewGroup都是要给看不见的用于组织子View的容器，而它的子View可能是输入控件 或者在UI上绘制了某块区域的小部件。有了层次树，你就可以根据自己的需要，设计简单或者复杂的布局了(布局越简单性能越好)  

![](https://ws1.sinaimg.cn/large/6bbf23f6gy1fw2w2hddttj20d6071dfw.jpg)  

定义你的布局，你可以在代码中实例化View对象并且开始构建你的树，但最容易和最高效的方式来定义你的布局则是使用一个XML文件，用XML来构成布局更加符合人的阅读习惯，而XML类似与HTML 使用XML元素的名称代表一个View。所以<TextView>元素会在你的界面中创建一个TextView控件，而一个<LinearLayout>则会创建一个LinearLayout的容器。举个例子，一个简单简单的垂直布局上面有一个文本视图和一个按钮，就像下面这样：  

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="fill_parent" 
              android:layout_height="fill_parent"
              android:orientation="vertical" >
    <TextView android:id="@+id/text"
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:text="I am a TextView" />
    <Button android:id="@+id/button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="I am a Button" />
</LinearLayout>
```

当你的App加载上述的布局资源的时候，Android会将布局中的每个节点进行实例化成一个个对象，然后你可以为这些定义一些额外的行为，查询对象的状态，或者修改布局。 完整创建UI布局的引导，请参考[XML Layouts](http://androiddoc.qiniudn.com/guide/topics/ui/declaring-layout.html "XML Layouts")  
你无需全部用View和ViewGroup对象来创建你的UI布局。Android给我们提供了一些app控件，标准的UI布局，你只需要定义内容。这些UI组件都有其属性介绍的API文档，本文后续也会介绍一些。  

# 二、常用布局 #
Android中有六大布局,分别是: 

- [LinearLayout(线性布局)](http://www.runoob.com/w3cnote/android-tutorial-linearlayout.html)
- [RelativeLayout(相对布局)](http://www.runoob.com/w3cnote/android-tutorial-relativelayout.html)
- [TableLayout(表格布局) ](http://www.runoob.com/w3cnote/android-tutorial-tablelayout.html)
- [FrameLayout(帧布局)](http://www.runoob.com/w3cnote/android-tutorial-framelayout.html)
- [AbsoluteLayout(绝对布局)](http://www.runoob.com/w3cnote/android-tutorial-absolutelayout.html)
- [GridLayout(网格布局)  ](http://www.runoob.com/w3cnote/android-tutorial-gridlayout.html)

这些布局都是继承与ViewGroup，在开发中使用频率非常高,其中AbsoluteLayout在Android中几乎已经没人使用了，主要原因是AbsoluteLayout几乎无法做到对Android来说很重要的UI适配，不能够自适应各种型号的手机，使用后会导致APP的UI界面很丑。

# 三、常用控件 #
 
以下列出在Android开发过程中常用的一些View

- [TextView](http://www.runoob.com/w3cnote/android-tutorial-textview.html)
- [EditText](http://www.runoob.com/w3cnote/android-tutorial-edittext.html)
- [Button与ImageButton](http://www.runoob.com/w3cnote/android-tutorial-button-imagebutton.html)
- [ImageView](http://www.runoob.com/w3cnote/android-tutorial-imageview.html)
- [RadioButton和CheckBox](http://www.runoob.com/w3cnote/android-tutorial-radiobutton-checkbox.html)
- [ToggleButton和Switch](http://www.runoob.com/w3cnote/android-tutorial-togglebutton-switch.html)
- [ProgressBar](http://www.runoob.com/w3cnote/android-tutorial-progressbar.html)
- [SeekBar](http://www.runoob.com/w3cnote/android-tutorial-seekbar.html)
- [ScrollView](http://www.runoob.com/w3cnote/android-tutorial-scrollview.html)
- [ListView](http://www.runoob.com/w3cnote/android-tutorial-listview.html)与[Adapter](http://www.runoob.com/w3cnote/android-tutorial-adapter.html)
- [RecycleView](https://www.kymjs.com/code/2016/07/10/01/)  

在这其中，ListView是所有控件中最常使用且对刚入门的新手来说最复杂的用法，各种Adapter的使用以及ListView的优化是一定要掌握的。