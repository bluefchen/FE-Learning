#### HTTPS身份认证

###### 证书主要包含三个部分：
- tbsCertificate(to be signed certificate)，待签名证书
- SignatureAlgorithm，签名算法（指定哈希算法）
- SignatureValue，签名值
tbsCertificate为明名方式传输，里面包含证书的一些属性：版本号，序列号，发行者，公钥等

它们之间的关系为：


                    哈希函数	            证书私钥签名
    tbsCertificate---------- -> 消息摘要------------------>SignatureValue

###### 如何进行身份认证？
1. 浏览器向Server发送请求；
2. Server返回证书，包含以上三部分
	. 浏览器读取证书中的tbsCeriticate部分，使用SignatureAlgorithm中的哈希算法计算得到信息摘要	与	通过公钥解密SignatureValue得到的信息摘要	判断是否一致。
4. 一致则在功，不一致则失败。

###### 证书信任链机制
以上方式验证的只是当前站点的证书。实际上Server发送的是一个证书链。从当前网站开始，逐级往上，CA的有效性依赖于更高级的CA签名认证，一层层递推上去，到根证书。根证书的顶端内置在浏览器中，被浏览器所信任。

为什么要使用证书链？
1. 安全
2. 保持CA的私钥离线，方便部署和撤销。