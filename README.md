
## ros-回调函数
针对存在多个回调函数，由于默认只有单线程轮流处理回调函数，尤其当存在大数据消息或耗时处理时，耗时处理如果放在回调函数中处理则会导致其他回调函数等待无法及时收到消息，因此建议：  
（1）回调函数中只负责：接收数据、数据预处理（类型转换/滤波等）、写到指定数据对象中  
（2）使用独立线程对数据对象的大数据量数据且耗时计算进行处理  
参考：  
  https://blog.csdn.net/mxdsdo09/article/details/103589134  
示例：  
  cartographer_ros接口的各订阅器只负责接收及转存各传感器数据，具体的数据后处理由cartographer库处理；  
  
## ros-spin() spinOnce() ctrl+c ros::ok() 
如何更优雅且实用的结束ros节点（进程）？   
传统结束ros节点方式为在终端按下ctrl+c，此时只会调用ros::shutdown()来停止订阅发布服务定时器等；   
实际情况多为在while循环（往往负责实时处理传感器数据）结束后希望做一些必要的结束前的工作，例如保存处理后的数据到本地（cartographer建图结束后保存pbstream文件）；  
方法：  
使用signal捕获ctl+c，在signal回调函数中做结束前的准备工作最后调用ros::shutdown()结束ros相关。   
```
#include <signal.h>
void MySigintHandler(int sig)
{
	//这里主要进行退出前的数据保存、内存清理、告知其他节点等工作
	ROS_INFO("shutting down!");
	ros::shutdown();
}
int main(int argc, char** argv){
	while(ros::ok() && sec++ < 5){
		loop_rate.sleep();
		ROS_INFO("ROS is ok!");
		ros::spinOnce();
	}
	//ros::ok()返回false，代表可能发生了以下事件
		//1.SIGINT被触发(Ctrl-C)调用了ros::shutdown()
		//2.被另一同名节点踢出 ROS 网络
		//3.ros::shutdown()被程序的另一部分调用
		//4.节点中的所有ros::NodeHandles 都已经被销毁
	//ros::isShuttingDown():一旦ros::shutdown()被调用（注意是刚开始调用，而不是调用完毕），就返回true
	//一般建议用ros::ok()，特殊情况可以用ros::isShuttingDown()

	ROS_INFO("Node exit");
	printf("Process exit\n");
	return 0;
```

参考：   
http://wiki.ros.org/roscpp/Overview/Callbacks%20and%20Spinning  
https://www.csdn.net/tags/NtzaggwsNzc2MTAtYmxvZwO0O0OO0O0O.html  
  
