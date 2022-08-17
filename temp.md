1.ndt_node移植，目前思路有点乱

编译通过，但是运行ndt_map_publisher_exe时，错误：

![1](/home/john/Pictures/20220810-014524)

ldd ./build/ndt_map_publisher_exe

![2](/home/john/Pictures/20220810-014937)



2.重新看了一下ndt_node的代码，其中#include <rclcpp_components/register_node_macro.hpp>

用到了ros2的components，又网上简单了解一下components的概念



3.看了一下autoware中关于client和service的packages，发现也是比较复杂，有些package中的create_client是通过rclcpp::action去实现的，有些package的src下面的文件比较多，又被include当中，有些复杂，目前梳理的比较乱



4.我从ros2官方中，下了examples的package，对应的都是比较小的demo，打算先把这里面的topic	timers	services，移植到我们的caros，

![3](/home/john/Pictures/20220810-024516)
