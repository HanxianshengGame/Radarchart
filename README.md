@[TOC]( UGUI实现雷达图?)

# （一） 雷达图介绍
<br>

**<font color=red size=4> 介绍:</font>**

&ensp;&ensp;&ensp;&ensp;雷达图是以从同一点开始的轴上表示的三个或更多个定量变量的二维图表的形式显示多变量数据的图形方法，它利用代表各项数据占比的顶点位置连接成面状，**直观的反应当前事物的一个能力倾向**。

例如下图：我们能直观的看到该玩家对于（输出，发育）两项的能力比较好?
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190917102506368.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nemlfZGFpbWE=,size_16,color_FFFFFF,t_70)
# （二）UGUI实现雷达显示图的原理
<br>

**<font color=red size=4> 雷达图基本原理介绍:</font>**

（这里我们以正五边形的雷达背景图为例，其他正多边形原理相同）
**1.我们首先需要具备一张美术的雷达背景图，之后在背景图下创建Image；
2.之后重写image的OnPopulateMesh(VertexHelper vh)函数，清空所有顶点信息（VertexHelper.Clear() 清空后该image就不会进行显示）；
3.之后我们需要构建五边形的5个顶点(初始位置在正五边形的五个顶点位置)，之后创建外部系数控制5项数据的大小，得到5个顶点的最终显示位置，之后3个顶点构成面即可；
4.重新给VertexHelper对象添加顶点信息，三角面信息，我们原始的Image就会显示为五边形形状  来表现数据倾向。**

<br>

**大致来说：**
<font size=4>&ensp;&ensp;我们游戏中的雷达图都是接收了数据源，计算得到该数据源每项占各项最大限度的比值，之后将原本的各项顶点的位置Position与比值相乘，得到新的顶点位置Position，再利用OnPopulateMesh(VertexHelper vh)重新绘制显示面。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190917110617451.gif)


<br>

**<font color=red size=4>Image重新绘制图形API：**

&ensp;&ensp;&ensp;&ensp;了解：Image类继承自MaskableGraphic，实现了ISerializationCallbackReceiver, ILayoutElement, ICanvasRaycastFilter这三个接口。**最关键的是MaskableGraphic类，MaskableGraphic负责绘制逻辑，MaskableGraphic继承自Graphic，Graphic里有个OnPopulateMesh函数**，这正是我们需要的函数。
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMyMDE1LmNuYmxvZ3MuY29tL2Jsb2cvNjA4OTk2LzIwMTcwMi82MDg5OTYtMjAxNzAyMjExMzMwMzY3MjYtMTEwNzcyOTExOC5wbmc?x-oss-process=image/format,png)


---



# （三）利用UGUI制作雷达图
以正五边形为例

**<font color=red size=4>步骤1：**
**&ensp;首先我们应该知道雷达背景图(所代表的圆)的半径radius，由于背景图的image尺寸不同，因此我们可以通过自己手动测量得到。**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190917111629452.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nemlfZGFpbWE=,size_16,color_FFFFFF,t_70)


<font color=red>手动测量演示：</font>
 在背景图下创建一个物体，位置居于雷达图的中心，记录其posY1，向上拖到最顶端的顶点位置处，记录其posY2，利用
**posY2-posY1即可得到radius**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190917111644860.gif)
<br>
<br>

**<font size=4 color=red>步骤2.</font>
&ensp;&ensp;利用半径以及弧度求五边形雷达图最顶端的5个顶点坐标。**
<br>
原理：<font color=red>5个顶点都是基于雷达图中心进行的一个坐标偏移，因此我们可以通过**sin()*radius，cos()*radius**求出这个偏移量，例如下图一。

图一：(黑线即五边形以及对应的圆， 红粗线为五边形对应的圆的半径)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190917112921905.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nemlfZGFpbWE=,size_16,color_FFFFFF,t_70)
<br>
<br>
<font color=red>**注意：**</font>
&ensp;由于在**c#和unity中 Mathf.Cos()和Mathf.Sin() 传参时传递的为弧度(radian)**,我们需要把角度转换为弧度计算。


首先整圆的弧度为  **2*pi**  ，因此我们可以计算出五边形每条边对应圆的弧度，即 **(2*pi)/5** ,之后我们以1号顶点为起始点，逐渐加上
 (2*pi)/5得到2，3，4，5号顶点的弧度。
 
即：
1号起始顶点对应弧度(整圆的1/4) =2*pi / 4 ；
       2号顶点对应弧度 =（2*pi / 4） +(2*pi) / 5 * 1；
       3号顶点对应弧度 =（2*pi / 4） +(2*pi) / 5 * 2；
       4号顶点对应弧度 =（2*pi / 4） +(2*pi) / 5 * 3；      
       5号顶点对应弧度 =（2*pi / 4） +(2*pi) / 5 * 4；     
<br>
之后有了五个弧度，即可直接求得5个顶点坐标:

```
      private void SetPointPos()          //设置点坐标
        {
            float radian = 2 * Mathf.PI / _pointCount;  //计算出当前多边形的每一段对应填充圆的弧长
            float radius = 130; //暂定当前正多边形的所填充圆的半径为100.
            float curRadian = 2 * Mathf.PI / 4.0f;     //计算出初始弧度位置 为正五边形的正上方  （起点选择与坐标轴延伸线相接的地方）
            for (int i = 0; i < _pointCount; i++)
            {
                float x = Mathf.Cos(curRadian) * radius;
                float y = Mathf.Sin(curRadian) * radius;
                curRadian += radian;
                _points[i].anchoredPosition = new Vector2(x, y);
            }
        }
```

<br>
<br>
<br>
<br>
<br>

**<font size=4 color=red> 步骤3.</font>
&ensp;&ensp;&ensp;设置雷达图的各项数据比例系数，得到每项数据对应雷达图的沿半径比例的准确顶点位置**
<br>
&ensp;&ensp;&ensp;各项数据比例系数是从外部传递的，我们不需要管，我们的目的是**如何让雷达图按照这些数据给显示出来。**
**<font color=red>原理：**
&ensp;&ensp;&ensp;红圆圈是我们按照比例系数重新计算的顶点坐标，黑圆圈是我们的五个原始顶点坐标，我们不难看出，如果比例系数为1时，这项数据就是已经到顶了(顶点位置依旧不变)，当小于1时，我们只需要将我们的顶点坐标与比例系数(radio)相乘就可以得到新的顶点坐标。

例如：1号顶点原本是黑圆圈位置，乘上比例系数radio=0.7的位置，是在黑圆圈与中心的连线的十分之七处。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190917125812389.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nemlfZGFpbWE=,size_16,color_FFFFFF,t_70)![在这里插入图片描述](https://img-blog.csdnimg.cn/20190917130135287.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nemlfZGFpbWE=,size_16,color_FFFFFF,t_70)
<br>



**<font size=4 color=red> 步骤4</font>
&ensp;&ensp;创建5个雷达图顶点UIVertex和一个雷达图中心点UIVertex**
<br>

当我们知道了重新进行计算的5个雷达图顶点位置之后，我们直接创建5个顶点即可，创建顶点格式如下

```
            UIVertex uiVertex=new  UIVertex();
            uiVertex.color = base.color;   //这里指基类Image的颜色
            uiVertex.position = pointPos; //雷达图顶点坐标
            uiVertex.uv0=Vector2.one;      //uv坐标   由于雷达图都是纯色显示，不依赖Texture，我们直接默认Vector2.one即可
```

<font color=red>注意：中心点UIVertex的坐标为 雷达背景图中心点位置 。

<br>
<br>
<br>

**<font size=4 color=red> 步骤5.  </font>
&ensp;利用OnPopulateMesh(VertexHelper vertexHelper) 重新添加顶点信息以及三角面信息。**
<br>

<font color=red>原理：</font>
&ensp;&ensp;image的画面展示都是由**OnPopulateMesh方法完成的，其中VertexHelper 是承载顶点信息与面信息的类**，我们将它的数据清理掉后，添加我们的面和顶点信息，我们就能渲染出一个雷达图了。

```
         protected override void OnPopulateMesh(VertexHelper vertexHelper)
        {
            vertexHelper.Clear();
            AddVerts(vertexHelper);     //添加全部顶点信息
            AddTriangles(vertexHelper);  //添加全部面信息
        }

```

```
添加顶点api：VertexHelper.AddVert( Vertex vert)
添加面api：    VertexHelper.AddTriangle(int index0，int index1,int index2)
```

<br>

<br>
<font size=4 color=red>VertexHelper接受三角形信息原理讲解：</font>

**&ensp;&ensp;&ensp;&ensp;VertexHelper是通过AddTriangle接口接受三角形信息：**public void AddTriangle(int idx0, int idx1, int idx2)**
接口的传入参数并不是UIVertex类型，而是int类型的索引值。哪来的索引？还记得之前往VertexHelper传入了一堆顶点吗？按照传入顺序，第一个顶点，索引记为0，依次类推。每次传入三个顶点的索引，就记录下了一个三角形。**


<font color=red>**需要注意**：</font>
<font size=4>&ensp;&ensp;&ensp;&ensp;GPU 默认是做backface culling(背面剔除)的，GPU只渲染正对屏幕的三角面片，当GPU认为某个三角面片是背对屏幕时，直接丢弃该三角面片，不做渲染。那么GPU怎么判断我们传入的某个三角形是正对屏幕，还是背对屏幕？答案是通过三个顶点的时针顺序，当三个顶点是呈顺时针时，判定为正对屏幕；呈逆时针时，判定为背对屏幕。</font>
<br>
<font size=5>下图中：左边的图中指定顶点的顺序是顺时针的，右边是逆时针的。</font>
![在这里插入图片描述](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMyMDE1LmNuYmxvZ3MuY29tL2Jsb2cvNjA4OTk2LzIwMTcwMi82MDg5OTYtMjAxNzAyMjExODI1MDE2MDEtNDYxNDAzMzI4LnBuZw?x-oss-process=image/format,png)
因此：我们添加顶点时以 （1，0，2）（2，0，3）（3，0，4）.。。。这样**顺时针添加**即可完成添加面的操作
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190917132314941.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nemlfZGFpbWE=,size_16,color_FFFFFF,t_70)
**<font size=4 color=red> 步骤6.&ensp;项目展示：**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190917110617451.gif)

----
<br>
<br>

# （四）项目工程链接地址(GitHub)
<br>

求github小星星哈，谢谢啦?
[Radarchart项目工程链接](https://github.com/HanxianshengGame/Radarchart)
(https://github.com/HanxianshengGame/Radarchart)

**参考 scene Example 4与Script Example4 进行学习：**


![在这里插入图片描述](https://img-blog.csdnimg.cn/20190917133159476.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nemlfZGFpbWE=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190917133217522.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nemlfZGFpbWE=,size_16,color_FFFFFF,t_70)
<br>
<br>

 --------------------------------------------------------我是有底线的-------------------------------------------------------------
  
 &ensp;&ensp;&ensp;&ensp;<font color=red>感谢能够观看博客的各位Unity开发爱好者们，有问题发表评论呐，*★,°*:.☆(￣▽￣)/$:*.°★* 。 
