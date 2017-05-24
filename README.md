# CAN与CANopen
---
选自：[maifansnet的博客](http://blog.csdn.net/maifansnet/article/details/48950615)

## 基本概念
CAN(Controller Area Network, 控制器局域网络)是由博世开发的一种现场总线，首先应用在汽车领域。由于它的低成本和可靠性，现在被广泛应用在工业测控和工业自动化领域。

### CAN与CANOpen的关系
下面是CAN协议与OSI网络模型的一个对比。CAN的物理层分了三层分别是MDI，PMA和PLS，数据链路层分了两层：MAC与LLC。这五层就是最原始的CAN协议，标准是ISO11898。也就是说CAN协议一开始是没有应用层的。后来有一种叫CANOpen的基于CAN的应用层协议被开发出来，标准是CiA301。

在实际开发CAN器件的时候不一定要用CANOpen，你可以根据自己的需要定制自己的应用层协议。
![](http://img.blog.csdn.net/20151007202502666?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center "CAN与CANOpen的关系")

### CAN的基础知识
#### 显性和隐性
显性(Dominant)与隐性(Recessive)是总线上最基本的两个状态，也可以表示为“0”与“1”。在物理上它有两条线的压差表示。在隐形的时候，两条线的电压相同，压差为0。当压差超过一定的阈值的时候，总线的状态就变为显性。
![](http://img.blog.csdn.net/20151007202525236?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center "显性与隐性")
为什么叫显性和隐形？

假设在总线上挂了2个器件1,2.器件1将总线设为显性，而同时器件2将总线设为隐形。最终总线的状态会呈现为显性。所以当总线上的所有器件都为隐形时，总线的状态才为隐形。如果有一个器件为显性，则总线为显性
![](http://img.blog.csdn.net/20151007202546627?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center "CAN总线")

#### 冲突裁决
当总线上的几个器件同时发送数据的时候，CAN总线必须决定哪个器件可以发送，而其他的器件必须等待。冲突裁决是CAN协议最重要的一个特性，也是CAN总线做的最漂亮的地方，用很小的成本就解决了这个问题。

总线上的每一个CAN器件都会有一个唯一的ID。ID的大小决定了器件的优先级。ID越小优先级越高。如果几个器件同时发送数据，ID小的优先发送。以下图为例，总线上有A，B，C三个器件。A首先发送数据。当总线上有器件发送数据时，其他器件只能处于监听模式，所以B，C虽然有发送数据的需求但是只能等待A发送结束。当A发送结束之后，B，C同时发送，但是B的ID更小，B优先发送。B发送结束之后C才可以发送
![](http://img.blog.csdn.net/20151007202615282?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center "冲突处理与Node ID")

下面说明一下当几个器件同时发送时CAN总线是如何做裁决的。

以下图为例。总线上有器件A，B，C，D。A，B，C同时发出SOF位为显性。而D为隐形，当它发现总线上的状态与自己的状态不一致时，D就进入监听状态。A，B，C继续发送数据。发送到ID的第5位时A，C为显性，B为隐形。B检测到总线的状态于自己的状态不一致,进入监听状态。A,C继续发送数据。这也说明B的ID比A，C要大。当发送到ID的第1位时A为隐形，C为显性，A进入监听状态。C继续发送。最终ID最小的C发送成功，A，B只能等待C发送完成之后再进行发送。然后A会发送成功，B等待，最后才是B发送。从上面的裁决过程可以看出，对于C来说，它的数据发送没有因为冲突而产生延迟。

![](http://img.blog.csdn.net/20151007202620019?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center "冲突处理细节")

## 报文格式
### 帧
 CAN协议的报文传输主要由下面的4种帧来实现：

    +数据帧：从发射端携带数据到接收端。
    +远程帧：总线单元发出远程帧，请求发送具有同一识别符的数据帧。
    +错误帧：任何单元检测到一总线错误就发出错误帧。
    +过载帧：过载帧用以在先行的和后续的数据帧（或远程帧）之间提供一附加的延时。

同时帧间空间用来间隔数据帧/远程帧与其他帧。
#### 数据帧
![数据帧](http://img.blog.csdn.net/20160515164555430?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center “数据帧”)

