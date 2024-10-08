# 数据链路层

## 基本概念

+ 结点：主机、路由器。
+ 链路：网络中两个结点之间的物理通道。
+ 数据链路：网络中两个结点之间的逻辑通道。
+ 帧：链路层的协议数据单元。

### 数据链路层功能

1. 为网络层提供服务：无确认无连接服务、有确认无连接服务、有确认面向连接服务。面向连接就一定会确认。
2. 链路管理：控制对物理传输介值访问。
3. 组帧（定义数据格式）：帧定界、帧同步、透明传输。规定了帧的数据部分的长度上限——最大传送单元（$MTU$）。
4. 流量控制：发送方的流量。
5. 差错控制。

## 组帧

在一段数据的前后部分添加首部和尾部，构成一个帧来定界传输（由于数据链路层以上的传输单元不用直接传输所以只用添加本层首部就可以了）。接受端在收到物理层上交的比特流后就可以根据首部和尾部的标记从而识别帧的开始和结束。

不管所传数据为什么样的比特组合都能在链路上传输。因此链路层就看不见有什么方案数据传输的东西（控制信息）就是透明传输。

那么如何组帧呢？

### 字符计数法

帧首部使用一个计数字段（第一个字节，八位）表明帧内字符数。题目计算时这个计数字段也要算成一个字符。

![字符计数][charcount]

缺点：如果在某一个帧内，标记位后面的某个字节的数据丢失，那么会影响后面所有的帧。

比如$3\,1 \,1$和$4\,2\,2\,2$，如果前面的帧丢失变成$3\,1$，那么后面的4就会被补到前面变成$3\,1\,4$导致错误。

### 字符填充法

帧就是加头加尾分别标记开始结束，而如果数据内某段比特流数据正好与标记字段重复，就会导致误判断的情况。

字符填充法就是在误会的字符前添加转义字符。

$SOH$表示帧开始，$EOT$表示帧结束。

![字符填充][charfill]

### 零比特填充法

当帧开始符和结束符一样，且帧定界符中含有$5$个以上连续的$1$，就可以使用零填充法。

+ 在发送端扫描整个信息字段（不包括首尾定界符），只要有连续的五个$1$，就立即在第五个$1$后加上一个$0$，无论后面是$1$还是$0$。
+ 在接收端收到一个帧时，先找到标志字段确定边界，再用硬件对比特流进行扫描，发现连续五个$1$时就把后面的$0$删除。

### 违规编码法

使用帧中不会用到的编码来定帧的起始和终止。

![违规编码法][errorfill]

由于字节计数法的脆弱性与字符填充实现的复杂性与不兼容性，所以基本上使用零比特填充与违规编码法。

## 差错控制

噪声来源：

+ 全局噪声（热噪声，随机噪声）：产生随机差错。线路本身电气特征所产生的固有；可以通过改善传感器调高信噪比来减少。
+ 局部噪声（冲击噪声）：产生突发差错。外界特定短暂的原因所产生，是产生差错的主要原因；可以通过编码计数来解决。

差错：

+ 位错：比特位出错，$1$变为$0$，$0$变为$1$。也称为比特差错。
+ 帧错：有三种情况，假如发送的是$1\,2\,3$：
  1. 丢失：$1\,2$：计时器超时重发。
  2. 重复：$1\,2\,2\,3$：帧编号机制。
  3. 失序：$3\,1\,2$：帧编号机制。

为了避免位错进行校验：

+ 检错编码：
  + 奇偶校验法。
  + 循环冗余码（$CRC$）。
+ 纠错编码：海明码。

物理层所说的编码是针对单个比特的调制，为了达成同步。

而数据链路层所说的编码是针对一组比特，通过冗余码的计数实现对一串二进制比特率的检查。

### 奇偶校验码

首先一串数据长度为$n-1$，需要在数据第一位加上一位校验码。

奇校验要选校验码$1$或$0$使得$n$位的总数据里的$1$为奇数，而偶校验要选校验码$1$或$0$使得$n$位的总数据里的$1$为偶数。

奇偶校验码对于出错的数据仍有对应的奇偶个$1$则无法检验出是否出错。即出错奇数个位数可以检查出来，偶数个位数不能检查出来，所以其检错能力为$50\%$。

如字符为$1100\,101$原数据有$4$位$1$，若使用奇校验，则需要首位校验位为$1$让整个数据有奇数$5$个$1$，从而整个数据变为$1110\,0101$。若接受方收到$1100\,0011$、$1100\,1010$这种数据则能校验，因为数据有偶数个$1$，$1101\,0011$则无法校验，因为有奇数个$1$。

### CRC循环冗余码

分为三个部分：传输数据、生成多项式、$FCS$帧检验序列/冗余码。

如简单来说在接受端：

|传输数据|  |生成多项式|  |冗余码|
|:-----:|:-:|:-----:|:-:|:--:|
|5|÷|2|=2|1|

所以最终发送的数据就是传输数据＋冗余码：$5+1=6$。

在接收端：

|接受数据|  |生成多项式|  |冗余码|
|:-----:|:-:|:-----:|:-:|:--:|
|6|÷|2|=3|0|

余数为$0$，判断无错，就接受。

其中正式的计算方法如下，并给出例题：发送数据为$1101\,0110\,11$，采用$CRC$校验，生成多项式为$10011$，其最终发送数据是？

#### 原数据加0

假设生成多项式$G(x)$的阶为$r$，则需要在原数据后面加$r$个$0$（多项式有$N$位，那么阶就是$N-1$位）。（**就是除数P(X) = 10011 化成生成多项式**）

如题目中生成多项式$10011$表示为多项式是$x^4+x^1+x^0$，所以阶就是$4$。

最后数据就是$1101\,0110\,1100\,00$。**（这个原数据就是用来除的）**

#### 模二除法

在数据加$0$后使用模二除法除以多项式，余数就是冗余码$CRC$校验码的比特序列。

模二除法就是异或，同$0$，异$1$，无需进位或借位或者不够减的问题。

![模二除法][CRC]

所以就得到了帧检验序列$FCS$为$1110$。

所以要发送的数据就是要发送的数据加上帧检验序列：$1101\,0110\,1111\,10$。

#### 检错过程

把接收到的每一个帧都除以相同的生成多项式，然后检查得到除数$R$，如果$R=0$，则接受，否则丢弃。

$FCS$的生成与接收端$CRC$检验都是由硬件完成，所以处理很快，因此不会延误数据的传输。

$CRC$具有纠错能力，但是数据链路层只使用了其检错能力，错误就直接丢弃。

### 海明码

能发现双比特错，但是只能纠正单比特错。

#### 确定校验码位数r

海明不等式：$2^r\geqslant k+r+1$，其中$r$为冗余信息位，$k$位信息位。（添加的信息位能表达所有数据状态）

假如传输的数据$D=1011 01$为$6$位，即信息位$k$为$6$，就可以一个个代入得到最小的$r$为$4$，所以海明码的校验码为$4$位，传输数据为$6+4=10$位。

#### 确定校验码和数据的位置

假设4位校验码分别为$P_1$、$P_2$、$P_3$、$P_4$，数据从左到右为$D_1$、$D_2\cdots D_6$。

校验码$P_n$需要插入到原数据之中作为传输数据，且只能插入到$2^n$的位置，即$1,2,4\cdots$。

然后在其他位置按需填入$D_n$：

数据位|1|2|3|4|5|6|7|8|9|10
:---:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:
代码|$P_1$|$P_2$|$D_1$|$P_3$|$D_2$|$D_3$|$D_4$|$P_4$|$D_5$|$D_6$
实际值| | | 1 | |0|1|1| |0|1

#### 求出校验码的值

需要将数据位置的十进制先用二进制表示。

然后根据二进制中校验码$P_n$位置的$1$来判断其可以校验哪些位，因为第一步校验位的插入规定，$P_n$的位置使用二进制表示一定只有一个$1$，那么就要根据这个$1$所在的位置找到所有同样这个位置是$1$的位置所在的数据位，就是可以被校验码校验的原数据位。同一位可以被不同的校验码同时校验。

$P_1$是第一位，即$0001$位，所以$1$在二进制位置的第四个地方，找到$10$个位置中所有二进制位置的第四个位为$1$的数据位，即$0011$（$3$）：$D_1$；$0101$（$5$）：$D_2$；$0111$（$7$）：$D_4$；$1001$（$9$）：$D_5$。所以$P_1$可以校验这$4$个数据。

要使其能校验，就要令所有要校验的位异或等于$0$，或直接等于所有校验数据位异或值。

即令$P_1\oplus D_1\oplus D_2\oplus D_4\oplus D_5=0$。从而$P_1\oplus1\oplus0\oplus1\oplus0=0$，根据同$0$异$1$得到$P_1=0$。

同理$P_2$的位置用二进制表示是$0010$，所以它可以校验二进制位置第三位是$1$的所有数据位，即可以校验$D_1$、$D_3$、$D_4$、$D_6$。

即令$P_2\oplus D_1\oplus D_3\oplus D_4\oplus D_6=0$。从而$P_1\oplus1\oplus1\oplus1\oplus1=0$，根据同$0$异$1$得到$P_2=0$。

同理$P_3$的位置用二进制表示是$0100$，所以它可以校验二进制位置第二位是$1$的所有数据位，即可以校验$D_2$、$D_3$、$D_4$。

即令$P_3\oplus D_2\oplus D_3\oplus D_4=0$。从而$P_1\oplus0\oplus1\oplus1=0$，根据同$0$异$1$得到$P_3=0$。

同理$P_4$的位置用二进制表示是$1000$，所以它可以校验二进制位置第一位是$1$的所有数据位，即可以校验$D_5$、$D_6$。

即令$P_4\oplus D_5\oplus D_6=0$。从而$P_4\oplus0\oplus1=0$，根据同$0$异$1$得到$P_4=1$。

数据位|1|2|3|4|5|6|7|8|9|10
:---:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:
二进制|0001|0010|0011|0100|0101|0110|0111|1000|1001|1010
代码|$P_1$|$P_2$|$D_1$|$P_3$|$D_2$|$D_3$|$D_4$|$P_4$|$D_5$|$D_6$
实际值|0|0|1|0|0|1|1|1|0|1

从而最后海明码就是$0010\quad0111\quad01$。

如果海明码需要一个最高的全校验位，则需要将其对全部数据位和其他全部校验位进行异或为$0$得到。

#### 检错并纠错

海明码“纠错”$d$位，需要码距为$2d+1$的编码方案；“检错”$d$位，则只需码距为$d+1$。

已知海明码就是$0010\,0111\,01$。假如第五位出错，从而变为$0010\,1111\,01$。

令所有要校验的位进行异或运算：

$P_1\oplus D_1\oplus D_2\oplus D_4\oplus D_5=0\oplus1\oplus1\oplus1\oplus0=1$。

$P_2\oplus D_1\oplus D_3\oplus D_4\oplus D_6=0\oplus1\oplus1\oplus1\oplus1=0$。

$P_3\oplus D_2\oplus D_3\oplus D_4=0\oplus1\oplus1\oplus1=1$。

$P_4\oplus D_5\oplus D_6=1\oplus0\oplus1=0$。

从$P_1$进行的运算结果开始，将所有计算结果组成为$1010$，这个数字即表示出错的位置，即第五位出错。

## 流量控制

较高的发送速度和较低的接受能力不匹配时传输会出错。主要是控制发送方的流量。

数据链路层流量控制与传输层的流量控制的区别：

+ 数据链路层流量控制是点到点的，而传输层的流量控制是端到端的。
+ 数据链路层流量控制是相邻两个结点的，而传输层的流量控制是两个主机的。
+ 数据链路层流量控制是收不下就不回复确认（确认控制帧），而传输层的流量控制是接收端发送给发送端一个窗口公告。

流量控制由自动重传请求$ARQ$来实现。

$ARQ$流量控制协议分为停止等待协议与滑动窗口协议，滑动窗口协议又分为两种后退$N$帧协议与选择重传协议。其实停止等待协议也是一种特殊的滑动窗口协议。

后两种协议是滑动窗口技术和请求重复技术的结合，所以称为连续$ARQ$协议。

滑动窗口可以解决流量控制和可靠传输两个部分的功能。

数据链路层协议的滑动窗口在同一次发送与接受中都是固定的。

窗口大小|停止等待协议|后退N帧协议|选择重传协议
:--------:|:--------:|:-------:|:--------:
发送|=1|>1|>1
接受|=1|=1|>1

### 停止等待协议（Stop-and-Wait）

每发送完一个帧就停止发送并等待对方确认，如果收到确认就再发送下一个帧。因为是停止等待，所以只用一位以$1$、$0$为值来对帧编号就可以了。虽然同样是标号为$0$的帧，但是是不同的帧。

#### 无差错情况

$ACK$表示确认$acknowledge$。

![无差错停止等待协议][SW1]

#### 有差错情况

帧丢失或帧出错：

![有差错停止等待协议帧丢失出错情况][SW21]

1. 发送完一个帧后，必须保存其副本，传输完毕才能丢失该副本。
2. 数据帧与确认帧必须编号。
3. 超时计时器：每次发送一个帧就启动一个计时器。
4. 超时计时器设置的重传事件应该比帧传输的平均RTT更长。

ACK确认丢失：

![有差错停止等待协议帧确认丢失情况][SW22]

ACK确认迟到：

![有差错停止等待协议帧确认迟到情况][SW23]

信道利用率=数据帧长度÷(数据帧长度+往返时延$RTT$+确认帧长度)。

信道利用率=(单位时间内发送数据的比特数÷发送发发送数据传输率)÷发送周期。

当确认帧忽略时可以用下面的公式。

信道吞吐率=信道利用率×发送方的发送速率。

#### 停止等待协议特点

+ 简单。
+ 信道利用率低。

### 后退N帧协议（GBN）

+ 发送窗口：发送方维持一组连续的允许发送的帧的序号。
+ 接受窗口：接受方维持一组连续的允许接受的帧的序号。

$GBN$会一次性将在发送窗口内的$n$个帧一个个全部发送完，然后再移动受确认的帧的个数个窗口，如果一直收不到确认信息则一直不移动发送窗口并不断依次重传。

使用累计确认模式，如果接发送方收到当前$n$号帧的确认消息，则前面的所有帧都已经确认接收到。

#### 帧类型

假设发送窗口一次一共有$N$个帧，且帧开始的索引值为$0$，帧的类型分为：

1. 发完被确认的帧$h$：发送窗口已经发送过的且已经被接收端发送确认消息且被接受的帧个数。此时从$0$到$h-1$都是已经发送且被确认的帧，发送窗口的开始索引为$h$。已经移动发送窗口$h$次。
2. 已经发送但仍等待确认的帧$n$：在发送窗口中已经被发送但是因为确认信息在路上所以等待确认的，此时$h$到$h+n-1$都是已经发送但仍等待确认的帧。
3. 还能发送的帧：即在发送窗口中还没有被发送的帧。还能发送的帧个数为$N-n$个。开始的索引为$h+n$，结束的索引为$h+N-1$。
4. 还不能发送的帧：在发送窗口后面的不能被发送的帧，开始的索引为$h+N$。

#### GBN发送方必须响应的事件

1. 上层调用：上层发送数据，发送方必须检查发送窗口是否已满，未满则产生帧并发送，若已满则先缓存数据，等窗口不满时再发送。
2. 收到一个$ACK$：对$n$号帧采取累积确认，偶尔捎带确认（在计算机通信中，当一个数据帧到达的时候，接收方并不是立即发送一个单独的控制帧，而是抑制一下自己并且开始等待，直到网络层传递给它下一个分组。然后，确认信息被附在往外发送的数据帧上——帧头中的$ACK$域。实际上，确认报文搭了下一个外发数据帧的便车。这种“将确认暂时延迟以便可以钩到下一个外发数据帧”的技术称为捎带确认）的方式，即标明接收方已经收到$n$号帧和其之前的全部帧。
3. 超时事件：如果出现超时，则发送方重传所有已发送但是未被确认的帧，即超时以及**后面**的帧。接收方、发送方只按**顺序**接受帧，对于其非期待（**乱序**）的确认帧则丢弃。

#### GBN接收方要做的事件

1. 如果正确收到$n$号帧且顺序一致，则接收方为$n$帧发送一个$ACK$，并将该帧中的数据部分上交上层。
2. 其余情况都丢弃帧，并为最近按序接收的帧重新发送$ACK$，向发送方要重传帧。接收方无需缓存任何失序帧，只用维护下一个按序接收的帧序号。

#### GBN运行

假如发送窗口尺寸为$4$：

![GBN][GBN]

#### GBN窗口长度

采用$n$个比特对帧编号，则接收窗口的尺寸$W_T$应满足$1\leqslant W_T\leqslant 2^n-1$。

如果帧太大会让接收方无法区分新帧旧帧。为什么只到$2^n-1$？因为假设最坏情况下这轮从$0$开始的所有的帧都未收到确认，由于$GBN$使用累计确认，则最大编号会空留一个编号只到$2^n-2$，计算机就明白这里前面的还没有收到不会继续发送了，如果是全部用完编号会到$2^n-1$，如果此时接收方收到$0$号确认，会认为前面的全部收到这是第二轮的编号从而跳过了这轮的确认环节。

#### GBN协议特点

+ 因连续发送数据帧而提高了信道利用率。
+ 在重传时必须把原来已经正确传送的数据帧重传，从而令传送速率降低。

### 选择重传协议（SR）

累计确认会导致批量重传问题，所以$SR$为了实现只重传出错的帧，解决的办法就是设置单个确认而非累计确认，同时加大接收窗口，缓存乱序到达的帧。

发送窗口中分为已经发送被确认的帧、已经发送但等待确认的帧、还能发送的帧三种，其中这三种帧不一定是连续的，而只有发送窗口的第一个帧是已经发送被确认的帧发送窗口才能移动一格或多格。

接收窗口分为希望收到但是没有收到的帧、希望收到且已收到的帧、等待接收的帧三种，其中这三种帧不一定是连续的，接收窗口中收到且确认的帧位于缓存之中，而只有接收窗口的第一个帧是希望收到且已收到的帧接收窗口才能移动一格或多格。

#### SR发送方必须响应的事件

1. 上层调用：上层发送数据，发送方必须检查发送窗口是否已满，未满则产生帧并发送，若已满则先缓存数据，等窗口不满时再发送。
2. 收到一个$ACK$：如果收到$ACK$对应的帧序号在窗口内，则$SR$发送方将那个被确认的帧标记为已接收。如果该帧序号是窗口的下界（最左边），在窗口向前移动到具有最小序号的未确认帧处。如果窗口移动且有序号在窗口内未发送的帧，则发送这些帧。
3. 超时事件：每一个帧都具有自己的计时器，一个超时事件发生后只重传一个帧。

#### SR接收方要做的事件

1. 确认一个正确接收的帧而不管是否按序。失序的帧将被缓存，并返回发送方一个该帧的确认帧，直到所有比其序号更小的帧都被接收为止，这时才能将这一批帧交付上层。
2. 然后向前移动接收窗口。
3. 如果收到了接收窗口前的一个接收窗口长度以内的帧，就返回一个$ACK$表明发送方超时重传的帧已经得到了确认。
4. 如果是其他情况就忽略该帧。

#### SR运行

假如发送窗口、接收窗口尺寸为$4$：

![SR][SR]

#### SR窗口长度

发送窗口最好等于接收窗口，大于会溢出，小于意义不大。

采用$n$个比特对帧编号，则窗口的尺寸$W_T$应满足$W_T=2^{n-1}$。（即编号数量的一半）

如下面以$0$到$3$编号，表示使用两位编号，而滑动窗口大小为$3$。

![SR窗口长度][SRwidth]

所以接收方就不知道发送的$0$号帧是新帧还是旧帧。此时滑动窗口应该大小为$2$。

#### SR协议特点

+ 对数据帧逐一确认，收到一个确认一个。
+ 只用重传出错帧。
+ 接收方有缓存。
+ 发送窗口与接收窗口相等。
+ 全部收到后一起上传。

## 介质访问控制

+ 点对点链路：两个相邻结点通过一个链路链接，无第三者，常用于广域网。如$PPP$协议。
+ 广播式链路：所有主机共享通信介质，常用于局域网。典型拓扑结构：总线型与星型。

介质访问控制就是采取措施使得两对结点之间的通信不会互相干扰。分为：

+ 静态划分信道：信道划分介质访问控制$MAC$
  + 频分多路复用$FDM$。
  + 时分多路复用$TDM$。
  + 波分多路复用$WDM$。
  + 码分多路复用$CDM$。
+ 动态划分信道：
  + 随机访问介质访问控制（所有用户可随机发送信息，可占用全部带宽）：
    + $ALOHA$协议。
    + $CSMA$协议。
    + $CSMA/CD$协议。
    + $CSMA/CA$协议。
  + 轮询访问介质访问控制：
    + 轮询协议。
    + 令牌传递协议。

### 信道划分介质访问控制

使用一条共享信道，但是通过多路复用技术（多格信号组合在一条物理信道上传输使得多个终端设备共享信道资源并提高信道利用率的技术）组合进行传输，提高了信道的利用率。

#### FDM

用户在分配到一定的频带后，在通信过程中自始至终都占用这个频带。频分复用的所有用户在同样时间占用不同的频率带宽资源。

+ 充分利用传输介质带宽，系统效率较高。
+ 技术较成熟，实现容易。

$FDM$使用较少，而是使用$TDM$较多，这是因$TDM$抗干扰能力强，能逐级再生整形避免干扰累计，且数字信号易于自动转换，所以$FDM$用于传输模拟信号，$TDM$用于传输数字信号。

#### TDM

将时间划分为一段段等长的时分复用帧（$TDM$帧：在物理层传输的比特流所划分的帧，表明一个周期）。每一个时分复用的用户在每一个$TDM$帧中占用固定序号的时隙，所有用户轮流占用信道。

+ 当用户使用率较低时会导致信道的利用率很低
+ 用户的等待时间长。

分为同步时分多路复用和异步时分多路复用（统计时分复用）.

统计时分复用$STDM$：

+ 添加了一个集中器，将不同用户分散的数据集中在一起，单位时间的数据组成一个$STDM$帧，再发送。
+ 每一个$STDM$帧中的时隙数小于连接在集中器上的用户数。
+ 每个用户有数据就随时发送给集中器的输缓存，然后集中器按顺序依次扫描输入缓存，把缓存中的输入数据放入$STDM$帧中，一个$STDM$帧满了就发出。
+ $STDM$帧并不是固定分配时隙，而是按需动态分配时隙。

#### WDM

就是光的频分多路复用，根据同一根光纤中传输多种不断波长（频率）的光信号，根据不同的波长做用波长分解复用器分解出来。

#### CDM

码分复用既共享了空间也共享了时间。

码分多址（$CDMA$）是码分复用的一种方式。

+ 一个比特分为多个码片/芯片，每一个站点被指定一个唯一的$m$位的码片序列。
+ 发送$1$时站点发送码片序列，发送$0$时发送码片序列反码（一般$0$写作$-1$）。
+ 如何复用：多个站点同时发送数据的时候，要求各个站点码片序列相互正交。
+ 如何合并：各路数据在信道中按位线性相加。
+ 如何分离：合并的数据和源站码片序列规格化内积。

假如站点$A$的码片序列被指派为$0001\,1011$，则$A$站发送$0001\,1011$就表示发送比特$1$，发送$1110\,0100$就表示发送比特$0$。

为了方便，按惯例将码片中的$0$写为$-1$，将$1$写为$+1$，因此$A$站的码片序列是$-1-1-1+1+1-1+1+1$。

令向量$\vec{S}$表示$A$站的码片向量，令$\vec{T}$表示$B$站的码片向量。两个不同站的码片序列正交，即向量$\vec{S}$和$\vec{T}$的规格化内积为$0$：$\vec{S}\cdot-\vec{T}=\dfrac{1}{m}\sum\limits_{i=1}^mS_i\overline{T_i}=0$。

任何一个码片向量和该码片向量自身的规格化内积都是$1$，任何一个码片向量和该码片反码的向量的规格化内积是$-1$。即自己×自己$=1$ ，自己×别人$=0$ ，自己×反码$=-1$。

令向量$T$为$(-1-1+1-1+1+1+1-1)$，即$0010\,1110$。

当$A$站向$C$站发送数据$1$时，就发送了向量$(-1-1-1+1+1-1+1+1)$。当$B$站向$C$站发送数据$0$时，就发送了向量$(+1+1-1+1-1-1-1+1)$。两个向量到了公共信道上就进行叠加，实际上就是线性相加，得到$\vec{S}-\vec{T}=(0\,0-2+2\,0-2\,0+2)$。

到达$C$站后，进行数据分离，如果要得到来自$A$站的数据，$C$站就必须知道$A$站的码片序列，让$\vec{S}$与$\vec{S}–\vec{T}$进行规格化内积。根据叠加原理，其他站点的信号都在内积的结果中被过滤掉了，内积的相关项都是$0$，而只剩下$A$站发送的信号。得到$\vec{S}\cdot(\vec{S}–\vec{T})=1$，所以$A$站发出的数据是$1$。同理，如果要得到来自$B$站的数据，那么$\vec{T}\cdot(\vec{S}–\vec{T})=-1$，因此从$B$站发送过来的信号向量是一个反码向量，代表$0$。

+ 频谱利用率高。
+ 抗干扰能力强。
+ 保密性强。
+ 语音质量好。
+ 减少投资和降低运行成本。
+ 主要用于无线通信系统，特别是移动通信系统。

### ALOHA协议

#### 纯ALOHA协议

不监听信道，不按时间槽发送，随机重发（想发就发）。

![ALOHA][ALOHA]

+ 检测冲突：如果发送冲突，接收方就会检测到差错然后不予确认，发送方在一定时间内收不到确认就判断冲突。
+ 解决冲突：超时后等待随机时间重传。

假设网络负载（$T_0$时间内所有站点发送成功的和未成功而重传的帧数）为$G$，则纯$ALOHA$网络的吞吐量利用率（$T_0$时间内成功发送的平均帧数）为$S=Ge^{-2G}$。当$G=0.5$时极大，$S=0.5e^{-1}\approx0.184$。

#### 时隙ALOHA协议

把时间划分为若干个相同的时间片（时间槽），所有用户只能在时间片的开始时刻同步接入网络信道，若发生冲突则必须等到下一个时间片开始的时刻再发送。

假设网络负载（$T_0$时间内所有站点发送成功的和未成功而重传的帧数）为$G$，则间隙$ALOHA$网络的吞吐量利用率（$T_0$时间内成功发送的平均帧数）为$S=Ge^{-G}$。当$G=1$时极大，$S=e^{-1}\approx0.368$。

![时隙ALOHA][SA]

### CSMA协议

载波监听多路访问协议。发送前监听信道。当信道空闲发送帧，当信道忙推迟发送。

+ $CS$载波监听：表示每一个站在发送数据之前都要检查一下总线上是否有其他计算机在发送数据。当几个站同时在总线上发送数据时，总线上的信号电压摆动值就增大，因为互相叠加。当一个站检测到的信号电压摆动值超过一定门限值时，就认为总线上至少两个站在同时发送数据，表明产生了碰撞。
+ $MA$多点接入：表示许多计算机以多点接入的方式连接在一根总线上。

#### 1-坚持CSMA

+ 发送信息前监听信道。
+ 当信道空闲时会直接传输，不必等待。
+ 当信道忙时会一直监听，直到空闲立刻传输。
+ 如果冲突（一段时间内未收到确认），则等待一个随机长的时间再监听。

+ 优点：只要媒体空闲站点就立马发送，避免媒体使用率的损失。
+ 缺点：若有两个或以上的站点有数据要发送，冲突就无法避免。

#### 非坚持CSMA

+ 发送信息前监听信道。
+ 当信道空闲时会直接传输，不必等待。
+ 当信道忙时会等待一个随机时间之后再进行监听。

+ 优点：采用随机的重发延迟时间可以减少冲突发生的可能性。
+ 缺点：可能存在所有站点都在延迟等待中，使得媒体空闲，降低媒体使用率。

#### p-坚持CSMA

+ 发送信息前监听信道。
+ 当信道空闲时以$p$概率直接传输，不必等待，概率$1-p$等待到下一个时间槽再侦听，以同样的概率侦听或传输。
+ 当信道忙时会等待一个随机时间之后再进行监听。

+ 优点：既能像非坚持算法减少冲突，又能像$1$-坚持算法那样减少媒体空闲时间。
+ 缺点：无法在冲突时就发现，所以发生冲突时还是坚持发送完数据帧，从而造成浪费。

信道状态|1-坚持|非坚持|p-坚持
:------:|:----:|:----:|:----:
空闲|立即发送数据|立即发送数据|以概率p发送数据，以概率1-p推迟到下一个时隙
忙|继续坚持侦听|放弃侦听，等待一个随机的时间后再侦听|持续侦听，直至信道空闲

### CSMA/CD协议

载波监听多点接入/碰撞检测协议。是$CSMA$协议的完善，不仅先监听再发送，还可以边监听边发送，当发现碰撞就立刻停止发送，就避免了$CSMA$协议的无法在冲突时就发现，发生冲突时还是坚持发送完数据帧造成浪费的缺点。

所以说明这个协议是适用于总线型或半双工网络上。

站点最迟总线**最远**一个端到另一个端的**往返**传播时延（**争用期/冲突窗口/碰撞窗口**）才会知道自己发送的数据没有和别人碰撞（即撞到目标站点前了），，所以只要该站在两倍的总线端到端的传播时延时间内没有检测到碰撞，那么就可以肯定本次发送没有碰撞。

#### 确定重传时机

采用截断二进制指数规避算法。

1. 确定基本退避（推迟）时间为争用期$2\tau$。
2. 定义参数$k$，其表示重传次数，但是$k$不超过$10$。当重传次数不超过$10$时，$k$等于重传次数，当重传次数大于$10$时，$k$就一直等于$10$。
3. 从离散的整数集合${0,1,2,3,4\cdots,2^k-1}$之中随机取出一个数$r$，重传所需要的退避的时间就是$r$倍的基本退避时间，即$2r\tau$。
4. 当重传达$16$次仍不能成功时，就说明网络太拥挤，认为该帧永远无法正确发出，就抛弃该帧并向高层报错。

令重传时间为$t$。

+ 第一次重传，$k=1$，$r\in\{0,1\}$，$t\in\{0,2\tau\}$。
+ 第二次重传，$k=2$，$r\in\{0,1,2,3\}$，$t\in\{0,2\tau,4\tau,6\tau\}$。
+ 第三次重传，$k=3$，$r\in\{0,1,2,3,4,5,6,7\}\cdots$

#### 最小帧长

如果帧太短，那么如果发生碰撞就很容易发送完才检测到发生碰撞而无法停止发送。所以就需要规定一个最小的帧长。

最短帧长等于争用期时间内发出的比特数。

所以帧的传输时延至少要两倍于信号在总线中的传播时延。

即最小帧长=总线单向传播时延×数据传输速率×$2$=总线长度÷传播速率×数据传输速率×$2$=争用期×数据传输速率。

<span style="color:orange">注意：</span>以太网规定争用期为$51.2\mu s$，最短帧长为$64B=512bit$，凡是小于则被判定为无效帧。

### CSMA/CA协议

载波监听多点接入/碰撞避免协议。对比$CSMA/CD$协议只能检测碰撞，而$CSMA/CA$协议可以避免碰撞。且$CSMA/CD$协议只能用于总线类型的以太网，而$CSMA/CA$可以用于无线局域网。

+ 发送信息前监听信道。
+ 当信道空闲时发送端发出$RTS$（$requests\,to\,send$），$RTS$包括发射端的地址、接收端的地址、下一份数据将持续发送的时间等信息。
+ 当信道忙时会等待一个随机时间之后再进行监听。
+ 接收端接收到$RTS$后，会响应$CTS$（$clear\,to\,send$）。
+ 发送端收到$CTS$后，开始发送数据帧，同时预约信道，发送方告知其他站点要传输多久的数据。
+ 接收端收到数据帧后，将用$CRC$来检验数据是否正确，正确则响应$ACK$帧。
+ 发送方收到$ACK$帧后才可以进行下一个数据帧的发送，若没有则一直重传一直到规定重发次数为止（使用二进制指数退避算法确定推迟时间）。

为了尽量避免碰撞，$802.11$规定，所有的站完成发送后，必须再等待一段很短的时间（继续监听）才能发送下一帧。这段时间称为帧间间隔（$InterFrame\,Space$，$IFS$）。帧间间隔的长短取决于该站要发送的帧的类型。$802.11$使用了$3$种$IFS$：

1. $SIFS$（短$IFS$）：最短的$IFS$，用来分隔属于一次对话的各帧，使用$SIFS$的帧类型有$ACK$帧、$CTS$帧、分片后的数据帧，以及所有回答$AP$探询的帧等。
2. $PIFS$（点协调$IFS$）：中等长度的$IFS$，在$PCF$操作中使用。
3. $DIFS$（分布式协调$IFS$：最长的$IFS$，用于异步帧竞争访问的时延。

$CSMA/CA$协议与$CSMA/CD$协议的相同点：

1. 都是需要先侦听信道。
2. 冲突后都会进行有限的重传。

$CSMA/CA$协议与$CSMA/CD$协议的不同点：

1. 传输介质不同：$CSMA/CA$协议无线传输；$CSMA/CD$协议有线则总线型传输。
2. 载波检测方式不同：$CSMA/CA$协议采用能量检测（$ED$）、载波检测（$CS$）和能量载波混合检测三种方式；$CSMA/CD$协议采用电缆电压检测。
3. $CSMA/CA$协议避免冲突；$CSMA/CD$协议检测冲突。

### 轮询访问介质访问控制

信道划分介质访问控制协议在网络负载重时信道效率高且公平，而在负载轻时则效率低；随机访问控制协议在网络负载重时会产生冲突开销，而负载轻时效率会比较高，单个结点可以利用信道全部带宽。

也称为轮流协议。即减少产生冲突，又尽量发送时占用全部带宽。

#### 轮询协议

主结点会轮流以发送数据帧的形式询问从属结点是否发送数据，没有被询问的从属结点无法发送数据。

问题：

1. 轮询开销。
2. 等待延迟。
3. 单点故障。

#### 令牌传递协议

一般使用令牌环网实现，逻辑上是环型的，但是物理上是星型的。

![令牌传递协议][TP]

$TCP$用于转发令牌。而令牌是一个特殊格式的$MAC$控制帧，不包含任何信息，在令牌环上循环，控制信道使用，只有有令牌才能发送数据，确保同一个时刻只有一个结点独占信道。

每一个结点都可以在**令牌持有时间**内获得发送数据的权利，而不能无限制的持有令牌，超过时间无论是否发送完成都要归还令牌。

问题：

1. 令牌开销。
2. 等待延迟。
3. 单点故障。

采用令牌传输方式的网络基本上是负载较重，通信量较大的网络。

## 局域网

$LAN$使用广播信道。

### 局域网特性

特点：

1. 覆盖范围较小。
2. 采用专门的传输介质（双绞线、同轴电缆）进行联网，数据传输速率较高。
3. 通信延迟时间端，误码率低，可靠性高。
4. 各站平等共享信道。
5. 多采用分布式控制与广播式通信，可以广播与组播。

#### 拓扑结构

+ 星型：中心结点时控制中心，任意两个结点之间的通信最多只用两步，传输速度快，且网络结构简单，建网容易，便于控制与管理，但是可靠性低，网络共享能力差，会单点故障。
+ 总线型：网络可靠性高，网络结点之间响应速度快，共享资源能力强，设备投入量少，成本低，安装使用方便，当某结点故障时对整个网络系统影响小。但是总线损坏也会造成巨大影响。
+ 环型：通信设备和线路比较节省，有单点故障问题，由于线路封闭，不易于拓展，系统响应延时长，且信息传输效率较低。
+ 树型：易于拓展，易于隔离故障，也容易单点故障。

#### 局域网介质访问控制

1. $CSMA/CD$：常用于总线型局域网，也用于树型网络。
2. 令牌总线：常用于总线型局域网，也用于树型网络。把总线或树型网络中的各个工作站按一定的顺序如按接口地址大小排列形成一个逻辑环，只有令牌持有者才能控制总线发送信息。
3. 令牌环：用于环型局域网，如令牌环网。

#### 局域网的分类

1. 以太网：是应用最为广泛的局域网。包括标准以太网（$10Mbps$）、快速以太网（$100Mbps$）、千兆以太网（$1000Mbps$）和$10G$以太网，都符合$IEEE802.3$系列标准规范。逻辑拓扑总线型，物理拓扑是星型或拓展星型，使用$CSMA/CD$。
2. 令牌环网：物理星型，逻辑环型，基本上已经过时。
3. $FDDI$网：物理双环结构，逻辑环型，使用光纤，造价高，基本上没有使用到。
4. $ATM$网：较新的单元交换技术，用$53$字节固定长度的单元进行交换。
5. 无线局域网：采用$IEEE802.11$标准。

#### IEEE802标准

局域网与城域网技术标准，使用的范围有以太网、令牌环、无线局域网。

+ $IEEE802.3$：$CSMA/CD$与物理层技术规范。
+ $IEEE802.5$：令牌环网$Token-Ring$的介质访问控制协议与物理层技术规范。
+ $IEEE802.8$：$FDDI$的光纤技术规范。
+ $IEEE802.11$：无线局域网$WLAN$的介质范文控制协议与物理层技术规范。

$802.3$局域网简称为以太网。

#### MAC子层与LLC子层

$IEEE802$标准将局域网的数据链路层分为逻辑链路层$LLC$子层与介质访问控制$MAC$子层。

+ $LLC$逻辑链路控制：
  + 建立和释放数据链路层连接、提供与高层接口、差错控制、帧加序号。
  + 为网络层提供服务：无确认无连接、面向连接、带确认无连接、高速传送。
+ $MAC$介质访问控制：
  + 组帧拆帧（根据$LLC$的序号），帧寻址和识别，竞争处理，比特差错控制等。
  + 屏蔽了不同物理链路种类的差异性。

### 以太网

+ $Ethernet$是基带总线局域网规范，使用$CSMA/CD$技术，主要负责物理层与数据链路层的规范。
+ 使用$MAC$协议，其特点是无连接，不可靠，不确认，不对帧编号。
+ 传输介质由粗同轴电缆到细同轴电缆再到双绞线+集线器。
+ 拓扑结构逻辑上总线型，物理上星型。

#### 以太网传输

当以太网发送一个数据，那么以太网将以广播形式发送，以太网上的**所有主机**包括发送端本身都能收到数据。

通过曼彻斯特编码以中间电平跳动作为同步信号，无序其他同步操作。

#### 以太网标准

+ $DIX\,Ethernet\,V2$：第一个局域网产品（以太网）规约。
+ $IEEE802.3$：$IEEE$指定的第一个$IEEE$的以太网标准，帧格式有所不同。

#### 以太网类型

参数|1OBASE5|10BASE2|1OBASE-T|10BASE-FL
:--:|:-----:|:-----:|:------:|:-------:
传输媒体|基带同轴电缆（粗缆）|基带同轴电缆（细缆)|非屏蔽双绞线|光纤对（850nm）
编码|曼彻斯特编码|曼彻斯特编码|曼彻斯特编码|曼彻斯特编码
拓扑结构|总线形|总线形|星形|点对点
最大段长|500m|185m|100m|2000m
最多结点数目|100|30|2|2

对于$10BASE-T$以太网

+ $10$表示传输速率为$10Mb/s$，$BASE$表示传输基带信号，$T$代表使用双绞线。现在一般采用无屏蔽双绞线（$UTP$）。
+ 网络拓扑物理上是星型，逻辑上是总线型，每段双绞线最长为$100m$。
+ 采用曼彻斯特编码。
+ 使用$CSMA/CD$介质访问机制。
+ 超过覆盖范围拓展使用中继器。

当速率大于$100Mb/s$时就可以称为高速以太网（快速以太网）。

快速以太网仍然使用$CSMA/CD$协议，它采用保持最短帧长不变而将最大电缆长度减少到$100m$的方法，使以太网的数据传输速率提高至$100Mb/s$及以上。

1. $100BAST-T$以太网：在双绞线上传输$100Mb/s$基带信号的星型拓扑以太网，仍使用$IEEE802.3$的$CSMA/CD$协议。支持全双工与半双工，可以全双工方式工作下无冲突。所以全双工方式下不使用$CSMA/CD$协议。
2. 吉比特以太网：在光纤（$IEEE 802.3z$）或$4$对$UTP5$双绞线（$IEEE 802.3ab$）上传送$1Gb/s$信号。支持全双工与半双工，可以全双工方式工作下无冲突。所以全双工方式下不使用$CSMA/CD$协议。
3. $10$吉比特以太网：在光纤上传送$10Gb/s$信号。只支持全双工，无冲突，所以不使用$CSMA/CD$协议。

#### 适配器与MAC地址

+ 计算机与外界局域网的连接通过通信适配器完成，过去通过单独网络接口卡即网卡$NIC$实现。
+ 网卡主要工作在物理层和数据链路层。
+ 适配器上与处理器和存储器，包括$RAM$和$ROM$。
+ $ROM$上有计算机硬件地址，称为介质访问控制$MAC$地址。
+ 在局域网中，硬件地址又被称为物理地址或$MAC$地址。
+ $MAC$地址是全球唯一的$48$位二进制适配器地址，即$6$个字节，一般用连字符或冒号分隔的$12$个十六进制数表示。前$24$位代表厂家（$IEEE$规定），后$24$位由厂家指定。

#### 以太网MAC帧

如以太网标准所说一共分为两种标准，最常用的是$V2$的$MAC$格式：

![MAC帧格式][MACformat]

以太网帧附加信息$18B$，规定帧最短为$64B$，所以数据最短为$46B$，规定数据部分最长为$1500B$，所以以太网帧最长为$1518B$。

+ 物理层会插入前导码，用于接收端与发送端时钟同步。包括前同步码（用于快速实现$MAC$帧的比特同步）与帧开始定界符（表示后面的信息就是$MAC$帧）两个部分。
+ 地址即$MAC$地址。
+ 类型表示$IP$数据报的类型，表示数据应该交给哪个协议实体处理。。
+ 数据最小值为$46$是因为$CSMA/CD$算法规定以太网最小帧长为$64$字节，所以数据最小就是$64-6-6-2-4=46$字节，最长为$1500$字节。数据包括高层的协议信息。
+ 填充用于帧长太短来填充到$64$字节，所以长度为$0\sim46$字节。
+ $FCS$：校验范围从目的地址到数据端末尾，采用$32$位循环冗余码$CRC$，但是不用校验$MAC$帧的前导码。
+ 以太网帧没有结束定界符是因为以太网采用曼彻斯特编码，如果与数据必然是电压变化的，而如果电压没有变化就代表已经结束，所以不需要结束定界符。但是$MAC$帧也要加尾部。

与$IEEE802.3$的区别：

1. 第三个字段是长度/类型。
2. 当长度/类型字段小于$0x0600$时，数据字段必须装入$LLC$子层。

### 无线局域网

#### 有固定基础设施

用于有固定基础设施的无线局域网。使用星型拓扑。其中心位接入点$AP$。

无线局域网最小构建是基本服务集$BBS$。一个基本服务集包括一个基站和若干移动站。

$BSSID$（基本服务集$ID$）不超过$32$字节，代表基站的$MAC$地址。

一个基本服务集覆盖的地理范围称为一个基本服务区$BSA$。

一个$BSS$可以独立也可以通过$AP$连接到一个分配系统$DS$然后连接到另一个$BSS$，构建一个扩展服务集$ESS$。$ESS$通过门桥为无线用户提供以太网的接入。

#### 无固定基础设施

称为自组网络，没有$AP$，而是一些平等状态的移动站相互通信构成的临时网络。因此都具有路由器的功能。

#### IEEE802.11的MAC帧头格式

![IEEE802.11的MAC帧头格式][WLANMAC]

对于$WDS$（无线分布式系统）：

+ 地址1：$RA$接收端，接收端基站地址。
+ 地址2：$TA$发送端，发送端基站地址。
+ 地址3：$DA$目的地址，目标主机的$MAC$地址。
+ 地址4：$SA$源地址，发送端的$MAC$地址。

![MAC帧头格式表格][WLANMACtable]

#### 碰撞检测

无线局域网不使用$CSMA/CD$协议，而使用$CSMA/CA$协议，因为发送过程中不需要进行冲突检测：

1. 在无线局域网的适配器上，接收信号的强度往往远小于发送信号的强度，因此若要实现碰撞检测，那么硬件上的花费就会过大。
2. 在无线局域网中，并非所有站点都能听见对方，由此引发了隐蔽站和暴露站问题，而“所有站点都能够听见对方”正是实现$CSMA/CD$协议必备的基础。

### 虚拟局域网

即$VLAN$。基于交换技术按逻辑进行划分局域网一个广播域。可以隔离冲突域，也可以隔离广播域。

+ 有效共享网络资源。
+ 简化网络管理。
+ 提高网络安全性。

$VLAN$可以隔离冲突域也可以隔离广播域。

#### 虚拟局域网实现

通过$802.ac$标准定义。它在以太网帧中插入一个四字节的标识符（插入在源地址字段和类型字段之间)，称为$VLAN$标签，用来指明发送该帧的计算机属于哪个虚拟局域网。插入$VLAN$标签的帧称为$802.1Q$帧。由于首部增加了四字节，因此以太网的最大帧长从原来的$1518$字节变为$1522$字节。

虚拟局域网通过标识符实现逻辑分组和管理，不需要额外的硬件支持。

$VLAN$技术可以将一个物理局域网在逻辑上划分成多个广播域，即划分为多个$VLAN$。$VLAN$技术部署在数据链路层，用于隔离二层流量。同一个$VLAN$内的主机共享同一个广播域，它们之间可以进行二层通信。而$VLAN$间的主机属于不同的广播域，不能直接实现二层互通。这样，广播报文就被限制在各个相应的$VLAN$内，并提高了网络的安全性。本质上虚拟局域网使用的是三层架构的交换技术（否则不能隔离广播域和冲突域）。

#### 虚拟局域网划分

虚拟局域网的划分与物理地址无关，可以处于不同的实际局域网中，本身只是一种技术。划分方式只能基于物理层、数据链路层和网络层（不能基于更上层的数据）：

+ 基于交换机端口。
+ 基于网卡$MAC$地址。
+ 基于网络层$IP$子网地址。
+ 基于协议。
+ 基于策略。

## 广域网

是单一的网络。$WAN$的通信子网主要使用分组交换技术，达到资源共享的目的。

&nbsp;|广域网|局域网|
:----:|:----:|:----:
覆盖范围|很广，通常跨区域|较小，通常在一个区域内
连接方式|结点之间都是点到点连接，但为了提高网络的可靠性，一个结点交换机往往与多个结点交换机相连|普遍采用多点接入技术
OSI参考模型层次|三层：物理层，数据链路层，网络层|两层：物理层，数据链路层
着重点|资源共享|数据传输

广域网不等于互联网，因为互联网可以接入不同类型的网络，可以是局域网也可以是广域网。

### PPP协议

点对点协议是使用最广泛的面向字节的数据链路层协议，用户使用拨号电话接入因特网时一般使用$PPP$协议。

#### PPP协议的特点

+ 简单：对于链路层的帧，无须纠错、序号、流量控制。
+ 只支持全双工链路。
+ 封装成帧：具有帧定界符。
+ 透明传输：面向字符。对于与帧定界符一样的比特组合的数据应如何处理：异步线路使用字节填充（因为按字节或字符传送），同步线路使用比特填充（因为按比特传送）。
+ 多种网络层协议：可以采用多种协议。两端可以连接不同的网络层协议。
+ 多种类型链路：串行/并行，同步/异步。
+ 差错检测：错就丢弃，只检错不纠错。
+ 检测连接状态：链路是否正常工作。
+ 最大传送单元：数据部分最大长度$MTU=1500$。
+ 网络层地址协商：需要知道通信双方的网络层地址。
+ 数据压缩协商：传输数据时需要对数据进行压缩。
+ 无需支持多点线路：只用满足点对点就可以了。
+ 动态分配$IP$地址：因为可用于拨号连接。
+ 身份验证：支持$PAP$和$CHAP$。

&emsp;|PAP|CHAP
:----:|:-:|:--:
安全性|低|高
传输密码|密码明文|密码哈希值
实现方式|两次握手|三次握手
请求方式|被叫方提出，主叫方响应|主叫方提出并发送随机哈希值，被叫方返回一个包含这个哈希值的数据报，主叫方确认后发送一个连接成功的数据报连接

#### PPP协议的组成

1. 一个将$IP$数据报封装到串行链路（同步或异步）的方法。
2. 链路控制协议$LCP$：建立并维护数据链路连接，并进行身份验证。
3. 网络控制协议$NCP$：$PPP$支持多种网络层协议，每个不同的网络层协议都需要一个相应的$NCP$来配置，为网络层建立和配置逻辑连接。

#### PPP协议的工作流程与帧格式

![PPP协议状态图][PPPstatus]

![PPP帧格式][PPPformat]

+ $F$：表示帧定界符，用二进制表示就是$0111\,1110$。当数据内出现帧定界符就需要插入转义字符$7D$：$0111\,101$。
+ $A$：表示$Address$，未完善。
+ $C$：表示$Control$，未完善。
+ 协议：表示信息部分的内容，是$IP$数据报、$LCP$数据或网络层控制数据等。
+ $FCS$：用$CRC$实现的帧检验序列。

### HDLC协议

高级数据链路控制协议是一个在同步网上传输数据、面向比特的数据链路层协议。

由$ISO$根据$SDLC$协议扩展开发，不属于$TCP/IP$族。

#### HDLC协议的特点

+ 可以透明传输。
+ 面向比特。只使用$0$比特填充法。
+ 易于硬件实现。
+ 只支持全双工通信。
+ 所有帧采用$CRC$检验。
+ 对信息帧进行顺序编号，可以防止漏收和重收，传输可靠性高。

#### HDLC协议的站

1. 主站：发送命令（包括数据信息）帧、接收响应帧，并负责对整个链路的控制系统的初启、流程控制、差错检测或恢复。
2. 从站：接收由主站发来的命令帧，向其发送响应帧，并配合主站参与差错恢复等链路控制。
3. 复合站：既能发送，又能接收命令帧和响应帧，并负责整个链路的控制。

#### 数据操作方式

1. 正常响应方式：只有经过主站同意从站才能传输数据。
2. 异步平衡方式：每一个复合站都能自主传输数据。
3. 异步响应方式：无需经过主站同意从站就能传输数据。

#### HDLC协议的帧格式

![HDLC帧格式][HDLCformat]

+ $F$：表示帧定界符，用二进制表示就是$0111\,1110$。当数据内出现帧定界符就需要插入转义字符$7D\,0111\,101$。
+ $A$：表示$Address$，当数据操作方式是正常响应方式或异步响应方式，都是表示从站的地址，而如果是异步平衡方式，则填充应答站的地址。
+ $C$：表示$Control$：
  1. 信息帧（$I$）：第一位是$0$，用于传输数据信息，或使用捎带技术对数据进行确认。
  2. 监督帧（$S$）：$10$，用于流量控制与差错控制，执行对信息帧的确认、请求重发和请求暂停发送等功能。
  3. 无编号帧（$U$）：$11$，用于提供对链路的建立、拆除等多种控制功能。

#### HDLC协议与PPP协议的联系与区别

联系：

1. 都只支持全双工链路。
2. 都可以实现透明传输。
3. 都只检测差错而不纠正差错。

区别：

&nbsp;|PPP协议|HDLC协议
:----:|:-----:|:------:
面向对象|面向字节|面向比特|
是否拥有协议字段|2B协议字段|没有
序号或确认机制|无|有
是否可靠|不可靠|可靠
透明传输技术|同步字符填充异步比特填充|比特填充
实现|软件|几乎硬件

## 数据链路层设备

主要包含网桥与交换机。

### 网桥（Bridge）

多格以太网通过网桥连接后就形成了更大的以太网，原来的每个以太网就是一个网段。

可以互联不同的物理层、不同的$MAC$子层及不同速率的以太网。

网桥工作在数据链路层的$MAC$子层。

当网桥收到一个帧时，不会广播此帧，而是先检测该帧的目的$MAC$地址，然后确定转发到哪一个接口或丢弃（过滤）。

#### 网桥的优点

1. 过滤通信量，增大吞吐量。
2. 扩大了物理范围。
3. 提高了可靠性。
4. 可互联不同物理层、不同$MAC$子层和不同速率的以太网。

#### 网桥的缺点

1. 增加时延。
2. $MAC$子层没有流量控制功能（流量控制需要编号机制，而编号机制在$LLC$子层）。
3. 不同$MAC$子层的网段连接需要帧格式转换。
4. 适用于少数据量的局域网，否则会出现广播风暴。

#### 网桥的种类

+ 透明网桥：指以太网上的站点并不知道所发送的帧将经过哪几个网桥，是一个即插即用的设备。通过自学习的方式来寻找和更新路径。
+ 源路由网桥：在发送帧时，把详细的最佳路由信息（路由最少/时间最短）放在帧的首部中。源站以广播的方式向欲通信的目的站发送多个发现帧，目的站再根据每一个发现帧的路径发送响应帧，从而以枚举的方式获得最优路径。
+ 使用源路由网桥可以利用最佳路由。若在两个以太网之间使用并联的源路由网桥，则还可使通信量较平均地分配给每个网桥。采用透明网桥时，只能使用生成树，而使用生成树一般并不能保证所用的路由是最佳的，也不能在不同的链路中进行负载均衡。
+ 透明网桥和源路由网桥中提到的最佳路由并不是经过路由器最少的路由，而可以是发送帧往返时间最短的路由，这样才能真正地进行负载平衡，因为往返时间长说明中间某个路由器可能超载了，所以不走这条路，换个往返时间短的路走。

### 交换机（Switch）

交换机就是多接口网桥。每一个接口都是一个冲突域。可以独占传输媒体带宽

#### 交换机的种类

直通式交换机：查完目的地址（$6B$）直接转发。延迟小，可靠性低，无法支持具有不同速率的端口的交换。

存储转发式交换机：将帧放入高速缓存，并检查是否正确，正确则转发，错误则丢弃。延迟大，可靠性高，可以支持具有不同速率的端口的交换。

#### 交换机的特点

+ 以太网交换机的每个端口都直接与单台主机相连（普通网桥的端口往往连接到以太网的一个网段)，并且一般都工作在全双工方式。
+ 交换机的总带宽是各端口带宽之和。
+ 以太网交换机能同时连通许多对端口，使每对相互通信的主机都能像独占通信媒体那样，无碰撞地传输数据。支持多对用户同时通信。
+ 以太网交换机也是一种即插即用设备（和透明网桥一样)，其内部的帧的转发表也是通过自学习算法自动地逐渐建立起来的。
+ 以太网交换机由于使用了专用的交换结构芯片，因此交换速率较高。以太网交换机独占传输媒体的带宽。

使用交换机后带宽总容量会变为原来的$N$倍，$N$为端口数。

#### 交换方式

+ 直通式交换机：
  + 只检查帧的**目的地址**（$6B$），这使得帧在接收后几乎能马上被传出去。
  + 这种方式速度快，但缺乏智能性和安全性，也无法支持具有不同速率的端口的交换。
+ 存储转发式交换机：
  + 先将接收到的帧缓存到高速缓存器中，并检查数据是否正确，确认无误后通过查找表转换成输出端口将该帧发送出去。
  + 如果发现帧有错，那么就将其丢弃。
  + 存储转发式的优点是可靠性高，并能支持不同速率端口间的转换。
  + 缺点是延迟较大。

### 交换机与网桥

1. 网桥的端口一般连接局域网，而交换机的端口一般直接与局域网的主机相连。
2. 交换机允许多对计算机同时通信，而网桥仅允许每个网段上的计算机同时通信。
3. 网桥采用存储转发进行转发，而以太网交换机还可以采用直通方式进行转发，且以太网交换机采用了专用的交换结构芯片，转发速度比网桥快。

### 冲突域与广播域

+ 冲突域：在以太网中，如果某个$CSMA/CD$网络上的两台计算机在同时通信时会发生冲突（即同一时间只能一台主机发送信息），那么这个$CSMA/CD$网络就是一个冲突域（$collision\,domain$）。如果以太网中各个网段以集线器连接，因为不能避免冲突，所以它们仍然是一个冲突域。（一个数据链路层设备的接口一个冲突域）
+ 广播域：网络中能接收任一设备发出的广播帧的所有设备的集合。即如果一个站点发出一个广播信号，那么所有能接收到该信号的设备范围就是一个广播域。（一个网络层设备一个广播域）
+ 网段：指一个计算机网络中使用同一物理层设备（传输介质、中继器、集线器等）能直接通信的一部分。

一个网段就是一个冲突域，一个局域网就是一个广播域。

&nbsp;|能否隔离冲突域|能否隔离广播域
:----:|:-----------:|:-----------:
物理层设备|否|否
链路层设备|是|否
网络层设备|是|是

