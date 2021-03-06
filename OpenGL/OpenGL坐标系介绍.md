OpenGL坐标系介绍
===============
[TOC]

OpenGL可以分成四种坐标系，分别是世界坐标系，模型坐标系，眼坐标系，设备坐标系。

#数学的观点：向量空间和仿射空间
仿射空间（affine space）是向量空间的扩展，除了标量和向量，它还包含另外一种对象-点。
尽管在仿射空间中对两个点以及一个点和一个标量没有定义运算，但对一个向量和一个点定义了一种运算——向量-点加法，它的结果是一个点。也可以说有一种称为点-点减法的运算，这种运算由两个点得到一个向量。

#计算机科学的观点
把标量、点和向量看做是集合中的元素，并且可以按照某些公理在元素之间进行运算，这是数学家更愿意采用的观点，而计算机科学家更愿意把它们看做是抽象数据类型（ADT），ADT是在数据上定义的一些运算，这些运算的定义不依赖与数据在计算机内部的表示方法和运算的具体实现方式。数据抽象是现代计算机科学的基本概念。例如，把一个元素添加到一个列表中是一种运算，对这种运算的定义不依赖与列表的存储方式。从计算机角度来看，不管一个特定系统在其内部如何表示对象，都应该能够通过下面的代码来定义几何对象：
    
    vector u.v;
    point p,q;
    scalar a,b

#坐标系和标架
目前为止，向量和点都被当做抽象的对象，并没有考虑它们在某个坐标系下的表示。在一个三维向量空间中，可以用任意三个线性无关的向量v1,v2,v3把任意一个向量w表示为

    w = a1v1 + a2v2 + a3v3

把w关于这个基[v1 v2 v3]的表示写成矩阵形式：

        |a1|
     a= |a2|
        |a3|

通常我们认为基向量v1,v2,v3定义了一个坐标系(coordinate system)。然而，除了向量，我们还要处理点和标量，所以还要有一种更一般的方法。

三个基向量可以构成一个基，即可以表示三维空间中的任意向量。不过向量虽然有大小和方向，但是没有位置属性。我们没有解决如何表示点的问题。点和向量是不同的，他们是有固定位置的实体。

因为仿射空间中包含点，所以一旦在这样的空间里选定了一个特定的参考点（原点），就可以以一种明确方式来表示所有的点。习惯上把原点作为坐标轴的起点，这在仿射空间里是有意义的，因为我们既要表示点，又要表示向量。不过，这种表示除了要确定基向量意外，还要确定参考点。原点和基向量决定了一个标架（frame）。关于标架的一个不太严谨的说法是：它把向量坐标系的原点固定在了某个点P0处。在给定的一个标架下，每个向量可以唯一的表示为

    w = a1v1 + a2v2 + a3v3
这和向量空间中一样，但是点可以唯一表示为：

    P= P0 + b1v1 + b2v2 + b3v3

这样，在一个标架下，表示一个向量需要三个标量，而表示一个点需要三个标量和原点的位置。通过放弃更熟悉的坐标系概念转而使用标架的概念，可以避免由于向量具有大小和方向而没有固定位置所带来的困难。此外，我们还能借助矩阵来表示点和向量，并且点和向量这两种几何类型的矩阵是有区别的。

>注意，向量和点一样，其存在与参考系无关，不过我们最终还是要考虑点和向量在某个特定的参考系下的表示。

![a-dangerous-representaion-of-a-vector][1]

#坐标系的变换
我们经常需要在基向量改变时求出向量在新坐标下的表示。例如，在OpenGL中，我们利用对于模型来说是自然的坐标系或者标架来定义几何对象，这个标架称为对象标架(object frame)，或者建模标架（model frame）。然后这些对象标架中的模型又被放到世界标架（world frame），有时，我们需要知道这些对象在照相机中看来是什么样子的，此时就要从世界标架变换到照相机标架（camera frame）或者眼标架（eye frame）。从对象标架到眼标架的转换是由模-视变换矩阵来完成的。

首先假设世界坐标系的基为w，模型坐标系的基为m，w和m的关系为：

    m = w * M

其中M是变换矩阵，把世界坐标系变化成模型坐标系，那我们可以得出在世界坐标系中的坐标a和在模型坐标系中的坐标b之间的关系为：

    a = M * b

当在坐标系中加上固定点成为标架后，以上关系仍然成立。

注意，在使用OpenGL进行坐标变换时，在应用程序中，矩阵进行的是右乘，而在管线中，矩阵进行的是左乘，如：

    glTranslate()
    glRotate()
    分别对应两个变换矩阵T和R，在应用程序中对应对当前矩阵进行右乘变换：
    默认刚开始当前矩阵C是单位阵
        C <—— I
        C <—— C * T * R

当把当前矩阵传入关系进行坐标变换时，则变成了右乘：

    vectex <—— C * vectex 其中 C = T * R

可以看出对于标架的乘法顺序和对顶点的顺序想法，这正符合前面推出的转换公式。
注意，如果加上视图变换，则可以得到新的解释

      C <—— I
      C <—— I * Carema
      C <—— C * Carema * T * R

      vectex <—— C * vectex 其中 C = Carema * T * R

因此这些转换可以有两种看法：
1. 对标架进行变换，使用glTranslate, glRotate等变换其实是对当前坐标架进行变换，然后把顶点坐标放置在当前的新坐标系中。
2. 对坐标进行变换，对坐标进行左乘，进行坐标变换，但由于矩阵乘法的不可交换性，因此变换的顺序和坐标变换的顺序相反，从代码上看就是从下往上进行变换。

>注意，以上的描述都有一个隐含的前提，所有变换都需要一个固定的参考系，这个参考系到底是什么，是世界标架吗，就我目前的理解来看，并不是，而是照相机标架，一切的变换都可以想成是从照相机标架展开的，可以把照相机标架当做不同的标架，其实运动都是相对的，把世界标架当成静止的也没有什么不对，但是在默写API的参数上可能会造成迷惑。

#OpenGL中的标架
绘制流水线会涉及6个标架，但程序员一般不会直接看到全部6个标架。同一顶点在不同的标架下会有不同的表示。按照它们在绘制流水线中出现的先后顺序，依次是：
1.对象标架或建模标架
2.世界标架
3.眼标架或者照相机标架
4.裁剪标架
5.规范化的设备标架
6.窗口标架或者屏幕标架

让我们考虑下当用户在程序中通过函数glVertex3f(x,y,z)指定一个顶点时发生了什么。

#建模标架
这个顶点可以直接在应用程序中指定，也可以通过某个基本对象的实例化来间接指定。在大多引用程序里，我们往往在建模标架(model frame)或者称为对象标架（object frame）下指定对象，因为对象标架是对象自身的标架，在这个标架下可以方便的描述对象的大小、方向和位置。例如，在对象标架下，立方体的面和标架的轴平行，它的中心位于标架的原点，并且长为1个或者2个单位。在相应的函数调用中，所用的坐标值都是相对于对象坐标系的。

#世界标架（world frame）
一个应用程序可能包含成百上千个单独的对象，我们必须把它们放到一个公共的场景里。应用程序通常对每个对象实施一系列的变换，这些变换可以改变它们的大小、方向和位置，从而使它们处于一个适合于该特定应用程序的标架中。
应用程序使用的这个标架称为世界标架（world frame），而在这个标架下的坐标就是世界坐标（world coordinate）。

>注意，如果在调用函数glVertex之前没有利用预先定义好的对象来建模，并且也没有应用任何变换，那么对象坐标系和世界坐标系是相同的。

世界标架是OpenGL用来描述场景的标架，初始值和照相机标架重合，Z+轴指向屏幕外部，X+从左向右，Y+从下向上，是右手笛卡尔坐标系统。

对于应用程序来说，对象标架和世界标架都是很自然的标架，然而生成的图像依赖于照相机或者观察者所能看到的内容。几乎所有的图形系统都使用一个标架，该标架的原点位于照相机透镜的中心（对于透视投影，透镜的中心是投影中心COP，而对于正交投影（默认投影方式），投影的方向和照相机的侧面平行），并且坐标轴平行与照相机侧面。这就是照相机标架（carema frame）。

#照相机标架（camera frame）
照相机标架，也可称为眼标架（eye frame）。和物理定义不同，OpenGL中的照相机标架仍然是右手坐标系，默认位置在世界坐标系下是（0，0，1），标架方向和世界标架相同，但是照相机方向指向-Z方向，这个在后面的视见体中有用。因为每次标架变换都对应一个仿射变换，所以从模型坐标到世界坐标以及世界坐标到眼坐标都可以用4X4矩阵来表示。OpenGL把这些变换合并为模——视变换。对应的变换矩阵是模式变换矩阵。通常，使用模-视变换矩阵而不是两个单独的变换矩阵不会给应用程序员带来任何不变（直接一起用似乎减弱了世界坐标系的存在感）。在可编程流水线中，可以分开处理这两个变化矩阵。

#绘图坐标系（假想坐标系）
假象当前坐标系经过一系列变换从世界坐标系分离，图形在当前坐标系中绘制，把这个坐标系假象成绘图坐标系，其实这个坐标系并不存在，它其实是一系列变换矩阵，能够把顶点从模型标架变换到世界标架。

#裁剪标架、规范化的设备标架、窗口标架或者屏幕标架
把对象变换到眼坐标系中以后，OpenGL就要检查它们是否位于视见体内。如果一个对象位于视见体外，那么就在光山化之前把它从场景里裁剪掉。为了更有效率的裁剪，OpenGL先执行投影变换，这个变换把所有可能是可见的对象变换到一个立方体中，该立方体的中心是裁剪坐标系（clip coordinate）的原点。在投影变换之后，顶点的表示仍然是齐次坐标。顶点的齐次坐标表示再经过透视除法（perspective division），即用w分量去除其他分量，就得到了规范化的设备坐标系（normalized device coordinate）下的三维表示。最后一步变换根据视口提供的信息，把规范化的设备坐标系下的表示变换为窗口坐标系（window coordinate）下的三维表示。窗口坐标系是用显示器上的像素来度量的，但仍然保留了深度信息。如果去掉深度坐标，就得到了二维的屏幕坐标（screen coordinate）。

#总结
从应用程序员的角度看，OpenGL从两个标架开始：眼标架和对象标架。模-视变换矩阵是对象标架在眼标架下的表示。因此，模-视矩阵是系统状态的一部分，所以总是有一个当前的照相机标架和一个当前的对象标架。OpenGL提供了矩阵栈，这样就能把模-视变换矩阵保存起来，也相当于把标架保存起来。

[1]: images/a-dangerous-representaion-of-a-vector.png