加密的类型：
在日常设计及开发中，为确保数据传输和数据存储的安全，可通过特定的算法，将数据明文加密成复杂的密文。目前主流加密手段大致可分为单向加密和双向加密。

单向加密：通过对数据进行摘要计算生成密文，密文不可逆推还原。算法代表：Base64，MD5，SHA;

双向加密：与单向加密相反，可以把密文逆推还原成明文，双向加密又分为对称加密和非对称加密。

对称加密：指数据使用者必须拥有相同的密钥才可以进行加密解密，就像彼此约定的一串暗号。算法代表：DES，3DES，AES，IDEA，RC4，RC5;

非对称加密：相对对称加密而言，无需拥有同一组密钥，非对称加密是一种“信息公开的密钥交换协议”。非对称加密需要公开密钥和私有密钥两组密钥，公开密钥和私有密钥是配对起来的，
也就是说使用公开密钥进行数据加密，只有对应的私有密钥才能解密。这两个密钥是数学相关，用某用户密钥加密后的密文，只能使用该用户的加密密钥才能解密。如果知道了其中一个，并
不能计算出另外一个。因此如果公开了一对密钥中的一个，并不会危害到另外一个密钥性质。这里把公开的密钥为公钥，不公开的密钥为私钥。算法代表：RSA，DSA。
　　以前一直对客户端传给服务器的信息加密这一块一脸懵，如果app里面的用户登录信息被抓包拿到了，大写着 username:root，password:123456， 那不是很尴尬。

偶然做版权输入的时候遇到了rsa，在支付宝支付的时候也接触过，当时不知道这是啥子，现在才知道。

他能保证，客户端给出的信息，只有拥有私钥的服务器才能看，其他人看的都是乱码，嘿嘿。

非对称加密算法
需要两个密钥：公开密钥（publickey）和私有密钥（privatekey）。
公开密钥与私有密钥是一对，如果用公开密钥对数据进行加密，只有用对应的私有密钥才能解密；
如果用私有密钥对数据进行加密，那么只有用对应的公开密钥才能解密。
因为加密和解密使用的是两个不同的密钥，所以这种算法叫作非对称加密算法。
注意以上的一个点，公钥加密的数据，只有对应的私钥才能解密

在日常使用中是酱紫的：

将私钥private_key.pem用在服务器端，公钥发放给android跟ios等前端
客户端用公钥加密过后，数据只能被拥有唯一私钥的服务器看懂。

具体实现：
1、加密解密的第一步是生成公钥、私钥对，私钥加密的内容能通过公钥解密（反过来亦可以）

1 下载开源RSA密钥生成工具openssl（通常Linux系统都自带该程序），解压缩至独立的文件夹，进入其中的bin目录，执行以下命令：
2 a、openssl genrsa -out rsa_private_key.pem 1024
3 b、openssl pkcs8 -topk8 -inform PEM -in rsa_private_key.pem -outform PEM -nocrypt -out private_key.pem
4 c、openssl rsa -in rsa_private_key.pem -pubout -out rsa_public_key.pem
5
6 第一条命令生成原始 RSA私钥文件 rsa_private_key.pem
7 第二条命令将原始 RSA私钥转换为 pkcs8格式
8 第三条生成RSA公钥 rsa_public_key.pem
9
10 上面几个就可以看出：通过私钥能生成对应的公钥
也有一些网站提供生成rsa公钥私钥的服务：http://www.bm8.com.cn/webtool/rsa/

2、PHP的加密解密类库：

```php
<?php
 
 
class Rsa {
 
    /**     
     * 获取私钥     
     * @return bool|resource     
     */    
    private static function getPrivateKey() 
    {        
        $abs_path = dirname(__FILE__) . '/rsa_private_key.pem';
        $content = file_get_contents($abs_path);    
        return openssl_pkey_get_private($content);    
    }    
 
    /**     
     * 获取公钥     
     * @return bool|resource     
     */    
    private static function getPublicKey()
    {   
        $abs_path = dirname(__FILE__) . '/rsa_public_key.pem';
        $content = file_get_contents($abs_path);    
        return openssl_pkey_get_public($content);     
    }
 
    /**     
     * 私钥加密     
     * @param string $data     
     * @return null|string     
     */    
    public static function privEncrypt($data = '')    
    {        
        if (!is_string($data)) {            
            return null;       
        }        
        return openssl_private_encrypt($data,$encrypted,self::getPrivateKey()) ? base64_encode($encrypted) : null;    
    }    
 
    /**     
     * 公钥加密     
     * @param string $data     
     * @return null|string     
     */    
    public static function publicEncrypt($data = '')   
    {        
        if (!is_string($data)) {            
            return null;        
        }        
        return openssl_public_encrypt($data,$encrypted,self::getPublicKey()) ? base64_encode($encrypted) : null;    
    }    
 
    /**     
     * 私钥解密     
     * @param string $encrypted     
     * @return null     
     */    
    public static function privDecrypt($encrypted = '')    
    {        
        if (!is_string($encrypted)) {            
            return null;        
        }        
        return (openssl_private_decrypt(base64_decode($encrypted), $decrypted, self::getPrivateKey())) ? $decrypted : null;    
    }    
 
    /**     
     * 公钥解密     
     * @param string $encrypted     
     * @return null     
     */    
    public static function publicDecrypt($encrypted = '')    
    {        
        if (!is_string($encrypted)) {            
            return null;        
        }        
    return (openssl_public_decrypt(base64_decode($encrypted), $decrypted, self::getPublicKey())) ? $decrypted : null;    
    }
}
```

### 调用demo：

```php
<?php
 
require_once "Rsa.php";
$rsa = new Rsa();
$data['name'] = 'Tom';
$data['age']  = '20';
$privEncrypt = $rsa->privEncrypt(json_encode($data));
echo '私钥加密后:'.$privEncrypt.'<br>';
 
$publicDecrypt = $rsa->publicDecrypt($privEncrypt);
echo '公钥解密后:'.$publicDecrypt.'<br>';
 
$publicEncrypt = $rsa->publicEncrypt(json_encode($data));
echo '公钥加密后:'.$publicEncrypt.'<br>';
 
$privDecrypt = $rsa->privDecrypt($publicEncrypt);
echo '私钥解密后:'.$privDecrypt.'<br>';
```





PHP中接口加密之非对称加密算法

非对称加密算法是相对于对称加密而言，这种加密算法的秘钥分为“公开秘钥”和“私有秘钥”。信息的发送方事先生成好一份公钥和私钥，然后将公钥发给所有的信息接收方。当发送方需要发送信息给接收方式时，先通过私钥将信息加密，然后在发送。而当信息的接收方法收到信息后通过公钥将信息解密。这种算法的优势在于，解密的私钥不需要传递，降低了私钥泄密的可能性。

代码实现：

```php
<?php
/**
* 生成公钥和私钥并保存
*/
public function generate(){
    $config = array(
        "digest_alg" => "sha512",//加密方式
        "private_key_bits" => 1024, //加密长度
        "private_key_type" => OPENSSL_KEYTYPE_RSA,//加密类型
    );
    $res = openssl_pkey_new($config);//创建公钥和私钥

    //获取私钥
    openssl_pkey_export($res, $privateKey);
    //保存密钥到文件
    file_put_contents('test.pfx',$privateKey);

    //获取公钥
    $pubblic = openssl_pkey_get_details($res);
    $pubblicKey = $pubblic['key'];
    file_put_contents('test.cer',$pubblicKey);
}

/**
* 信息加密
*/
public function enctype(){

    $data = "this is source data";//原始数据
    $encrypted = "sha1";//原始数据密码方式

    //获取私钥
    $pi_key = file_get_contents('test.pfx');

    openssl_private_encrypt($data,$encrypted,$pi_key);//使用私钥加密

    //加密后的内容通常含有特殊字符，需要编码转换下，在网络间通过url传输时要注意base64编码是否是url安全的
    $encrypted = base64_encode($encrypted);
    //加密并编码后的信息
    echo $encrypted,"\n";

}

/**
* 公钥解密
*/
public function detype(){
    //网络传输过来的数据 模拟的
    $encrypted = 'Aae7obkme78HtrWuTSjyOm4jZ1Aym4HSxP51pXWbk/h9E+XWhdxTi/AGoNJz/FQAQwKquCeuQ5bBuVtcqx3DA4OmppdhaoKZR6NcTIm0Fe378vtlHtrBez7GH6UrMx48r9348406teRCndSqus+/qKWDqx1BN3qOz99WePmQN6s=';

    //原始数据解码
    $decrypted = "sha1";

    //获取公钥
    $pu_key = file_get_contents('test.cer');

    //私钥加密的内容通过公钥可用解密出来
    openssl_public_decrypt(base64_decode($encrypted),$decrypted,$pu_key);

    echo $decrypted,"\n";

}
```



本文实例讲述了PHP封装的非对称加密RSA算法。分享给大家供大家参考，具体如下：

将php的openssl扩展中的非对称加密函数封装成一个Rsa类。

需要注意的是，在windows上，需要打开openssl的配置文件，请参照官方的openssl扩展安装文档。

**在windows上安装openssl扩展**

1、将php路径下的两个库文件libeay32.dll和ssleay32.dll复制到操作system32下

2、配置openssl配置文件的位置，在php的路径下，有文件extras/openssl/openssl.cnf，添加环境变量OPENSSL_CONF指向这个文件的全路径。如何添加环境变量请google搜索之。

3、在php.ini里添加一行extension=php_openssl.dll

**使用的demo：**

```php
<?php
//====================demo=======================
//以下是一个简单的测试demo，如果不需要请删除
$rsa = new Rsa('sslkey'); //sslkey为存放密钥的路径，将已有的密钥文件复制到该路径下，公钥名称为pub.key，私钥名称为priv.key
$rsa->createKey(); //创建一对密钥，如果密钥对已经存在，不需调用
//私钥加密，公钥解密
echo 'source：脚本之家<br />';
$pre = $rsa->privEncrypt('脚本之家');
echo 'private encrypted:<br />' . $pre . '<br />';
$pud = $rsa->pubDecrypt($pre);
echo 'public decrypted:' . $pud . '<br />';
//公钥加密，私钥解密
echo 'source:干IT的<br />';
$pue = $rsa->pubEncrypt('干IT的');
echo 'public encrypt:<br />' . $pue . '<br />';
$prd = $rsa->privDecrypt($pue);
echo 'private decrypt:' . $prd;
//========================demo======================
```

本示例在windows7、php 5.2.14、openssl 0.98下开发

```php
<?php
/**
 * 使用openssl实现非对称加密
 *
 */
class Rsa
{
  /**
   * private key
   */
    private $_privKey;
    /**
     * public key
     */
    private $_pubKey;
    /**
     * the keys saving path
     */
    private $_keyPath;
    /**
     * the construtor,the param $path is the keys saving path
     */
    public function __construct($path)
    {
        if(empty($path) || !is_dir($path)){
            throw new Exception('Must set the keys save path');
        }
        $this->_keyPath = $path;
    }
    /**
     * create the key pair,save the key to $this->_keyPath
     */
    public function createKey()
    {
        $r = openssl_pkey_new();
        openssl_pkey_export($r, $privKey);
        file_put_contents($this->_keyPath . DIRECTORY_SEPARATOR . 'priv.key', $privKey);
        $this->_privKey = openssl_pkey_get_private($privKey);
        $rp = openssl_pkey_get_details($r);
        $pubKey = $rp['key'];
        file_put_contents($this->_keyPath . DIRECTORY_SEPARATOR . 'pub.key', $pubKey);
        $this->_pubKey = openssl_pkey_get_public($pubKey);
    }
    /**
     * setup the private key
     */
    public function setupPrivKey()
    {
        if(is_resource($this->_privKey)){
            return true;
        }
        $file = $this->_keyPath . DIRECTORY_SEPARATOR . 'priv.key';
        $prk = file_get_contents($file);
        $this->_privKey = openssl_pkey_get_private($prk);
        return true;
    }
    /**
     * setup the public key
     */
    public function setupPubKey()
    {
        if(is_resource($this->_pubKey)){
            return true;
        }
        $file = $this->_keyPath . DIRECTORY_SEPARATOR . 'pub.key';
        $puk = file_get_contents($file);
        $this->_pubKey = openssl_pkey_get_public($puk);
        return true;
    }
    /**
     * encrypt with the private key
     */
    public function privEncrypt($data)
    {
        if(!is_string($data)){
            return null;
        }
        $this->setupPrivKey();
        $r = openssl_private_encrypt($data, $encrypted, $this->_privKey);
        if($r){
            return base64_encode($encrypted);
        }
        return null;
    }
    /**
     * decrypt with the private key
     */
    public function privDecrypt($encrypted)
    {
        if(!is_string($encrypted)){
            return null;
        }
        $this->setupPrivKey();
        $encrypted = base64_decode($encrypted);
        $r = openssl_private_decrypt($encrypted, $decrypted, $this->_privKey);
        if($r){
            return $decrypted;
        }
        return null;
    }
    /**
     * encrypt with public key
     */
    public function pubEncrypt($data)
    {
        if(!is_string($data)){
            return null;
        }
        $this->setupPubKey();
        $r = openssl_public_encrypt($data, $encrypted, $this->_pubKey);
        if($r){
            return base64_encode($encrypted);
        }
        return null;
    }
    /**
     * decrypt with the public key
     */
    public function pubDecrypt($crypted)
    {
        if(!is_string($crypted)){
            return null;
        }
        $this->setupPubKey();
        $crypted = base64_decode($crypted);
        $r = openssl_public_decrypt($crypted, $decrypted, $this->_pubKey);
        if($r){
            return $decrypted;
        }
        return null;
    }
    public function __destruct()
    {
        @ fclose($this->_privKey);
        @ fclose($this->_pubKey);
    }
}
```

