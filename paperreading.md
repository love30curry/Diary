# CBMA: Coded-Backscatter Multiple Access


**2023.9.13**

**`1. introduction`**  
![image](https://github.com/love30curry/Diary/assets/144334874/d6aa7d51-3814-4142-98f7-7dff5331ec20)
据我们所知，现有的后向散射系统很少同时考虑到这两种要求(见表1)。BackFi[2]侧重于提高数据速率范围可达1米。FM反向散射[3]、LoRa反向散射[4]和PLoRa[5]能够实现远距离反向散射通信，但没有考虑多址和高数据速率。Netscatter[6]部署了大量数据速率有限和/或通信范围较短的标签。此外，如何对标签进行分散控制在这些工作中是缺乏讨论的
因此，本文在直接序列扩频(DSSS)的启发下，提出了一种反向散射通信的编码-反向散射多址(CBMA)方案来满足这两种要求。DSSS具有抗衰落和支持多个标签同时传输的能力。然而，为了设计和实现反向散射的CBMA方案，我们面临以下挑战:

- 异步信号。通过发送控制信号来协调所有标签是困难的，这导致标签之间的传输是异步的。异步问题严重影响了扩频序列的正交性，使得传统的信号分离方法不适合进行信号分离。

- 接收功率多样化。反向散射信号的强度本质上是微弱的，并且受到标签和接收器之间距离的强烈影响。因此，接收到的信号强度在标签之间是不同的，由于传统码分多址(CDMA)系统中众所周知的近距离问题[7]，这会导致显著的性能下降。使问题更具挑战性的是，由于标签本身不直接产生RF信号，因此标签上的功率控制非常困难

我们做出了以下贡献:•CBMA是第一个倡导与CDMA方案同时进行标签反向散射传输的设计。我们证明了多个标签可以使用所提出的方法同时反向散射信息，开销很小，并且内容可以在不影响原始WiFi通信的情况下由商品WiFi网卡解码。我们已经用10个标签的试验台实验验证了性能。

- CBMA提出了第一个无源标签的电源控制方案和硬件实现。它可以调节标签天线的阻抗来控制功率，从而实现更有效的反向散射。在此基础上提出了一种节点选择方法，通过实现自适应复用方案来提高容量。

- 我们用现成的硬件构建了CBMA的原型。它实现了高达8Mbps的多标签比特率，标签与接收器的距离可达5米。与单透镜解决方案相比，在具有障碍物和干扰的挑战性室内场景中，CBMA可以将后向散射吞吐量提高10倍以上。


**`2. 反向散射通信和多址的初步知识`**  
- B.反向散射通信的多址接入
   在传统的反向散射通信中，很少考虑多个标签在同一频段同时反向散射信息的情况。然而，在密集物联网系统中，多标签反向散射通信的设计非常重要。由于标签的能力有限(标签上没有ADC，信号强度未知)，在标签 
   侧  进行载波感知是不可行的，这就导致了与传统的多用户通信完全不同的情况。通常，在反向散射通信中，可以采用几种方法来避免标签之间的冲突。
   基于cdma的多址就是其中之一。在该方案中，每个标签都有一个局部的“伪噪声”(PN)代码来传播其信息。PN码被称为“伪随机”，因为代码是可预测的和重复的，尽管它看起来是随机噪声。在标签处，信息的每个位乘以 
   与信息无关的PN码，以产生反向散射的编码序列。在接收端，通过将接收到的数据与相同的PN码相乘来重构原始数据。由于PN码之间的正交性，使得具有不同PN码序列的干扰最小。因此，通过在不同的标签上使用不同 
   的PN码来实现多路访问。


**`3. CBMA的系统概述`**  
A.反向散射标签
CBMA系统中的反向散射标签执行以下操作，包括分帧、编码、功率选择、上采样和频移过程。
- 框架。被传输的标签的数据首先被封装成具有以下字段的帧:(1)一个字节的已知前导{10101010};(2)表示所述帧长度的一个字节数据;(3)多达126字节的有效载荷数据和(4)两个字节的循环冗余检查，以验证是否发生了错误

- 编码。然后由编码块使用在标签处确定生成的PN代码处理结构化框架。然后将数据乘以PN码。在这里，我们给出一个简单的示例来说明编码过程。假设标签要传输“10”，采用PN码“01001”，则编码数据的结果为“0100110110”。在这项工作中，我们考虑了两种类型的PN码，分别是Gold码[8]和2NC码[9]。
- 功率控制。系统性能受每个标签接收功率的影响很大。基于传统CDMA系统的原理，最好的情况是每个标签接收到的功率保持在同一水平。因此，功率控制对CDMA系统至关重要。研究发现，后向散射信号的强度高度依赖于标签天线的阻抗，因此可以通过改变阻抗来调节后向散射功率。我们提出了一种有效的功率选择算法，通过自适应调整标签天线的阻抗来优化系统的整体容量

- 开/关调制和移频。然后使用特定PN码的编码序列对后向散射信号进行开/关调制。由于标签没有射频前端来调制其数据，因此使用振荡器产生的方波来控制天线的反射状态。方波可以随激励信号调制，产生频移后向散射信号。如果标签想要传输' 1 ' / ' 0 '，它启用/禁用控制天线状态的方波周期为一个符号时间(在我们的配置中为1μs)。因此，我们在没有射频前端的情况下实现了开/关调制。

B.接收器接收器以中心频率收听频道，中心频率是移位的频率。如果检测到数据帧，接收端以采样频率fs进行采样，并开始接收过程，包括帧同步、用户检测、解码和确认。

- 帧同步。通过滑动窗口的能量检测实现帧同步。具体地说，首先对接收的窗口大小为Wn的能级进行移动平均滤波。然后将滤波后的序列通过比较器，通过比较当前功率电平和滤波后的功率电平来确定是否接收到新帧。我们使用决策阈值Pth，它被配置为比滤波功率电平高3dB。

- 用户检测。为了解码进入的帧，有必要确定帧中包含哪个PN序列。
我们利用PN序列间的正交性特征进行用户检测。具体来说，我们使用每个PN序列与接收帧的前导交叉相关。如果某个PN序列的相关值大于预先设定的阈值，则判定该PN序列的用户大概率出现在帧中。

- 解码。在用户检测后，我们使用检测到的用户的PN序列与同步帧中的每个芯片(表示一个比特的扩展符号)进行相互关联。如果与表示“1”的PN序列的相关性大于与表示“0”的PN序列的相关性，则芯片被解码为“1”，反之亦然。
![image](https://github.com/love30curry/Diary/assets/144334874/5538abd5-4fa6-4a77-9d47-c3aa888ed7fc)




**`4. 提供了一个测量来证明为什么需要功率控制。`**  

在实际的后向散射系统中，碰撞解码的挑战主要在于标签的功率不同。从理论上讲，当多个标签发生碰撞时，如果标签的功率是线性组合的，我们可以通过互相关来解码每个标签的数据。然而，当标签之间的功率差异很大时，我们观察到解码性能很差。

因此，它提出了一个难题:如何解决标签之间的巨大功率差异?我们的想法是通过调整标签天线阻抗来控制反向散射功率。然后，我们在双标签碰撞情况下进行基准实验。我们构建一个如图3所示的坐标系统。标记为A和B的点表示激励源“Es”和接收器“Rx”。标记为O的点是这个坐标系的原点。然后，我们将激励源和接收器分别放置在(- D, 0)和(D, 0)位置(在我们的实现中D = 50cm)。同时，我们将(x1, y1)和(x2, y2)表示为“T ag1”和“T ag2”在一次测试中的位置。对于每个测试，我们从五个标签中选择两个(表2中由1、2、3、4和5表示)并随机放置它们。

由于篇幅所限，我们仅在表二中给出部分结果。差值计算为两者的功率差与较大的功率之间的比率。错误率计算为丢失的数据包数除以传输的数据包总数。

我们发现当功率差小于10%时(两个标签的功率相似)，错误率要低得多。

因此，我们可以利用这些结果来执行功率控制以提高整体性能。我们提出了我们的电源控制方案，并在下一节中介绍了细节。

**`5. CBMA的详细设计`**  

![image](https://github.com/love30curry/Diary/assets/144334874/1a29ee4b-0e5f-41c2-b50c-29e8daa8379d)
- A.标签上的通信
  如第三节所述，如果标签要传输“1”，则使方波在一段符号时间内控制天线的状态。否则，标记保持沉默和什么也不做。具体来说，我们有一个双层调制来实现标签上的反向散射通信。第一个是通过发送前一节中提到的方波来产生Δf频移。另一种开关键(OOK)调制是用标签传输数据位。
在这种调制中，方波充当载波。
在特定的持续时间内，载体的存在表示二进制的“1”，而在相同的持续时间内，载体的不存在表示二进制的“0”。因此，在接收端，接收信号可以解码为“1”，没有信号可以解码为“0”。在实际操作中，我们对上采样后的方波和数据流进行“与”运算，如图41所示。
要改变比特率，唯一需要改变的是标签反射(ON)和吸收信号(OFF)的时间周期。


- B.阻抗选择
功率控制方案:我们接收I-Q空间的后向散射信号:I(t)和Q(t)。接收信号的功率为P(t) = (I2(t) + Q2(t))。由于我们的采样率高于比特率，我们首先对接收到的数据进行下采样。每个标签都有自己的PN码。当接收方检测到一个标签的前导时，它向这个标签发送一个ACK包。因此，当标签收到的ACK反馈报文很少时，我们认为通过该标签传输的大部分报文都丢失了。原因是后向散射信号的功率太低，接收器无法检测到。为了提高传输性能，我们必须增加功率。如上所述，我们可以改变天线阻抗来调整反射系数Γ *以实现功率控制。我们给出了算法1中功率控制算法的伪代码，省略了下采样和译码过程。在我们的实验中，循环执行功率控制以尝试每个可能的功率电平。为了避免我们的功率控制方案陷入无限循环，我们将执行周期的数量限制为标签数量的3倍。
- 
ACK（Acknowledgment）消息是在通信系统中用于确认接收到数据包或请求的一种消息。它用于表示成功接收和理解了之前发送的数据或请求，并向发送端发送确认的消息。
当发送端发送数据包或请求时，接收端会接收并处理这些信息。如果接收端成功接收并正确处理了发送端的消息，它会发送一个ACK消息作为确认。ACK消息通常是简短的消息或信号，表明接收端已经收到了发送端的消息并进行了相应的处理。

- C.节点选择
  然而，即使采用了上述功率控制方案，仍有部分标签无法接收到ACK消息，出现了错误比率仍然很高。原因是双重的。首先，反向散射信号比激励信号弱得多。如果一些标签离接收器很远，即使我们通过调整阻抗将标签功率设置为最高可能值，它们的功率也会变得太弱而无法在接收器上检测到。
其次，当两个标签在物理上彼此靠近时，它们会相互干扰，导致系统性能不佳。在这两种情况下，调整功率水平没有多大帮助。
为了解决我们的功率控制方案的局限性，我们提出了另一种节点选择的优化方案。如果系统性能不能满足功率控制的期望，我们放弃那些成功ACK反馈率低于70%的“坏”标签。因此，主要的问题是如何重新选择标签。在我们的系统中，通信范围取决于两个因素:(i)激励源与标签之间的距离;(ii)标签与接收器之间的距离。具体来说，接收机处的信号强度Pr可以用Friis路径损耗模型表示如下


**`7. PERFORMANCE EVALUATION`**  
- 1)帧检测:我们首先关注系统的帧检测。作为核心组件，它对整个系统的性能有很大的影响。帧检测受距离、ES的传输功率、前导长度和比特率等因素的影响。
为了评估这些因素的影响，我们在办公环境中放置了一个激励源、一组标签和一个接收器，如图7所示。
距离的影响。首先研究了帧检测错误率随距离的变化规律。在实验中，我们将ES-to-tag的距离固定为50cm，将tag-to-RX的距离从10改变为400cm，步长为10cm。实验中的标签数设置为2、3、4。在每种情况下，我们收集1000个碰撞数据包并测量帧错误率(FER)。结果如图8(a)所示，我们可以看到:i)当距离大于2m时，随着距离的增加，FER略有增加。与传统的无线系统相比，这个错误率有点高。这是因为反向散射信号的强度远低于传统的无线信号;Ii)当距离小于2m时，错误率几乎保持恒定值，错误率取决于标签。正如预期的那样，2标签案例在这些实验中获得了最低的FER

- POWER的影响。然后，我们评估了改变后向散射功率的性能。然而，直接改变后向散射功率是一个挑战。因此，我们改变了激发源的发射功率，它与后向散射功率线性相关。根据Friis路径损耗方程，后向散射功率与激励源功率呈线性关系。因此，我们根据ES的传输功率来评估帧检测的准确性。在实验中，我们将标签数量从2个改变为4个，传输功率从-5dBm改变为20dBm，步长为5dBm。
我们为每种设置收集了1000个碰撞数据包，并在图8(b)中显示了相应的错误率。我们发现，随着传输功率的增加，误码率降低。当发射功率低于0dBm时，后向散射信号非常微弱，容易被环境噪声所掩盖。从图8(b)中可以看出，当传输功率降低到-5dBm时，误码率非常高。

- 前言长度的影响。为了研究前导长度的影响，我们将前导长度配置为4、8、16、32和64位。对于每种情况，我们使用2到4个不同数量的标签进行实验。图8(c)显示了帧检测的错误率。我们可以看到，前导长度对帧错误率有显著影响。随着序文长度的增加，错误率降低。在4标签碰撞的情况下，当序言长度设置为64bit时，错误率可以在1%以下。

- 比特率的影响。然后我们研究了比特率对帧检测的影响。由于接收器的采样能力有限，当标签以特别高的比特率传输时，每个信号状态的停留时间很短，可能导致采样点过少，影响帧检测的性能。在实验中，标签的数量从2到4不等。在每种情况下，比特率从250Kbps变化到5Mbps。结果如图9(a)所示。我们可以看到，虽然比特率是影响帧错误率的关键因素，但当比特率为5Mbps时，我们的系统仍然可以获得相当不错的性能

- PN码。在实验中，我们测试了两种类型的PN代码:2NC代码2和Gold代码。我们将并发标签的数量从2个更改为5个。图9(b)展示了不同采用的PN码错误率的比较。
发现随着标签数量的增加，错误率也随之增加。此外，2NC码具有更好的正交性，这使得标签之间的干扰更少，这意味着2NC码可以达到更好的性能。当在5个标签的情况下使用黄金码时，错误率突然上升到11%。我们发现2NC码的性能如预期的那样好于Gold码。因此，我们在接下来的实验中采用了2NC编码。

- 功率控制。为了分析我们的电源控制方案，我们比较了2到5个标签的性能，包括有和没有电源控制。对于每个设置，我们生成50组标签的随机位置。对于每一组，我们分别评估了我们的系统在有电源控制和没有电源控制时的错误率。
图9(c)显示了有电源控制和没有电源控制时错误率的比较。我们发现，无功率控制时，错误率远高于有功率控制时。我们还可以看到，随着标签数量的增加，系统错误率上升。然而，通过功率控制，系统2在我们的实验中，我们修改了2NC代码，其中代表0的芯片是代表1的芯片的负数。
即使有5个标签，错误率也低于5%。在5标签碰撞情况下，系统性能比无电源控制时提高5倍。

# GuardRider: Reliable WiFi Backscatter Using Reed-Solomon Codes With QoS Guarantee


**2023.9.6**

**`1. RS代码的初步知识`**  
**`2. RS码设计的优化算法`**  
RS在骑乘间歇信号的情况下显示了其对错误率的有效改进。由图4(a)-(c)可知，当冗余符号的数量相对足够时，错误率收敛到一定水平，即使冗余符号的数量增加，也不会获得更多的增益。换句话说，如果我们添加太多多余的符号(在一帧内反向散射的信息更少)，它会浪费反向散射链接的带宽。总之，我们必须知道在不同的场景下应该添加多少奇偶校验符号来保护反向散射通信，从而在效率和可靠性之间取得良好的平衡。因此，我们提出了一种优化算法，以搜索最优的RS码，使效率最大化，同时满足可靠性。
本文提出的优化算法包括三个主要步骤:
(1)估计WiFi流量开、关时长的概率分布;
(2)构建马尔可夫模型来描述状态之间的转换;
(4)优化RS代码以满足可靠性约束。

wifi状态的统计信息
为了设计自适应RS编码，标签必须知道states of excitation traffics(on and off duration of wifi traffic ),由于标签功率有限，对接收器优化，并将指示最佳RS代码参数的索引反馈给标签。接收器首先监听所需通道，并根据接收到的信号分别测量开、关状态的持续时间。然后将测量值输入到基于mle的估计器中，以生成每个状态的概率分布。并估计所需信道上wifi流量的开启和关闭状态的持续时间。将数据存储在X=[x1,x2,x3]和Y=[y1,y2,y3]

分析使用RS码的WiFi信号的反向散射通信的错误率
本文采用如图5所示的两态马尔可夫模型来表示状态之间的转换。Glaze[17]系统采用双态马尔可夫模型对WiFi流量进行预测，因此可以将下行数据纳入WiFi数据包。然而，我们的工作考虑的是与Glaze系统不同的反向散射上行链路。这里需要强调的是，马尔可夫模型是设计统计间歇信道上的RS码的关键。设α和β分别表示从开状态到关状态和从关状态到开状态的过渡概率。因此，马尔可夫模型的转移概率矩阵Q由式给出


**`3. GuardRider的系统设计与实现`**  
后向散射系统中的标签通过分帧、RS编码、基带转换、上采样和移频来后向散射其信息
![image](https://github.com/love30curry/Diary/assets/144334874/dfcc12c7-f0d4-4cb5-8130-71f37c48bab9)

让我们按照后向散射系统中的流程，将提供的数据（0，1，0，1，0，0，1，1）通过分帧、RS编码、基带转换、上采样和移频来后向散射。
分帧（Frameming）：将提供的数据分割成较小的帧。在这个例子中，我们可以选择以两个比特为一帧进行分割。因此，我们将数据分为以下四个帧：(0, 1)，(0, 1)，(0, 0)，(1, 1)。
RS编码（Reed-Solomon Coding）：对每个数据帧进行RS编码。对于每个帧，我们可以选择合适的RS编码参数。假设我们选择RS（7， 5）码，其中可以纠正2个比特的错误。对于每个帧，我们将进行RS编码，例如：
对于帧 （0， 1），RS编码后的结果可能是 （0， 1， e1， e2， e3， e4， e5），其中 e1， e2， e3， e4， e5 表示RS码中的纠错码比特。
对于帧 （0， 0），RS编码后的结果可能是 （0， 0， e1， e2， e3， e4， e5）。
对于帧 （1， 1），RS编码后的结果可能是 （1， 1， e1， e2， e3， e4， e5）。
基带转换（Baseband Conversion）：将经过RS编码的数据转换为基带符号序列。这涉及将每个帧中的比特转换为适当的基带符号。在这个例子中，我们简单地将每个比特映射为一个基带符号，例如(0, 1) 映射到 (-1, 1)。
对于帧 (0, 1)，基带转换后的结果可能是 (-1, 1)。
对于帧 (0, 0)，基带转换后的结果可能是 (-1, -1)。
对于帧 (1, 1)，基带转换后的结果可能是 (1, 1)。
上采样（Upsampling）：将基带符号序列的采样率增加。在这个例子中，假设我们将采样率增加两倍。因此，每个基带符号将复制一次，得到更高的采样率。
对于帧 (0, 1)，上采样后的结果可能是 (-1, -1, 1, 1)。
对于帧 (0, 0)，上采样后的结果可能是 (-1, -1, -1, -1)。
对于帧 (1, 1)，上采样后的结果可能是 (1, 1, 1, 1)。
移频（Frequency Shifting）：通过乘法将基带符号序列与载波信号相乘，来移动频率。假设我们选择的载波频率为 f_carrier。根据每个帧的结果和所采用的调制方案，我们可以应用相应的频率偏移和调制参数来移动频率。
对于帧 （0， 1），我们可以将帧的基带符号与频率为 f_carrier 的载波信号相乘。
对于帧 （0， 0），我们同样可以将帧的基带符号与频率为 f_carrier 的载波信号相乘。
对于帧 （1， 1），同样可以将帧的基带符号与频率为 f_carrier 的载波信号相乘。
通过以上步骤，我们将数据(0, 1, 0, 1, 0, 0, 1, 1)成功转换为基带符号序列，并进行了上采样和移频操作，以便在后向散射系统中传输和反射该信息。这样的信号可以由接收端接收、解调和解码，恢复出原始的数据。
请注意，实际的后向散射系统可能会有更复杂的编码和调制方案，并且具体的参数和实现细节可能会有所不同。以上步骤仅提供了一个简单的示例，用于说明后向散射中标签传输信息的过程。

- **`Frequency shift`** 
为简单起见，激励信号可表示为正弦信号，记为sin(2π)。后向散射标签的微控制器产生频率为∆f的方波，在吸收和反射状态之间切换。由于基于著名的傅立叶分析，方波的一次谐波也是正弦信号，因此当应用于输入的正弦信号时，在反向散射链路中实现了频率移位∆f。因此，我们可以使用一阶谐波将方波过程简化为正弦信号，如sin(2π∆f t)。因此，后向散射过程可以用上述两个正弦信号的乘积表示，即2sin (2πfct) sin(2π∆f t) = cos(2π(fc -∆f)t) - cos(2π(fc +∆f)t)。接收机对接收到的后向散射信号进行处理，将其中心频率变换为偏移信号之一fc−∆f或fc +∆f，接收到后向散射链路中的信号。移频的目的是为了消除主链路对后向散射链路的干扰，因为主链路的信号强度明显强于后向散射链路。
然后执行上采样过程以增加基带信号的频率以匹配移位频率，随后在上采样信号与频率为∆f的方波之间进行逻辑与运算。
与操作器的输出信号控制天线使用OOK调制来反向散射信息。
- **`receiver`**
  Receiver在GuardRider中有两项任务，一是优化处理，二是接收反向散射信息。各模块的功能在Labview平台上实现。
Optimizaiton。USRP RIO设备首先监听遗留链路的信道，并以5MHz的采样频率对遗留链路的信道进行同相(I)和正交(Q)采样。然后使用I/Q采样计算信号功率，并计算通断状态的持续时间。最后将持续时间输入到包含三个块的优化算法中，以获得最优RS码参数。
接收反向散射信息。USRP RIO设备被打开以收听移位频率的频道。一旦通过将接收信号的自相关与预定义阈值进行比较检测到帧，设备开始采集I/Q采样，随后进行匹配的滤波过程、解调、同步和RS解码。

复数信号是由实部（Real）和虚部（Imaginary）构成的信号。在数学和信号处理领域，复数可用于表示具有幅度和相位信息的信号。
复数信号可以用以下形式表示：x（t） = r（t）e^（jθ（t）），其中 r（t） 是信号的幅度函数，θ（t） 是信号的相位函数。
实部（Real）代表信号在时域上的振幅变化，虚部（Imaginary）代表信号在时域上的相位变化。复数信号可以通过对实部和虚部的组合来表示时域中的复杂波形。
复数信号在通信系统中具有重要的作用，例如，调制和解调过程中使用的信号通常是复数信号。复数信号通过携带幅度和相位信息，可以实现多种调制方式，例如正交振幅调制（QAM）、相位偏移键控调制（PSK）等。在信号处理领域，对复数信号进行傅里叶变换、滤波、频谱分析等操作也非常常见。
复数信号在电力系统、通信系统、信号处理和控制系统等领域中都扮演着重要角色。它允许我们对信号的振幅和相位进行独立处理，并提供了更丰富的表示和处理信号的方式。
  

**`4. GuardRider的性能进行了评价`**  
- **`a .实验设置`**

我们在一个4 × 6m2的办公室里进行实验，办公室平面图如图8所示。我们使用一台联想笔记本电脑，距离WiFi AP 1米远，生成2048字节的ping请求，间隔100µs。后向散射标签距离WiFi AP和接收天线分别为0.3m和0.6m。我们将传输速度设置为6 Mbps，并使用中心频率为2.422GHz的通道3和20MHz带宽用于传统链路的通信。Tag将输入的激励信号移频50MHz，即信号移至13通道，中心频率为2.472GHz，因为13通道相对其他通道干净，可以忽略其他WiFi网络的干扰。
此外，接收机将p - e设置为10−3进行优化，将采样频率设置为5MHz进行I/Q采样。


