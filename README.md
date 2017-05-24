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
![](http://img.blog.csdn.net/20160515164619930?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

图6是一个数据帧的示意图。其中绿色标识隐性，黑色表示显性，黄色标识显性或隐性（图7），以下相同。

一个完整的数据帧有7部分组成，依次为帧起始(SOF)、仲裁场(Arbitration Field)、控制场(Control Field)、数据场(Data Field)、CRC场、应答场(ACK Field)、帧结尾(EOF)。

帧起始是数据帧和远程帧开始的标志，它是一个显性位。一个CAN节点只有在总线处于空闲状态时才可以发送帧起始。
![](http://img.blog.csdn.net/20160515164624539?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
仲裁场在帧起始之后，控制场之前，共12位（注：协议的讲解以CAN2.0A为基础[3]，CAN2.0B版本的仲裁场为32位[4]）分为两部分11位的标识符和1位的远程发送请求位（RTR）。在数据帧中RTR为显性，在远程帧中RTR为隐性。所以如果相同标识符的数据帧与远程帧发生冲突，数据帧优先。
![](http://img.blog.csdn.net/20160515164633415?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
仲裁场之后便是控制场。控制场的头两位为保留位，为隐性。后面是数据长度代码（DataLengthCode）。数据长度代码指示了数据场中字节的个数。图10说明了数据长度的大小在DLC的表示。DLC最大为8。
![](http://img.blog.csdn.net/20160515164638102?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
 对于超出8的情况，各厂家有不同的实现。有的实现忽略“越界”DLC，传输8 bytes的数据和“错误”的DLC。有的传输8 bytes的数据并改DLC为8。有的直接不传输任何东西。

数据场在控制场之后，传输数据的长度由DLC决定。如果DLC为0，则没有数据场。数据场中高位先传输。
![](http://img.blog.csdn.net/20160515164708650?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
 CRC场在控制场和数据场之后，由CRC序列和界定符组成。CRC序列是帧起始，仲裁场，控制场和数据场组成的位流的CRC校验值。其中CRC校验的生成多项式为X15+ X 14+ X10+ X8+ X7+ X4+ X3+ 1。CRC序列之后是一个“隐性”CRC结束符。
![](http://img.blog.csdn.net/20160515164713759?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
 CRC场之后便是应答场。应答场由2个位组成，应答位和应答结束符。发射单元会发送“隐性”的应答位和应答结束符至总线上。而接收单元如果接收到的数据都是有效的，会在发射单元发送应答位的同时发送一个“显性”位至总线上，所以一个有效的数据帧，应答位在总线上应该表现为“显性”。

#### 远程帧
![](http://img.blog.csdn.net/20160515164909652?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
远程帧的主要作用是向其他的CAN节点发送数据请求，发送相同标识符的数据帧。与数据帧相比，远程帧的RTR位是隐性的，而且没有数据场。DLC中的值是数据帧的数据长度。

#### 错误帧
![](http://img.blog.csdn.net/20160515164931666?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
 错误帧由错误标志的叠加和结束符组成。

错误标志有主动错误标志与被动错误标志。主动错误标志为6个显性位，被动错误标志为6个隐性位。

错误主动节点与错误被动节点（参考“CAN节点的错误状态”）对错误的反应是不一样的。

当错误主动节点检测到错误时，会发送主动错误标志。而主动错误标志又会影响总线上原有传输内容的结构，从而让其他未检测到错误的节点发现错误。一种情况是错误帧破坏了应答场和帧结尾的固有形式；另一种情况是错误帧破坏了位填充规则。当其他节点发现错误后，也会发送错误帧。这样就会造成一个错误标志的叠加会有6-12bits大小。
![](http://img.blog.csdn.net/20160515164946169?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

图15就演示了第二种情况时的各个节点发送错误帧的情况。节点1首先检测到错误，发送错误帧，在连续发送了6个显性位之后，节点2和3检测到位填充错误，也发送错误帧。这样总线上错误帧的叠加就达到了12位。

所有节点发送完错误标志之后就会发送一个隐性位，并监控总线，直到总线上出现一个隐性位。然后在发送7个隐性位。这样一个错误帧就发送完毕了。
![](http://img.blog.csdn.net/20160515165004653?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

过载帧与主动错误帧非常类似，特别是位的组成和全局化的过程。主要的差别在于错误帧发生着数据帧，远程帧期间。而过载帧发生于间歇字段期间。

过载帧是由过载标志的叠加和过载结束符组成。有两种情况可以触发过载帧：

    +CAN节点的内部需求，例如需要时间准备数据帧的数据。这种情况下过载帧只允许起始于帧间隔的第一个位。
    +在帧间隔内侦测到显性位。这种情况下，过载帧起始于检测到显性位的后一位。

过载标志由6个显性位组成，过载帧破坏了间歇字段的结构从而导致了过载帧的全局化。发完过载标志后，CAN节点会往总线发送隐性位，并监控总线直至出现隐性位。然后再发送7个隐性位。

#### 帧间空间
![](http://img.blog.csdn.net/20160515165016732?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
![](http://img.blog.csdn.net/20160515165029904?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
 数据帧与远程帧的前面必然有帧间空间。对于主动错误节点和被动错误节点，帧间空间的结构稍有不同。对于主动错误节点，帧间空间由3个显性位的间歇字段和总线空闲组成。在间歇字段不允许发送数据帧与远程帧。总线空闲的长度任意，当有显性位时就被认为是帧起始。

被动错误标志除了上边两部分外，在间歇字段之后还有8个显性位的挂起传输。在挂起传输阶段被动错误节点不可以发送数据帧与远程帧。

### 通讯对象
CANOpen协议共有6种通讯对象，分别是：PDO、SDO、SYNC、TIME、EMCY、NMT。这6种通讯对象完成了CANOpen协议的所有通讯功能。其中我们只介绍使用较多的PDO、SDO、NMT。
#### 通讯对象ID(COB-ID)
CANOpen协议的通讯对象主要利用了CAN协议中的数据帧和远程帧。为了区分不同的通讯对象，CANOpen协议利用数据帧/远程帧中的ID。其中第7位到第10位为功能代码。第0位到第6位为节点ID，用以区分不同节点的相同功能。这样就允许最多127个从节点与主节点通讯。
![](http://img.blog.csdn.net/20160515192132336?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

下面是预定义的各通讯对象的COB-ID
![](http://img.blog.csdn.net/20160515192147664?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

其中绿色部分为广播的通讯对象，蓝色部分为点对点的通讯对象。

COB-ID的大小也决定了通讯对象的优先级，其中NMT的优先级最高，PDO的优先级高于SDO。

#### Process Data Object
CANOpen中的实时数据传输是由PDO来完成的。PDO的传输采用了生产者消费者模式。共有两种PDO，TPDO和RPDO。TPDO用来传输数据，支持TPDO的节点都是PDO数据的生产者。RPDO用来接收PDO数据，支持RPDO的节点是PDO数据的消费者。从表3可以看出，一个节点最多支持4个TPDO（分别是180h+NodeID、280h +NodeID、380h +NodeID、480h +NodeID）和4个RPDO（分别是200h +NodeID、300h +NodeID、400h +NodeID、500h +NodeID）。每一个PDO都对应一些参数，包括通讯参数和映射参数。
##### PDO参数
PDO的参数包括两部分，通讯参数和映射参数.他们占据了对象字典中从1400到1BFF之间的位置.
![](http://img.blog.csdn.net/20160515192202073?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

PDO的通讯参数定义了COB-ID,传输类型(同步,异步,循环,时间出发),inhibit time(两个PDO的最小间隔)等,见表5.
![](http://img.blog.csdn.net/20160515192217339?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

表6给出了PDO的映射参数.一个PDO最多可以映射到64个对象。每一项的含义见图29。最高16位是对象字典的索引，后面8位是子索引。最低8位是数据长度。
![](http://img.blog.csdn.net/20160515192227618?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
![](http://img.blog.csdn.net/20160515192237946?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

##### PDO映射
PDO的内容没有固定的形式，它的数据段可以是1到8个字节长。使用PDO的一个基本的出发点是发射端和接收端都知道PDO中的数据的含义。图30说明了具体的工作原理。PDO采用了生产者和消费者模式。生产者按照TPDO的映射参数从对象字典中抽取数据形成PDO的数据。当消费者接收到PDO的数据后，就会按照RPDO的映射关系，将PDO中的数据解析出来，填入对应的数据字典中。

图31给出了生产者生成PDO0数据的具体过程。首先生产者从[1800,01]中得到COB-ID。然后从[1A00,01]中获取第一个对象。[1A00,01]中的值为0x20000108h，那么第一个对象为[2000,01]，对应值为A,在PDO中占0x8位。接着是第二个对象[2003,03]，对应值为G，在PDO中占0x10位。最后是对象[2003,01]，对应值F，在PDO中占0x8位。这样一个有数据A、G、F组成的，32位的PDO就形成了。

#### SDO
SDO(Service Data Object)使用Client-Server模式建立起点到点的通讯并实现了对对象字典中条目的读写。其中被访问的对象字典的所在设备作为Server，访问对象字典的设备作为client。参考下图。SDO采用的请求应答模式，每次SDO访问都会有2条CAN的数据帧对应。一条是请求，一条是应答。
![](http://img.blog.csdn.net/20160515192526844?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

SDO主要提供3种服务：段传输，块传输，中止传输。下面分别来介绍这三种服务是如何实现的。

#### 段传输
下面是SDO段传输Download过程中的示意图。首先由client端发起，然后Server端应答。这样一来一回。知道把数据传输完毕。当传输的数据长度小于4时，一次应答就可把数据传输完毕。
![](http://img.blog.csdn.net/20160515192543266?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
![](http://img.blog.csdn.net/20160515192558183?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
![](http://img.blog.csdn.net/20160515192610839?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
图34、35标出了在SDO段下载的初始化和数据传输阶段的命令字。其中：

* ccs: client命令标识符
    + 0:段下载请求
    + 1:初始化下载请求
    + 2:初始化上传请求
    + 3:段上传请求
* scs: server命令表示符
    + 0:段上传响应
    + 1:段下载相应
    + 2:初始化上传响应
    + 3初始化下载响应
* n:指示不含数据的字节数.
* e:传输类型
    + 0:普通传输
    + 1:快速传输(输出长度<=4)
* s:大小指示器
* m:指示数据的索引和子索引.
* d:数据
* X:总是0
* reserved:预留，总是0
* c:指示是否还有数据需要下载/上传
* t:触发位。这个位没发送一次数据会反转一下。
SDO段上传的过程与段下载的过程类似，只是命令字不同。
![](http://img.blog.csdn.net/20160515192624595?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
![](http://img.blog.csdn.net/20160515192643752?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

##### 快传输
块传输的主要目的是为了提高传输效率。它与段传输的主要区别在于：块传输时，可以传输多次数据之后，才会有一次应答，如图38。CANOpen将数据分为多个block，每个block又由1-127个segment组成。在传输完一个block的数据之后，才会有一次的应答。
![](http://img.blog.csdn.net/20160515193011271?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

##### 中止传输
无论CAN设备处于段传输还是块传输中，都可以使用中止传输协议来中止传输。
![](http://img.blog.csdn.net/20160515193028060?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
协议的命令字解释如下：

 * cs:命令标识符
    + 4:中止传输请求
 * X:总是0
 * m:标识索引和子索引.
 * d:中止码（表7中止码表）.
下表是对各中止码的解释

![](http://img.blog.csdn.net/20160515193039810?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
