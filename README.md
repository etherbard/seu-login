# loginSEU
## 1.手动登陆，观察网页信息
通过Chrome手动登陆，按F12观察Network信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201024133258798.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzAyMjI1MQ==,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201024133320175.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzAyMjI1MQ==,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020102413344813.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzAyMjI1MQ==,size_16,color_FFFFFF,t_70#pic_center)

通过login信息可以知道向服务器request的方式是POST，发送信息的URL是https://newids.seu.edu.cn/authserver/login?goto=http://my.seu.edu.cn/index.portal

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201024133340322.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzAyMjI1MQ==,size_16,color_FFFFFF,t_70#pic_center)
进一步观察可以看到Post表单的格式和数据，username的值为一卡通号，password的值是一卡通密码在浏览器端通过JavaScript加密得到的数据。经过多次实验发现，dllt、_eventId、rmShown等字段的值均为固定值，而lt和execution的值会发生变换。因此，现在的任务是找出lt、execution和password的加密方式。

## 2.搜索网页代码寻找线索
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201024133523167.png#pic_center)

在登陆界面网页代码中搜索lt、execution、password等字段，发现在网页中表单已经包含这些数据，不过设置为hidden不能直接观察到。因此只需要先通过爬虫访问登陆页面即可得到这些数据。

另外，我们还在表单中发现了一项pwdDefaultEncryptSalt，猜测大概率是加密使用的密钥。继续使用该字段作为关键词搜索，发现一个函数调用：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201024133554126.png#pic_center)

将这个js的代码保存至本地，通过VSCode打开。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201024133612513.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201024133624352.png#pic_center)


不难发现pwdDefaultEncryptSalt作为参数传递给了另一个函数encryptAES，继续搜索，发现一个encrypt.js中包含该函数的定义：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201024133637198.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzAyMjI1MQ==,size_16,color_FFFFFF,t_70#pic_center)


至此，POST表单的所以秘密已经解开了，只需将密码和从网页代码中提取的pwdDefaultEncryptSalt作为参数调用这个js的encryptAES函数即可得到加密后的密码。

## 3.编写爬虫

爬虫使用requests、re、execjs和三个库。首先通过requests.session()建立会话并请求登陆页面，从登陆页面提取pwdDefaultEncryptSalt、execution等字段，再调用encrypt.js获得加密后的password参数。最后用得到的参数构造POST表单并发送传递给服务器即可成功登陆。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201024133648613.png#pic_center)
