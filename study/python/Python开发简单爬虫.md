* 爬虫：一段自动抓取互联网信息的程序
  * 价值：互联网数据，为我所用

![image-20200204175952051](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200204175952051.png)

![image-20200204180211491](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200204180211491.png)

![image-20200204180356218](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200204180356218.png)

![image-20200204180540549](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200204180540549.png)

![image-20200204180653796](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200204180653796.png)

* Python有哪几种网页下载器

  * urllib2：Python官方基础模块

    * 方法1：最简洁方法

      ```python
      import urllib2
      
      # 直接请求
      response = urllib2.urlopen('http://www.baidu.com')
      # 获取状态码，如果是200表示获取成功
      print response.getcode()
      # 读取内容
      cont = response.read()
      ```

    * 方法2：添加data、http header

      ```python
      # url data header作为参数传给urllib2.Request
      # urllib2.urlopen(request)
      ```

      ```python
      import urllib2
      
      # 创建Request对象
      request=urllib2.Request(url)
      # 添加数据
      request.add_data('a','1')
      # 添加http的header
      request.add_header('User-Agent','Mozilla/5.0')
      # 发送请求获取结果
      response=urllib2.urlopen(request)
      ```

    * 方法3：添加特殊场景的处理器

      ```python
      # HTTPCookieProcessor（需登录）、ProxyHandler（代理）、HTTPSHandler（HTTPS）、HTTPRedirectHandler 作为参数
      # opener = urllib2.build_opener(handler)
      # urllib2.install_opener(opener)
      # urllib2.urlopen(url)
      # urllib2.urlopen(request)
      ```

      ```python
      import urllib2,cookielib
      
      # 创建cookie容器
      cj = cookielib.CookieJar()
      # 创建1个opener
      opener = urllib2.build_opener(urllib2.HTTPCookieProcessor(cj))
      # 给urllib2安装opener
      urllib2.install_opener(opener)
      # 使用带有cookie的urllib2访问网页
      response=urllib2.urlopen('http://www.baidu.com/')
      ```

    * ![image-20200205210812040](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200205210812040.png)

    * ![image-20200205210959796](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200205210959796.png)

  * requests：第三方包更强大

![image-20200205211145041](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200205211145041.png)

![image-20200205211259703](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200205211259703.png)

![image-20200205211424412](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200205211424412.png)

![image-20200205211515142](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200205211515142.png)

![image-20200205212128496](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200205212128496.png)

![image-20200205212224273](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200205212224273.png)

![image-20200205212321028](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200205212321028.png)

![image-20200205212437387](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200205212437387.png)

![image-20200205212518269](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200205212518269.png)

![image-20200205213413000](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200205213413000.png)

![image-20200205213239258](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200205213239258.png)

![image-20200205213528957](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200205213528957.png)

![image-20200205213946678](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200205213946678.png)

