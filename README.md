##### 矢量图VectorDrawable实现搜索框轨迹动画
**SVG和Vector的差异**
- SVG——前端中使用，是一套语法规范
- Vector——在Android中使用
- Vector只实现了SVG语法的Path标签

**Vector常用语法**
- M = moveto(M X,Y)：将画笔移动到制定的坐标位置
- L = lineto(L X,Y)：画直线到指定的坐标位置
- Z = closepath()：关闭路径
- H = horizontal lineto(H X)：画水平线到指定的X坐标位置
- V = vertical lineto(V Y)：画垂直线到指定的Y坐标位置

#####相关网站
[SVG编辑器](http://editor.method.ac/) <br/>
[SVG转VectorDrawable(需翻墙)](http://inloop.github.io/svg2android/)<br/>
[iconfont](http://www.iconfont.cn)


#####使用VectorDrawable的好处
**三种格式的体积对比**
从下图可以看到从.png到.svg到再到Android可以使用的VectorDrawable体积成倍数减小。而且使用VectorDrawable可以不用考虑缩放，让图像完全保真。
![image.png](https://upload-images.jianshu.io/upload_images/11184437-2484c545396f07bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####生成VectorDrawable
**静态的VectorDrawable使用**
1.项目中`drawable`文件夹右键new>Vector Asset
    生成的xml文件中`android:viewportHeight="1024.0"`(定义图像被划分的比例大小)这个属性代表，把固定大小的矢量图均匀的分成1024等份。后面写 `android:pathData`的时候，就以1024为基线坐标，而不是以具体的大小数值去做图标。这样做的好处是，如果说VectorDrawable大小有变化，我们只需要通过`viewportHeight`去做映射就可以了，而不需要改变`pathData`。
#####项目中使用VectorDrawable
gradle中添加`vectorDrawables.useSupportLibrary = true`
```
<Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:srcCompat="@drawable/ic_home"
        />
```
对于Button这种带有状态的控件，不能直接使用VectorDrawable，但是可以通过使用`<selector>`，来让button控件调用。要注意的是需要在布局文件对应的Activity中添加如下代码：
```
static {
        AppCompatDelegate.setCompatVectorFromResourcesEnabled(true);
    }
```
**动态的VectorDrawable使用**
   配置动画粘合剂——animated-vector：让属性动画作用到VectorDrawable中
```
<animated-vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/arrow">
    <target
        android:animation="@animator/anim_left"    //对应的动画
        android:name="left"/>       //VectorDrawable中group的名字
    <target
        android:animation="@animator/anim_right"
        android:name="right"/>
</animated-vector>
```
VectorDrawable的xml文件中path要用group标签套上，否则效果会出不来
```
<group android:name="left">
        <path
            android:fillColor="#FF000000"
            android:pathData="M9.01,14L2,14v2h7.01v3L13,15l-3.99,-4v3z"/>
    </group>

    <group android:name="right">
        <path
            android:fillColor="#FF000000"
            android:pathData="M14.99,13v-3L22,10L22,8h-7.01L14.99,5L11,9l3.99,4z"/>
    </group>
```
#####实现搜索框与轨迹动画

![](https://upload-images.jianshu.io/upload_images/11184437-159ce3a2a9c2d8d1.gif?imageMogr2/auto-orient/strip)

动画分为横线bar和放大镜search俩部分
需要修改`androd:propertyName="trimPathStart"`属性
`androd:valueFrom="0"
    androd:valueTo="1"`
意思是截取的路径从0%到100%

然后使用动画粘合剂animated-vector即可
```
<objectAnimator
    xmlns:android="http://schemas.android.com/apk/res/android"
    androd:duration="1000"
    android:propertyName="trimPathStart"
    android:repeatCount="infinite"
    android:repeatMode="reverse"
    android:valueFrom="0"
    android:valueTo="1"
    android:valueType="floatType">
</objectAnimator>
```

#####动态VectorDrawable的兼容性
**向下兼容问题**
Path Morphing——路径变换动画，在Android pre-L版本下是无法使用的
Path Interpolation——路径插值器，在Android pre-L版本只能使用系统的插值器，不能自定义
**向上兼容问题**
Path Morphing——路径变换动画，在Android L版本以上需要使用代码配置
**抽取string兼容问题**
不支持从Strings.xml中读取<PahtData>


#####路径动画变换效果
![](https://upload-images.jianshu.io/upload_images/11184437-c8fc432ababc0161.gif?imageMogr2/auto-orient/strip)


修改`androd:propertyName="pathData"`属性，把两个形状的pathData值，放到`android:valueFrom`和`android:valueTo`里面，然后使用动画粘合剂即可。
Android5.0以下的设备，无法使用这个动画。
```
<objectAnimator
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="3000"
    android:propertyName="pathData"
    android:valueFrom="M 48,54 L 31,42 15,54 21,35 6,23 25,23 32,4 40,23 58,23 42,35 z"
    android:valueTo="M 48,54 L 31,54 15,54 10,35 6,23 25,10 32,4 40,10 58,23 54,35 z"
    android:valueType="pathType">
</objectAnimator>
```