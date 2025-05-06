目录

1.四次握手的目的

2.四次握手分析

2.1第一次握手：Authenticator->Supplicant

2.2第二次握手：Supplicant->Authenticator

2.3第三次握手：Authenticator->Supplicant

2.4第四次握手：Supplicant->Authenticator

3.名词解释：

 

1.四次握手的目的
通过握手过程协商出PTK和GTK，关于这两个名词的解释间“名词解释”小节。

先说一下PTK的结构如下，它的结构跟加密算法相关，前两段长度是一样的，区别在与第3段。举例说明，当加密算法为TKIP时，TK字段占256位，当加密算法为CCMP时，TK字段为128位。

TKIP加密算法：
![[Pasted image 20240606211516.png]]
 
CCMP加密算法：
![[Pasted image 20240606211530.png]]

KEK和KCK字段是给EAPOL-KEY使用的，即用于四次握手的过程中的加密和完整性校验。TK字段用于后续的加密。

再看一下EAPOL数据包的结构如下：



本想结合wireshark抓取数据包进行分析，怎奈何自己的电脑不给力，无法抓取到EAPOL数据包。只能获取到手机端的日志，如下所示，wpa_supplicant在正常情况下打印很少。可以看到明显的四次握手过程。



 

立两个flag：将来能够获取到EAPOL数据包时，再了解一下它的数据包结果。再者，随着工作经验的积累，总结一下常见的握手异常分析方法。

 

2.四次握手分析
2.1第一次握手：Authenticator->Supplicant
在本例中，Authenticator就是路由器，supplicant指的是手机端的supplicant。Authentor生存SNonce，然后将SNonce和自己的MAC地址发送出去。

supplicant接收到后，就可以计算出PTK。怎么来计算呢？计算公式可以参见名词解释小节，计算所用的输入值：PMK、AA、SPK、ANonce、SNonce都是已经知道的了，带入相应加密算法的公式，即可以得到PTK。

2.2第二次握手：Supplicant->Authenticator
Supplicant发送SNonce、自己的MAC、MIC给Authenticator。这里有引入了一个新的名词，我们在这里解释一下，MIC的中文解释是报文完整性校验值，怎么得到的呢？这样得到的：KCK（PTK中的前128位）加密该EAPOL报文得到的值，我们可以用这样一个公式来表示：MIC=mic(KCK,EAPOL)。

Authenticator接收到了之后，同样拥有了计算PTK的所有输入：PMK、AA、SPK、ANonce、SNonce，所有它也可以计算出PTK的值，然后取出KCK中的部分对EAPOL报文进行校验，即计算出MIC的值，如果该值与Supplicant计算出的值一致，则说明校验成功，否则，则说明校验失败。

更进一步，校验失败，则说明Supplicant端的PMK是错误的；

再进一步，PMK错误，则说明Supplicant端输入的密码是错误的。

2.3第三次握手：Authenticator->Supplicant
Authenticator发送的数据包括：组临时密钥GTK, WPA KEY MIC。GTK:用于后续更新组密钥，该密钥被KEK加密，KEK是PTK的中间128bit，MIC同样是KCK加密得来。

Supplicant接收到之后，同样进行MIC的检测（其实还有数据段的其他检测，在这里就不深入了）。如果校验成功了，Supplicant端就获取了GTK。

2.4第四次握手：Supplicant->Authenticator
Client 最后发送一次EAPOL-KEY给AP用于确认，如果认证成功，双方将安装（Install）key。而Authenticator中的校验也是对MIC进行检测。

什么是安装key?就是双发都决定了以后进行通信时，用什么Key来做加密（用了商定好的key加密、解密，双发的沟通才没有障碍）。Authenticator安装了PTK，Supplicant端安装了PTK和GTK。

最后附一下四次握手的流程图，该图是从80211-216文档中截取的。希望在深入学习后，根据自己的理解再画一下这个流程，因为毕竟“纸上得来终觉浅”嘛！

从下图可以看到，每次EAPOL-KEY的计算算法是有差异的，计算出来的MIC值肯定是不一样的。



 

3.名词解释：
PTK(pairwise transient key):成对传输秘钥，它用于单播数据帧的加密和解密

GTK(group temporal key):组临时秘钥，它用于组播数据帧和广播数据帧的加密和解密，管理帧、控制帧和空数据帧是不用加密的。

AA：client（即手机）的MAC

SPA：AP（即Authentor）的MAC

ANonce：AP产生的随机值

SNonce：Client 产生的随机值

PMK：跟认证方式有关，如果是PSK认证方式，PMK是由SSID和密码导出，公式如下：

PMK=pdkdf2_SHA1(passphrase,SSID,SSID length, 4096), 其中passphrase就是客户输入的登录密码。

如何计算PTK？

PMK转化成PTK是通过下面的函数完成的：

PTK<----PRF-X(pPMK,

"Pairwise key expansion", Min(AA,SPA)||Max(AA,SPA)||Min(ANoce,SNoce)||Max(ANonce, SNoce))。X指生成的PTK的长度，X=256+TK_bit，即KCK加KEK的256固定位加上TK位数，不同的加密方式TK_bits不一样。可查下表：
