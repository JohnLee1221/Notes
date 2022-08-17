# evn | grep " "

查看指定的环境变量



# find /usr/local/ -type f -name "*.txt"

f 普通文件
l 符号链接
d 目录
c 字符设备
b 块设备
s 套接字



# ros topic list --no-deamon

Do not spawn nor use an already running daemon



# set(**** false)



# pm-hibernate



# rosdep install --from-paths src --ignore-src --rosdistro=$ROS_DISTRO -y --os=ubuntu:focal



```
colcon build --packages-select smac_planner
libceres-dev
libceres1
libsuitesparse-dev
libsuitesparseconfig5
```





Swin-Transformer-Semantic-Segmentation

controller_testing

motion_model_testing_simulator

sensor_msgs_py





### **typedef**

为`现有类型`创建一个新的名字，基本用法举例：

```text
typedef unsigned char UCHAR;//右边代替左边

void tech(void) { printf("tech dreamer"); }
//命名一个类型，那么这个时候func不可以直接调用，而是一个类型了
typedef void (*func)();
void main()
{
    //定义一个可调用的指针变量（函数）：myfunc
    func myfunc;
    myfunc = &tech; //&可以不加
    myfunc();         //第一种调用方式，带参数也可以
    (*myfunc)();      //第二种调用方式，带参数也可以
}

typedef void(*Func)(void)为什么能这么用？
可能我们有这样的疑问，typedef一般的用法不是取个新名字吗？如果仿照变量类型声明，应该是这样typedef void(*)() variable;才比较眼熟易懂。

说一说我的理解，我们知道C语言的函数在被调用的时候是执行了一条转移指令，这条指令包含要转移到的地方的数据，同理，变量代表了一个地址，当你引用一个变量的时候，计算机会把这个地址的数据拿出来参与运算。

那么函数名也是一样，函数名代表的是一段代码的入口地址，引用了这个函数，那么计算机会使用转移指令转移到函数名代表的地址。

所以typedef void(*Func)(void)相当于定义了一种类型，这个类型具有下面的特征：他是一个函数，没有返回值，没有参数。

因为处理器在进行上下文切换或者转移的时候要进行现场保护，不同的函数对现场保护的内容可能不一样，传入的参数使用的栈也不一样，相同类型的函数保证了现场保护的内容是一样的，参数的形式也是一样的。所以这样使用可以像用指针引用数据一样使用函数。

typedef void(*Func)(void)的应用
用法的好处，定义一个函数指针类型。
```



/opt/ros/foxy/share/std_msgs

/opt/ros/foxy/share/builtin_interfaces



# cmake .. -DCMAKE_PREFIX_PATH=/usr/local/



# 共享盘

```
smb//:192.168.1.44

name: admin

password: linearx
```







