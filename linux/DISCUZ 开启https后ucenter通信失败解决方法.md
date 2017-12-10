#DISCUZ 开启https后ucenter通信失败解决方法

DISCUZ 开启https后ucenter通信失败解决方法，一般是做完301重定向https后通信失败的，下面是具体解决方法：

打开目录 `uc_server/model/misc.php` 文件；

找到69行（如下图），插入下面代码：

```
if(substr($url,0,5)=='https'){
	$ch = curl_init($url);
	curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
	if($post){
		curl_setopt($ch, CURLOPT_POST, 1);
		curl_setopt($ch, CURLOPT_POSTFIELDS, $post);
	}
	if($cookie){
		curl_setopt($ch, CURLOPT_COOKIE, $cookie);
	}
	curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE);
	return curl_exec($ch);
}

```

