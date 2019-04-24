- [一、什么是RSA](#%E4%B8%80%E4%BB%80%E4%B9%88%E6%98%AFrsa)
- [二、RSA算法密钥长度的选择](#%E4%BA%8Crsa%E7%AE%97%E6%B3%95%E5%AF%86%E9%92%A5%E9%95%BF%E5%BA%A6%E7%9A%84%E9%80%89%E6%8B%A9)
- [三、C#中的RSA加解密](#%E4%B8%89c%E4%B8%AD%E7%9A%84rsa%E5%8A%A0%E8%A7%A3%E5%AF%86)

## 一、什么是RSA

RSA公开密钥密码体制。所谓的公开密钥密码体制就是使用不同的加密密钥与解密密钥，是一种“由已知加密密钥推导出解密密钥在计算上是不可行的”密码体制。 　　

　　在公开密钥密码体制中，加密密钥（即公开密钥）PK是公开信息，而解密密钥（即秘密密钥）SK是需要保密的。加密算法E和解密算法D也都是公开的。虽然密钥SK是由公开密钥PK决定的，但却不能根据PK计算出SK。正是基于这种理论，1978年出现了著名的RSA算法，它通常是先生成一对RSA 密钥，其中之一是保密密钥，由用户保存；另一个为公开密钥，可对外公开，甚至可在网络服务器中注册。为提高保密强度，RSA密钥至少为500位长，一般推荐使用1024位。这就使加密的计算量很大。为减少计算量，在传送信息时，常采用传统加密方法 与公开密钥加密方法相结合的方式，即信息采用改进的DES或IDEA对话密钥加密，然后使用RSA密钥加密对话密钥和信息摘要。对方收到信息后，用不同的 密钥解密并可核对信息摘要。 　　

　　RSA算法是第一个能同时用于加密和数字签名的算法，也易于理解和操作。RSA是被研究得最广泛的公钥算法，从提出到现在的三十多年里，经历了各种攻击的考验，逐渐为人们接受，普遍认为是目前最优秀的公钥方案之一。

## 二、RSA算法密钥长度的选择
1. 非对称加密算法中1024 bit密钥的强度相当于对称加密算法80bit密钥的强度。

2. 密钥长度增长一倍，公钥操作所需时间增加约4倍，私钥操作所需时间增加约8倍，公私钥生成时间约增长16倍。

3. 一次能加密的密文长度与密钥长度成正比，加密后的密文长度跟密钥长度相同(RSA加密内容的长度有限制，和密钥长度有关，这是由它的算法决定的)

- a、加密的明文长度不能超过RSA密钥的长度减去11byte，比如密钥长度是1024位的，1024位=1024bit=128byte，128-11=117byte，所以明文长度不能超过117byte，如果长度超过该值将会抛出异常。

- b、加密后密文的长度为密钥的长度，如密钥长度为1024bit(128Byte)，最后生成的密文固定为 1024bit(128Byte)。  
##  三、C#中的RSA加解密
　.NET Framework 类库提供了System.Security 命名空间，System.Security 命名空间提供公共语言运行时安全系统的基础结构，包括权限的基类，而该命名空间下提供了RSACryptoServiceProvider类来执行RSA算法的不对称加密和解密。
1.密钥对的生成:

a、根据RSACryptoServiceProvider直接生成

``` csharp
/// <summary>
/// 生成密钥
/// </summary>
public RSAKey GenerateRSAKey()
{
    RSAKey RSAKEY = new RSAKey();
    RSACryptoServiceProvider RSA = new RSACryptoServiceProvider();
    RSAKEY.PrivateKey = RSA.ToXmlString(true);    //生成私钥
    RSAKEY.PublicKey = RSA.ToXmlString(false);    //生成公钥
    RSA.Clear();
    return RSAKEY;
}
```
b、通过Makecert证书创建工具生成安全证书

``` cmd
makecert -r -pe -n "CN=RSAKey" -b 03/31/2005 -e 12/31/2012 -sky exchange -ss my
```
可通过"Visual Studio命令提示行"执行以上命令生成证书。

查看生成的证书：

运行->输入mmc打开控制台->选择文件->添加/删除管理单元->在弹出框左侧找到证书->选中证书添加->选择我的用户账户->完成确定

此时就可以在对应位置查看到我们刚刚创建的名为RSAKey的证书了，如下图:
![RSAKEY](images/01.png?raw=true)


``` csharp
/// <summary>
/// 创建加密RSA
/// </summary>
/// <param name="publicKey">公钥</param>
/// <returns></returns>
private RSACryptoServiceProvider CreateEncryptRSA(string publicKey)
{
    try
    {
        RSACryptoServiceProvider RSA = new RSACryptoServiceProvider();
        RSA.FromXmlString(publicKey);
        return RSA;
    }
    catch (CryptographicException ex)
    {
        throw ex;
    }
}

/// <summary>
/// 创建解密RSA
/// </summary>
/// <param name="privateKey">私钥</param>
/// <returns></returns>
private RSACryptoServiceProvider CreateDecryptRSA(string privateKey)
{
    try
    {
        RSACryptoServiceProvider RSA = new RSACryptoServiceProvider();
        RSA.FromXmlString(privateKey);
        return RSA;
    }
    catch (CryptographicException ex)
    {
        throw ex;
    }
}

/// <summary>
/// 根据安全证书创建加密RSA
/// </summary>
/// <param name="certfile">公钥文件</param>
/// <returns></returns>
private RSACryptoServiceProvider X509CertCreateEncryptRSA(string certfile)
{
    try
    {
        X509Certificate2 x509Cert = new X509Certificate2(certfile);
        RSACryptoServiceProvider RSA = (RSACryptoServiceProvider)x509Cert.PublicKey.Key;
        return RSA;
    }
    catch (CryptographicException ex)
    {
        throw ex;
    }
}

/// <summary>
/// 根据私钥文件创建解密RSA
/// </summary>
/// <param name="keyfile">私钥文件</param>
/// <param name="password">访问含私钥文件的密码</param>
/// <returns></returns>
private RSACryptoServiceProvider X509CertCreateDecryptRSA(string keyfile, string password)
{
    try
    {
        X509Certificate2 x509Cert = new X509Certificate2(keyfile, password);
        RSACryptoServiceProvider RSA = (RSACryptoServiceProvider)x509Cert.PrivateKey;
        return RSA;
    }
    catch (CryptographicException ex)
    {
        throw ex;
    }
}

```
