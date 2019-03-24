# sccpu_design


### bne指令测试


添加BNE指令， 
![](https://upload-images.jianshu.io/upload_images/10651191-7d53150da757266a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这是最开始原始指令结果
![](https://upload-images.jianshu.io/upload_images/10651191-b6e5be1d9546f710.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后添加测试文件
![](https://upload-images.jianshu.io/upload_images/10651191-2509a5abd58c75a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到指令这次成功的实现了跳转！！

文件名。。。是.dat的后缀
然后
![](https://upload-images.jianshu.io/upload_images/10651191-3f46c54c35c1ecbd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
显然是没有成功的
为什么呢？

因为BNE需要一个比较的零信号， 然后我就忘记添加了

![](https://upload-images.jianshu.io/upload_images/10651191-35a952c1bb06795c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看一下仿真的波形和寄存器的值吧

在这之前需要添加测试的文件
![通过Mars 去dump一个十六进制的指令文件](https://upload-images.jianshu.io/upload_images/10651191-265f7b5b1912c758.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我不是特别清楚修改了源文件之后是不是需要重新编译， 不过最好还是退出仿真然后重新编译吧。

然后我们添加测试的信号
由于是跳转指令， 就看一下控制信号和PC NPC这几个信号吧
![修改成十六进制方便查看](https://upload-images.jianshu.io/upload_images/10651191-c899e950ed331090.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后我们看到在PC = 18的时候bne的控制信号为1
这时候发生跳转， NPC的值被置为44

![](https://upload-images.jianshu.io/upload_images/10651191-3a5712187455de84.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在transscript的输出窗口也可以看到PC的变化

![](https://upload-images.jianshu.io/upload_images/10651191-56f825f9c289e6a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## jr指令实现

jr指令是R类型的指令， 实现读取寄存器的值然后进行跳转。

调用了一条jal指令， 返回地址被压入了 $ra寄存器中


![](https://upload-images.jianshu.io/upload_images/10651191-f4f68ec57f827075.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

写到这里我终于知道自己出现什么问题了。
因为指令在译码的时候出错了。


![](https://upload-images.jianshu.io/upload_images/10651191-bd1aa0bf398ae9e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


看到这里指令的信号来到了上升沿。
所以成功了。

![](https://upload-images.jianshu.io/upload_images/10651191-4c40bc59ae4ab3cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 测试JALR指令


![指令译码](https://upload-images.jianshu.io/upload_images/10651191-47ac5e8fca5b7ee6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![需要写寄存器](https://upload-images.jianshu.io/upload_images/10651191-bcafbd00f19a9355.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![需要选择写哪些寄存器](https://upload-images.jianshu.io/upload_images/10651191-d13dfcc6cc9f2b9c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![需要选择写的数据](https://upload-images.jianshu.io/upload_images/10651191-d4bfe83e595168b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![需要对NPC进行选择](https://upload-images.jianshu.io/upload_images/10651191-7a9421b880e8bf7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


放一下测试代码

```
main:   addi $2, $0, 5          # initialize $2 = 5     00      0       20020005
        addi $3, $0, 12         # initialize $3 = 12    01      4       2003000c
        addi $7, $3, -9         # initialize $7 = 3     02      8       2067fff7
        jal label1
        or   $4, $7, $2         # $4 = (3 or 5) = 7     03      c       00e22025
        and  $5, $3, $4         # $5 = (12 and 7) = 4   04      10      00642824
        add  $5, $5, $4         # $5 = 4 + 7 = 11       05      14      00a42820
        bne  $5, $7, label2     # shouldn be taken      06      18      10a7000a
        slt  $4, $3, $4         # $4 = (12 < 7) = 0     07      1c      0064202a
        beq  $4, $0, label1     # should be taken       08      20      10800001
        addi $5, $0, 0          # shouldn't happen      09      24      20050000
label1: slt  $4, $7, $2         # $4 = (3 < 5) = 1      0A      28      00e2202a
        add  $7, $4, $5         # $7 = 1 + 11 = 12      0B      2c      00853820
        jalr $ra
        sub  $7, $7, $2         # $7 = 12 - 5 = 7       0C      30      00e23822
        sw   $7, 68($3)         # [80] = 7              0D      34      ac670044
        lw   $2, 80($0)         # $2 = [80] = 7         0E      38      8c020050
        j    label2             # should be taken       0F      3c      08000011
        addi $2, $0, 1          # shouldn't happen      10      40      20020001
label2: sw   $2, 84($0)         # write adr 84 = 7      11      44      ac020054
loop:   j    loop               # dead loop             12      48      08000012
```
看波形， 成功了！
![](https://upload-images.jianshu.io/upload_images/10651191-444cad5af66095c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


可以看到成功地跳转到了寄存器对应的地址，并且把下一条指令的值写入了$ra！

![](https://upload-images.jianshu.io/upload_images/10651191-c57c0707131494c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 测试NOR指令

这条指令也就是做一个或非的运算， R型指令， 实现方式和其他的R型指令是很类似的

我选择给ALU扩充一个或非的运算

![](https://upload-images.jianshu.io/upload_images/10651191-a851b5cdebbd359b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![NOR运算扩充](https://upload-images.jianshu.io/upload_images/10651191-d2fcfbb25e97570a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![指令的格式](https://upload-images.jianshu.io/upload_images/10651191-80dd756a62a73c90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](https://upload-images.jianshu.io/upload_images/10651191-49c4eec20cfbe354.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

R类型的指令都需要写寄存器， 所以这一步不需要了
然后是选择ALU的运算
到这里应该是结束了

![](https://upload-images.jianshu.io/upload_images/10651191-e6fef312d93ffdb3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从波形图上可以看到nor的信号了

![](https://upload-images.jianshu.io/upload_images/10651191-bfb8fd51c7d6bd16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

同时也可以看到修改了寄存器的值

![](https://upload-images.jianshu.io/upload_images/10651191-b07cbf228e1c493c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 测试sll  & sra & srl指令

这些指令有一个共性， 都需要移位操作。
但是ALU并没有提供移位的功能。

所以唯一的选择就是扩展ALU(我说呢， 怪不得老师给的代码里面的ALUop和书上的对不上)


![扩展ALU](https://upload-images.jianshu.io/upload_images/10651191-3e55c020d2eb45c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

为了对之前的指令不造成影响， 所以我决定原先的ALU操作的第四位都是0

这样的话，它默认就是0， 不需要去设置了

![扩展ALU的功能](https://upload-images.jianshu.io/upload_images/10651191-d91b47fada9ceefe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

功能扩展完了， 我还需要添加一个多路选择器，选择哪里的呢? 当然是对ALU的A输入口进行选择， 因为这三种移位类型的指令都是R型的指令。
也就是说它们都是对rt进行操作， 然后写会到rd中。

所以ALU的A输入口输入的就是shamt
ALU的B输入口输入的就是 rt的值(这个已经有多路选择器实现了，如图

因为lw， sw都是需要对来自rt和立即数的值进行选择， 也就是说它们的操作就是选择立即数与rs进行运算。


![ALUSrc](https://upload-images.jianshu.io/upload_images/10651191-0a46c00346109225.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


既然要加多路选择器， 首先要有一个多路选择信号， 然后就是两个待选择的信号， 一个是shamt， 一个是rs的值。

所以控制单元的输出要加一个选择信号

把这三条指令都添加上了

![](https://upload-images.jianshu.io/upload_images/10651191-f8878f1b96f59b2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

同时输出一个信号对多路选择器进行选择

![](https://upload-images.jianshu.io/upload_images/10651191-3487a3dfd4548aa2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当然这几条指令都需要写寄存器， 所以都默认了
当然不要忘了实例化的时候需要增加一个绑定

![](https://upload-images.jianshu.io/upload_images/10651191-6ab9e811dc9040b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

别忘了增加信号
![](https://upload-images.jianshu.io/upload_images/10651191-47b25045cdff80f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

译码的时候需要补充shamt信号

![写成32位的方便运算](https://upload-images.jianshu.io/upload_images/10651191-2e13267930e071bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![译码](https://upload-images.jianshu.io/upload_images/10651191-c9a1003762c1aed7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![给ALU的A输入口新增一个线](https://upload-images.jianshu.io/upload_images/10651191-8300d3fe7e363048.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![多路选择器的输出就是那根线](https://upload-images.jianshu.io/upload_images/10651191-ff17ac6adb6e6e17.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![同时修改ALU的口](https://upload-images.jianshu.io/upload_images/10651191-5852a444222ef2ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

ok到这里应该结束了

出错了~~
从这张图中可以看到ALU的op[3]一直是z。。。。

![](https://upload-images.jianshu.io/upload_images/10651191-19d20919f9ec85f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

原因是我没有修改控制信号

![](https://upload-images.jianshu.io/upload_images/10651191-1cb88524c42bb66c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![ok成功的实现了sll指令](https://upload-images.jianshu.io/upload_images/10651191-1be848110ee2e310.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![这个时候可以看到波形了](https://upload-images.jianshu.io/upload_images/10651191-dbb7a5d34fb4c646.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![可以看到这个是移位的shamt字段](https://upload-images.jianshu.io/upload_images/10651191-9ab616a0b9e39d42.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来测试srl指令
![ok](https://upload-images.jianshu.io/upload_images/10651191-e4b0ee497ebfd219.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![这时候可以看到ALU的输入里面一个是2，一个是5](https://upload-images.jianshu.io/upload_images/10651191-0a06470814c0a392.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

测试SRA指令

![](https://upload-images.jianshu.io/upload_images/10651191-3a560b3ee943ac7e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

ok大功告成

![](https://upload-images.jianshu.io/upload_images/10651191-517dd8cf425b9987.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 测试sllv和srlv
这两条指令也都是R类型的， 实现的功能和sll和srl差不太多， 但是它们要实现的可移位的位数必须是变化的。
![继续扩展ALU的功能](https://upload-images.jianshu.io/upload_images/10651191-6dde2567fa31f93f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![扩展ALU的功能](https://upload-images.jianshu.io/upload_images/10651191-6e61a9962031c5d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![添加控制信号](https://upload-images.jianshu.io/upload_images/10651191-a31c80fcbf51cbd7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

都需要写寄存器这是不用说的

修改ALUop
![ALUOp](https://upload-images.jianshu.io/upload_images/10651191-c27eb8e354a66c43.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


但是我这里犯了个错
我以为是把rt做偏移量， 没想到是把rs做偏移量。。。。
那行， 
![怎么感觉这个代码写的这么滑稽呢？](https://upload-images.jianshu.io/upload_images/10651191-eb378941c8827232.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其他的仿真过程都差不太多， 我就不放图了。

## ok最后两条指令了

先看slti指令

与立即数比较，小于则置位
其实和slt差不太多
不过比较的时候多路选择器要发挥它的作用了

然而神奇的是这是一条I类型的指令

很明显它需要写寄存器


![](https://upload-images.jianshu.io/upload_images/10651191-a14b004773203e08.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

同时还需要有对多路选择器进行选择
![ALU的B输入口需要立即数](https://upload-images.jianshu.io/upload_images/10651191-8c5d5cb72a92d18e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![ALU的选择和SLT的应该一样](https://upload-images.jianshu.io/upload_images/10651191-9b8f096dfeb2a85d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

ok开始测试

![ok寄存器没问题](https://upload-images.jianshu.io/upload_images/10651191-ce470ed8aa6abace.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![看看波形ok没问题](https://upload-images.jianshu.io/upload_images/10651191-3994c369e21569bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 最后一条lui

这个是取高16位放进一个寄存器里面去，
也是I类型的指令
不过我们需要将一个16位的立即数符号扩展之后，再左移16位， 然后把运算结果写会到寄存器中去。

所以这个时候我需要对ALU再增加一个选择信号

![](https://upload-images.jianshu.io/upload_images/10651191-f30316c7297fc117.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![对ALU进行扩展](https://upload-images.jianshu.io/upload_images/10651191-db88fecdec6dac90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![寄存器写信号](https://upload-images.jianshu.io/upload_images/10651191-e28cd5247fc884a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![ALU的B入口是一个立即数](https://upload-images.jianshu.io/upload_images/10651191-7146740190f53586.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![对ALUOp进行修改](https://upload-images.jianshu.io/upload_images/10651191-161c55965dab2d18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![i_lui指令还需要写会rt](https://upload-images.jianshu.io/upload_images/10651191-1739c25d244b623a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


zz的我把ALU运算搞错了

![应该是对B进行运算的。。。](https://upload-images.jianshu.io/upload_images/10651191-f6e95c587c67ab04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/10651191-2184017c4c206672.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
ok这样就成功了

## andi指令

andi指令也很类似

也就是把一个十六位的数字去& 一下
![控制信号](https://upload-images.jianshu.io/upload_images/10651191-ac3c1eeec74f4fcc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

需要符号扩展
需要写寄存器
还需要对ALU进行选择
![测试](https://upload-images.jianshu.io/upload_images/10651191-9ec6e73a68c18943.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![ok](https://upload-images.jianshu.io/upload_images/10651191-ac9deee4ee580228.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


至此， 蓝色部分指令全部添加完成

