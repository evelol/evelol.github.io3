

1: 索引 索引并不是越多越好，索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率，因为 insert 或 update 时有可能会重建索引

2: 敏感信息的查询
在进行敏感信息查询之前，我们先介绍下加密的方式及类型，现在的加密方式分位两种，一种是可逆的，一种的不可逆的，如Rsa 和 Des就是可逆的，而MD5和Sha256等等就是不可逆的，可逆和不可逆的区别就是加密完成后还能不能解出来，虽然现在Md5很多md5在线解密之类的，但是那些都是暴力破解，他们的数据库有很庞大的数据，但是要是md5稍微做些操作，如加盐md5和加盐hash等等，他们的破解就没这么容易了。

回到主题：不可逆的加密一般都应用于密码上，而那些查询会相对比较简单，如我们查询一个用户登录密码是否正确，首先是根据用户名查询对应的记录，然后对用户上传的密码进行对比，此处是加盐md5。

`md5(md5(password) + salt) == mysql_password`

这样就能轻易的做出查询了，速度也在0.01s之内就能完成！

而可逆的加密算法是应用于那些加密之后，自己还能够解出来的，如银行卡，身份证等等敏感信息，这些信息保存数据库，我们一般会采用Rsa来加密，那这时候我们要对这些银行卡，身份证信息作查询得怎么办呢？因为MySQL中没有内置的rsa加密函数，如何是好呢？
我们可对这些数据再加一个字段，做加盐hash或者是加盐md5
对应的查询语句就会变成如下(此处列举加盐hash)，加盐hash的算法是
hash('sha256',hash('sha256','加密字符串').'salt') 
$bank = hash('sha256', "6221558812340000");//hash sha256加密 //数据库查询 SELECT * FROM 表名 WHERE BANK_SHA256 = SHA2(CONCAT('$bank' , 'salt'));
其中 SHA2 为mysql内置函数.

为了提高查询效率, 采用分表查询, 分组字段查询策略. 对于无法加索引或敏感信息比较有用.

所有散列函数都有如下一个基本特性：如果两个散列值是不相同的（根据同一函数），那么这两个散列值的原始输入也是不相同的。这个特性是散列函数具有确定性的结果。但另一方面，散列函数的输入和输出不是一一对应的，如果两个散列值相同，两个输入值很可能是相同的，但并不能绝对肯定二者一定相等。输入一些数据计算出散列值，然后部分改变输入值，一个具有强混淆特性的散列函数会产生一个完全不同的散列值。

2
