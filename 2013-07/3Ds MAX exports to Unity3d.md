Tags: 3ds max, unity3d

最近在折腾unity3d，搜集了一些教程后，就开动了。
虽然我是一个码农，但大一大二这两年都在自学3ds max，所以有点这方面基础，感觉unity3d入门很快。

最重要的是对于一个后台工程师而言，用拖拉的方式来构建应用程序确实让人觉得很舒服。再也不用对着黑乎乎的窗口敲代码，还得google各种vim的技巧用来让人手抽筋。然后还得把自己的大脑变成计算机，让你的代码在你的脑子里跑。各种蛋疼。

总之，从建模，贴图，到进入unity3d，一天搞出来了个火箭筒的发射例子：

![RPG][1]

## 坐标轴不正确会遇到很多麻烦

一直没注意啊，反正把物体从3ds max通过FBX格式导入到 unity3d中，一看物体朝向不对，也没去研究到底该如何设置，就手动旋转一下，然后建立一个Empty Object，作为其父物体，急着一个可以运行的demo。到后面才发现这会给写脚本带来很大的麻烦。

刚才仔细研究了下坐标轴的问题。google了半天，自己也对着两个3d软件的axis看了半天，3ds max是右手坐标系，其他大部分三维软件（包括unity3d）都是左手坐标系，真心蛋疼啊。自己摸索着要让着两种坐标系重叠，光在导出FBX的时候翻转Y/Z轴是不行的。自己想着应该手动的旋转物体的axis。

但自己各种旋转后，发现此路不通。于是继续google，发现了这张示意图：

![shiyi][2]

他讲的和我的想法一样，都得在max中先旋转物体的轴向。于是我又去检查自己的max。
突然发现一个蛋疼的问题: **max 设定的 -y 轴是正前方**， 而我傻傻以为 y 轴应该和 x, y 一样吧，觉得 +y 才是前方…… 然后把物体先绕着z轴，转180°，让物体的前端指向 -y 轴。然后再操作就OK了，导入到unity3d中的模型默认的 rotation就是 0, 0, 0，而且朝向是正确的。

总结一下操作步骤：

1.  max中的模型确保物体前方朝向 -y 轴
2.  将物体的轴 x 翻转 90°
3.  导出FBX的时候，选择 Y UP， Y轴朝上。

OK，就这些要注意的。

## 导入unity3d后，默认的scale为什么不是1, 1, 1

在这个之前先说一下max的单位设置，

![max unit][3]

建模的时候就按照物体的大小来建模，导出FBX的时候，默认设置就是正确的。

![FBX][4]

也就是 单位用默认，然后设置Y轴朝上。

只要注意上面的设置，导入到unity3d的物体，默认参数就很漂亮，

![unity3d][5]

#### 为什么我导入后，默认的scale不是 1, 1, 1

这是因为你的模型在max中已经缩放过了。对于这种模型，最好在max中做一下 **reset xform**， 中文叫 **重置变换** 。

如何操作可以参考 [3ds max的官方文档][6]

**注意：**  重置变换后虽然缩放都会重置为1, 1, 1， 但物体的轴向也重置了。
我们上面说过要旋转物体的x轴90°，所以在重置完毕后，要重新旋转物体的轴向。

[1]: http://i1297.photobucket.com/albums/ag23/yueyoum/firstunitywork_zps8b099495.jpg
[2]: http://i1297.photobucket.com/albums/ag23/yueyoum/4phcI_zpse5a68434.jpg
[3]: http://i1297.photobucket.com/albums/ag23/yueyoum/unit_zps0cebe7d9.png
[4]: http://i1297.photobucket.com/albums/ag23/yueyoum/fbx_zpsb88fd467.png
[5]: http://i1297.photobucket.com/albums/ag23/yueyoum/uu_zps6f814779.png
[6]: http://docs.autodesk.com/MAXDES/13/ENU/Autodesk%203ds%20Max%20Design%202011%20Help/index.html?url=./files/WSf742dab041063133-61da5a1f112a1cebf4a-7ff2.htm,topicNumber=d0e45546