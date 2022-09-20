# porting意见和建议

1.cmakelists.txt中要手动添加路径；

2.工程目录的目录结构需要梳理更加规范一些；

3.依赖的库文件，能够通过自己的编译框架，源码编译生成指定的库；

4.代码中有些相同功能函数用的比较乱，比如sleep()和usleep()，可以统一一个；

5.caros运行时候与ros2的shared memory冲突，Segmentation fault (core dumped)；	(已经解决，是环境变量设置的问题)

6.通过源码，在我司的编译框架下自动编译生成lib文件和header文件，无需手动放置；

------















# porting总结

1.ros环境关闭掉；

2.idl要通过我司框架（内部的函数模板），生成hpp和so文件之类；

3.一些函数实现，如果依赖过多，可以通过简单的自己手动更改，内部实现；

------











# porting问题:

### 1.下面这两种运行node的方式有何不同，分别适用于什么情况？

POT/caros/sample/demo_stage1/node1.cpp

```c++
  //CarOS 运行环境， 包括Node运行， 全局event列表注册，读入和监听
  caros::RuntimeEnvironment* rev = new caros::RuntimeEnvironment();  
  
  //CarOS 运行环境初始化
  rev->Init(3, {0,1});
  
  //把Node指针放入CarOS 运行环境运行
  //单Node可以以下这种方式
  rev->Run(std::make_shared<TestNodeApp>("node_1"));
  
  //如果多node运行在一个runtime env， 用如下方式， 可以添加多个Node
  // rev->AddNode(std::make_shared<TestNodeApp>("node_1"));
  // rev->Run();
  
  //等待关闭
  rev->Shutdown();
```

POT/caros/sample/demo_stage/node4_autoware_pub.cpp

```c++
  //把Node指针放入CarOS 运行环境运行
  TestNodeApp * testNode = new TestNodeApp("node4_autoware_pub");

  std::thread node4_autoware_pub(&TestNodeApp::topicSender, testNode);

  node4_autoware_pub.join();
```



### 2.node1中的void node1_Handler1(void)，干了哪些事情？和简单的callback有什么区别？

```c++
  void handler_pub()
  {
    tutorial_interfaces::msg::Num sendData1; 
    tutorial_interfaces::msg::Num sendData2;
    this->getChnlDDSWriterPtr()->send("/linearx/hello", sendData1); //"tutorial_interfaces::msg::Num"
    this->getChnlDDSWriterPtr()->send("/linearx/world", sendData2); //"tutorial_interfaces::msg::Num"
  }

  void node1_Handler1(void)
  {
      std::cout << "CallBack Test node1_Handler1" <<std::endl;

      static std::shared_ptr<caros::Node> _node=std::make_shared<caros::Node>("node_1");
      static std::shared_ptr<caros::channel::ChannelWriter<caros::WriterBase>> _ddsWriter=std::make_shared<caros::channel::ChannelWriter<caros::WriterBase>>(_node);       // Data数据通道
      static std::shared_ptr<caros::channel::ChannelWriter<::ipc::channel>> _ipcWriter=std::make_shared<caros::channel::ChannelWriter<::ipc::channel>>();
      static tutorial_interfaces::msg::Num sendmsg;
      static bool init=false;
      if(init==false)
      {
          init=true;
          _ddsWriter->init({{"/topic_1",10,"tutorial_interfaces::msg::Num"}});
          _ipcWriter->init({IPCMSGTOSCHEDULE(_node->getNodeName())});
      }
      sendmsg.num+=1;
      std::cout<< "topic_1 send " <<sendmsg.num <<std::endl;
      _ddsWriter->send("/topic_1",sendmsg);

      using namespace caros::channel;
      std::string data_event_name = "Topic_1";
      auto package = std::make_shared<Package>(
          CAROS_CMD_TYPE::CAROS_CMD_DATA | CAROS_DATA_CMD::CAROS_CMD_DATA_UPDATE,
          std::make_shared<DataEventMsg>(data_event_name.c_str(), data_event_name.length(), EventStatus::E_S_ACTIVE));
      // 序列化
      auto [buf, len] = package->PackSendData();
      _ipcWriter->send(IPCMSGTOSCHEDULE(_node->getNodeName()), buf, len);

      static tutorial_interfaces::msg::Num control_cmd;
      linearx_publisher_lateral_->write(control_cmd);
      std::cout << "node1_Handler1 send topic :'/linearx/control' " <<control_cmd.num<<std::endl;
      control_cmd.num+=10;
  }
```



### 3.是不是存在多种方式生成subscription？

```c++
std::shared_ptr<caros::Writer<tf2_msgs::msg::TFMessage>>  m_pub_earth_map;
m_pub_earth_map = createPublisher<tf2_msgs::msg::TFMessage>("topic_1",caros::QoS(10));


std::shared_ptr<caros::Reader<tutorial_interfaces::msg::Num>> reader;
reader = this->createSubscription<tutorial_interfaces::msg::Num>(topic, qos, 
            [this](tutorial_interfaces::msg::Num& msg)->void{
            async_cbk_func(msg);
            }
            );
```



POT/caros/sample/demo_stage/node4_autoware.cpp

```c++
std::function<void(tutorial_interfaces::msg::Num)> callback1=std::bind(&TestNodeApp::node4_handle1,this,std::placeholders::_1);
this->createSubscription<tutorial_interfaces::msg::Num>(std::string("topic_1"),caros::QoS(10),callback1);
```



### 4.ros中的timer如下，caros的定时器是不是直接在while循环中添加一个sleep来实现？

```
rclcpp::WallTimer<CallbackT>::SharedPtr rclcpp::create_wall_timer ( std::chrono::duration< DurationRepT, DurationT > period,
																	CallbackT 										callback,
																	rclcpp::CallbackGroup::SharedPtr 				group,
																	node_interfaces::NodeBaseInterface * 			node_base,
																	node_interfaces::NodeTimersInterface * 			node_timers 
																	)
```

```
std::this_thread::sleep_for(std::chrono::seconds(1))；
```



### 5.channel做了什么事情？





------

















# ndt_node

## ndt_node输入输出



### 创建：

```
m_pub_earth_map = createPublisher<tf2_msgs::msg::TFMessage>("/tf_static",caros::QoS(10));
```

### 输入：

```c++
//通过ndt::load_map加载pcd文件

void NDTMapPublisherNode::run()
{ 
  //***将PCD文件加载到PointCloud2消息中
  ndt::geocentric_pose_t pose = ndt::load_map(m_yaml_file_name, m_pcl_file_name, m_source_pc);

  //***应用Quaternion变换加载的PointCloud2消息
  publish_earth_to_map_transform(pose);

  m_ndt_map_ptr->insert(m_source_pc);
  m_ndt_map_ptr->serialize_as<SerializedMap>(m_map_pc);

  if (m_viz_map) {
    reset_pc_msg(m_downsampled_pc);
    downsample_pc();
    // if (m_downsampled_pc.width > 0U) {
    //   m_viz_pub->publish(m_downsampled_pc);
    // }
  }
  publish();
}
```

### 输出：

```c++
tf2_msgs::msg::TFMessage static_tf_msg;
...
m_pub_earth_map->write(static_tf_msg);
```

------













# lanelet2_map_provider

## mapping/hadmap/lanelet2_map_provider输入输出



### client接口:

```c++
m_client = this->create_client<autoware_auto_mapping_msgs::srv::HADMapService>("HAD_Map_Service");
```

#### 请求_输入：

```c++
void Lanelet2MapVisualizer::visualize_map_callback(
rclcpp::Client<autoware_auto_mapping_msgs::srv::HADMapService>::SharedFuture response)
	{
	解析map msgs
	对map msgs进行设置
	添加到visualization_msgs::msg::MarkerArray map_marker_array并发布出去
	}
```

### service接口:

```c++
m_map_service =
this->create_service<autoware_auto_mapping_msgs::srv::HADMapService>(
"HAD_Map_Service", 
std::bind(&Lanelet2MapProviderNode::handle_request, this, std::placeholders::_1, std::placeholders::_2));
```

#### 服务_输出：

```c++
  void Lanelet2MapProviderNode::handle_request(
  	std::shared_ptr<autoware_auto_mapping_msgs::srv::HADMapService_Request> request,
  	std::shared_ptr<autoware_auto_mapping_msgs::srv::HADMapService_Response> response)
  	{
  	添加地图信息版本和格式到message header
  	检测地图设置
  	发送地图设置
  	...
  	}
```

------





回调函数注册：

caros_node_service.cpp

line 106:

```c++
this->registerHandle(std::dynamic_pointer_cast<caros::NodeService>(shared_from_this()), handle.get_handler_name());
```













# schedule.cpp

## void Scheduler::Init(const std::string& use_scenario_name);



加载json文件夹下的配置文件

```c++
CarosConfig::JsonLoader::HostConfLoader json_profile(this->host_conf_path.c_str());
_nodes_profile = json_profile.get_node_list();
_event_profile = json_profile.get_event();
_topic_profile = json_profile.get_topic();
```



等待node init

```c++
SyncNodeConfig(); 
```



```c++
void Scheduler::SyncNodeConfig()
{
    auto back_list = _node_name_list;
    // NodeStatus all=NodeStatus::READY;
    // wait all init msg off all nodes
    while(!back_list.empty())
    {
        //***遍历列表中的node
        for(auto item = back_list.begin(); item != back_list.end(); )
        {
            std::string msg;
            if(_chnlReader_sp_->recv(IPCMSGTOSCHEDULE(*item),msg,0))
            {
                auto pkgrcv = std::make_shared<Package>(); 
                pkgrcv->UnPackRecvData(msg.c_str(),msg.length());
                if (pkgrcv->getCmdType() == (CAROS_CMD_TYPE::CAROS_CMD_INIT|CAROS_INIT_CMD::CAROS_CMD_INIT_CONF_UP_REQ))
                {
                    auto node_profile_ptr = std::dynamic_pointer_cast<CarosConfig::NodeProfile>(pkgrcv->getData());
                    if (node_profile_ptr.get())
                    {
                        //sync node attr
                        CarosConfig::Node::NodeAttributes nodeAttr = node_profile_ptr->node_;
                        OnSyncNodeAttr(std::make_shared<CarosConfig::Node::NodeAttributes>(nodeAttr));

                        //sync event attr
                        CarosConfig::Event::EventAttributes eventAttr = node_profile_ptr->event_;
                        OnSyncEventAttr(std::make_shared<CarosConfig::Event::EventAttributes>(eventAttr));

                        //sync topic attr
                        CarosConfig::Topic::TopicAttributes topicAttr = node_profile_ptr->topic_;
                        OnSyncTopicAttr(std::make_shared<CarosConfig::Topic::TopicAttributes>(topicAttr));
                    }                  
                    
                    // response to node
                    std::shared_ptr<NodeCtrlMsg> response=std::make_shared<NodeCtrlMsg>();
                    response->setNodeStatus(NodeStatus::READY);
                    auto pkgsed = std::make_shared<Package>(CAROS_CMD_TYPE::CAROS_CMD_INIT|CAROS_INIT_CMD::CAROS_CMD_INIT_CONF_UP_RSP, response); 
                    auto [buf,len] = pkgsed->PackSendData();
                    _chnlWriter_sp_->send(IPCMSGTONODE(*item),buf,len);
                    std::cout<< "response to node init : " << *item <<std::endl;
                }

                item = back_list.erase(item);
            }
            else
            {
                std::cout<< "wait node init : " << *item <<std::endl;

                ++item;
            }
        }
        usleep(10000000);  // 1/1000*000	// 进程挂起一段时间， 单位是微秒（百万分之一秒）
    }
}
```



## void Scheduler::Run()

------





# 2022.08.08

1.ndt_node移植，目前思路有点乱

编译通过，但是运行ndt_map_publisher_exe时，错误：

![1](caros_porting.assets/20220810-014524)

ldd ./build/ndt_map_publisher_exe

![2](caros_porting.assets/20220810-014937)



2.重新看了一下ndt_node的代码，其中#include <rclcpp_components/register_node_macro.hpp>

用到了ros2的components，又网上简单了解一下components的概念



3.看了一下autoware中关于client和service的packages，发现也是比较复杂，有些package中的create_client是通过rclcpp::action去实现的，有些package的src下面的文件比较多，又被include当中，有些复杂，目前梳理的比较乱



4.我从ros2官方中，下了examples的package，对应的都是比较小的demo，打算先把这里面的topic	timers	services，移植到我们的caros，

![3](caros_porting.assets/20220810-024516)











