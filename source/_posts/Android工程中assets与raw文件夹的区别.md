---
title: Android工程中assets与raw文件夹的区别
date: 2018-11-13 17:34:18
categories: Android
tags: [AndroidStudio,Android]
toc: false
description: Clear thinking requires courage rather than intelligence.
---
  我们都知道Android工程中assets与raw文件夹都可以用来存放文件
比如已经设计好的数据库文件可以选择放到assets中（当然你们也可以放到raw里 ），这样程序在打包时会原封不动的保存到apk包中，不会被编译成二进制。早期android2.3以前的版本有着assets和raw里资源文件不能超过1M的限制，当然现在已经没有这个限制了。
# res/raw和assets的相同点
两者目录下的文件在打包后会原封不动的保存在apk包中，不会被编译成二进制

# res/raw和assets的不同点 
1. res/raw中的文件会被映射到R.java文件中，访问的时候直接使用资源ID即R.id.filename；assets文件夹下的文件不会被映射到R.java中，访问的时候需要AssetManager类。
2. res/raw不可以有目录结构，而assets则可以有目录结构，也就是assets目录下可以再建立文件夹

> - 由于raw是Resources (res)的子目录，Android会自动的为这目录中的所有资源文件生成一个ID，这个ID会被存储在R类当中，作为一个文件的引用。这意味着这个资源文件可以很容易的被Android的类和方法访问到，甚至在Android XML文件中你也可以@raw/的形式引用到它。在Android中，使用ID是访问一个文件最快捷的方式。MP3和Ogg文件放在这个目录下是比较合适的。
> - assets目录更像一个附录类型的目录，Android不会为这个目录中的文件生成ID并保存在R类当中，因此它与Android中的一些类和方法兼容度更低。同时，由于你需要一个字符串路径来获取这个目录下的文件描述符，访问的速度会更慢。但是把一些文件放在这个目录下会使一些操作更加方便，比方说拷贝一个数据库文件到系统内存中。要注意的是，你无法在Android XML文件中引用到assets目录下的文件，只能通过AssetManager来访问这些文件。数据库文件和游戏数据等放在这个目录下是比较合适的。
# 读取文件资源
1. 读取res/raw下的文件资源，通过以下方式获取输入流来进行写操作
```
InputStream is = getResources().openRawResource(R.id.filename); 
```
2. 读取assets下的文件资源，通过以下方式获取输入流来进行写操作
```
AssetManager am = null;  
am = getAssets();  
InputStream is = am.open("filename");  
```
# 读取/res/raw下文件并写入sd卡的示例方法
```
void makeFile() {
    Toast.makeText(MainActivity.this, "开始创建文件", Toast.LENGTH_SHORT).show();
        if (Environment.getExternalStorageState().equals(
            Environment.MEDIA_MOUNTED)) {
        String dirPath = Environment.getExternalStorageDirectory()
                .getPath() + "/WeiPics"; // 要保存的路径
        String fileName = "notes1.png"; // 文件名
 
        try {
            File dir = new File(dirPath);
            if (!dir.exists()) {// 如果目录不存在，创建目录
                dir.mkdirs();
            }
 
            File file = new File(dirPath + "/" + fileName);
            if (!file.exists()) {// 如果文件不存在，创建文件
                InputStream ins = getResources().openRawResource(
                        R.raw.notes1);
                FileOutputStream fos = new FileOutputStream(file);
                byte[] buffer = new byte[8192];
                int count = 0;
                while ((count = ins.read(buffer)) > 0) {
                    fos.write(buffer, 0, count);
                }
                fos.close();
                ins.close();
                Toast.makeText(MainActivity.this, "创建文件成功",
                        Toast.LENGTH_SHORT).show();
            }
        } catch (Exception e) {
 
        }
    }
}
```
另外，在Manifest中记得加入相关权限
```
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```
读取/assets下文件并写入sd卡的示例方法
和上面的方法几乎一致，只是把：
```
InputStream ins = getResources().openRawResource(
                        R.raw.notes1);
```
改为:
```
AssetManager assetManager = getAssets();
InputStream ins = assetManager.open("Notes1.png");
```
# 获取assets文件夹的所有文件
```
AssetManager am = context.getAssets();
String[] path = null;
try {
    path = am.list("");  // ""获取所有,填入目录获取该目录下所有资源
} catch (IOException e) {
    e.printStackTrace();
}
```
