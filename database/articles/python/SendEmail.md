电子邮件的传输协议是 SMTP，Python 内置了对 SMTP 协议的支持，对应的模块是 smtplib 和 email 模块。

- smtplib

  smtplib 模块负责在发送邮件的过程中和服务器通信，简单来说就是：

  - 连接服务器
  - 登录邮箱
  - 发送邮件
  - 退出服务器

- email

  email 模块负责和服务器通信的信息，比如邮件内容，附件，收件人，发件人，主题等等。

也就是说 smtplib 模块负责发邮件，email 模块负责写邮件

发送一份邮件的步骤：

用 qq 邮箱举例：

1. 第一步要做的就是连接服务器。这里用到的是 smtplib 模块的 SMTP 方法：

   - host，SMTP 服务器主机地址
   - port，端口号，默认端口是 25

   需要传入上面两个参数，因为不知道所以查一下：

   ![](/madao.github.io/database/images/articles/python/send_email/image.png)

   在 qq 邮箱的帮助中心找到了。

   但是还没完，因为邮件是私密的信息，不可能用明文传输，所以一般都会加密，所以还要找一下 qq 邮箱的加密标准。

   同样的在 qq 邮箱帮助中心找到，是 SSL 方式加密的。

   ![](/madao.github.io/database/images/articles/python/send_email/image1.png)

   smtp 模块的不同的加密写法如下：

   ```

   import smtplib

   # 端口是用TLS加密

   smtplib_obj = smtplib.SMTP('服务器地址',端口)

   # 端口是用SSL加密

   smtplib_obj = smtplib.SMTP_SSL('服务器地址',端口)

   ```

   好了，现在就写代码，连接服务器：

   ```
   import smtplib

   qq_mail = smtplib.SMTP_SSL('smtp.qq.com', 465)
   ```

2. 然后写用户名密码登录邮箱：

   ```
   from_address = input('请输入邮箱账号：')
   password = input('请输入邮箱密码：')
   qq_mail.login(from_address, password)
   ```

   登录是用 login 方法

3. 写邮件，这里先不用 email 模块去构造，先直接写一个看看

   ```
   to_address = input('请输入收件人地址：')
   message = input('请输入邮件内容：')
   ```

   获取收件人地址和邮件内容

4. 发送邮件

   ```
   qq_mail.sendmail(from_address, [to_address], message)
   ```

   sendmail 方法用来发送邮件，它的参数是：

   - from_addr: 邮件发送者地址。
   - to_addrs: 字符串列表，邮件发送地址。
   - msg: 发送消息

5. 退出服务器

   ```
   qq_mail.quit()
   ```

6. 运行

   这里还有一个小坑，如果你的文件名叫 email.py 的话，它会报这个错

   ```
   AttributeError: module 'smtplib' has no attribute 'SMTP_SSL'
   ```

   具体原因看这里[stackoverflow](https://stackoverflow.com/questions/16512256/no-attribute-smtp-error-when-trying-to-send-email-in-python)，解决办法就是不要用 email 这个文件名

   如果按照上面的例子一步一步写，那么会出现这个错误：

   smtplib.SMTPAuthenticationError

   这是因为你的邮箱没有开启 SMTP 服务。现在去开启：
   在【设置】- 【账户设置】往下拉会看到这个：

   ![](/madao.github.io/database/images/articles/python/send_email/image2.png)

   开启它

   然后你会得到一个 16 位的授权码，你用 smtplib 登录邮箱的密码就是这个，不再是邮箱原本的密码。

   ![](/madao.github.io/database/images/articles/python/send_email/image3.png)

   用户就会收到一封这样的邮件，没有发件人没有主题，连我们输入的邮件内容都没有

7. 使用 email 模块添加主题等信息

   - 使用 MIMEText 构造邮件内容。

     email.mime.text.MIMEText(\_text[, \_subtype[, _charset]])

     MIMEText 类用于创建主要类型 text 的 MIME 对象。

     - \_text: 文本内容
     - \_subtype：指定文本类型，支持 plain（默认值）或 html 类型的字符串。
     - \_charset：用于指定文本内容的编码表

     ```
     from email.mime.text import MIMEText

     msg = MIMEText('你好，这是用python发送的邮件', 'plain', 'utf-8')
     ```

   - 使用 Header 添加邮件的收件人，发件人，主题

     email.header.Header(s=None, charset=None, maxlinelen=None, header_name=None, continuation_ws=' ', errors='strict')

     创建一个 MIME 兼容的头，可以包含不同字符集中的字符串。

     常用的参数是 s 和 charset（其他参数还没搞清楚 +\_+）

     - s：初始标头值
     - charset：charset 指定编码表

     ```

     from email.header import Header

     msg['From'] = Header("发件人",'utf-8')

     msg['To'] = Header("收件人",'utf-8')

     msg['Subject'] = Header('主题', 'utf-8')

     ```

   - 使用 parseaddr, formataddr 格式化邮件头部信息。

     - email.utils.parseaddr(address)

       email.utils.parseaddr 接受包含了邮件地址的字符串，然后返回一个解析后的元组。例子：

       ```
       print(parseaddr('name<email_address@qq.com>'))
       # ('name', 'email_address@qq.com')
       ```

     - email.utils.formataddr(pair, charset='utf-8')
       formataddr 是 parseaddr 方法的逆，接受包含收件人昵称和邮箱账号的元组或列表，返回一个字符串，例子：
       ```
       print(formataddr(('name', 'email_address@qq.com'), 'utf-8'))
       # name <email_address@qq.com>
       ```

   - 发送

     ```
     qq_mail.sendmail(from_address, [to_address], msg.as_string())
     ```

     as_string 是 MIME 实例对象的方法，将整个消息作为字符串返回。

   我在看廖雪峰大神的 Python 教程的时候，发现他的收件人信息等等都进行了编码处理，教程上说如果有中文的话需要通过 Header 对象进行编码，我自己写的时候没有编码，直接用的中文也是成功了，所以我也不确定需不需要加这个处理。这里还是加上吧，可能不加会有一些我没遇到的坑，最终用 email 模块构造邮件的代码是：

   ```

   import smtplib

   from email.header import Header

   from email.mime.text import MIMEText

   from email.utils import parseaddr, formataddr

   def format_addr(str):

       name, address = parseaddr(str)

       return formataddr([Header(name, 'utf-8').encode(), address])

   try:

       qq_mail = smtplib.SMTP_SSL('smtp.qq.com', 465)

       from_address = input('请输入邮箱账号：')

       password = input('请输入邮箱密码：')

       qq_mail.login(from_address, password)

       to_address = input('请输入收件人地址：')

       message = input('请输入邮件内容：')

       msg = MIMEText(message, 'plain', 'utf-8')

       msg['From'] = format_addr('神秘人 <%s>' % from_address)

       msg['To'] = format_addr('另一个神秘人 <%s>' % to_address)

       msg['Subject'] = Header('今天吃了吗？', 'utf-8').encode()

       qq_mail.sendmail(from_address, [to_address], msg.as_string())

       print('发送成功')

       qq_mail.quit()

   except smtplib.SMTPException as e:

       print('邮件发送失败，原因:%s' % e)

   ```

   结果
   ![](/madao.github.io/database/images/articles/python/send_email/image4.png)

   但是收件人的名字并不是代码里写的，而是 qq 的昵称，可能腾讯对这个做了什么限制吧。

   一般的文本内容发送就是以上这些了，下面看一下发送附件怎么发送。

- 发送 html 邮件

  发送 html 的邮件很简单，只需要将 MIMEText 方法中的 plain 改为 html，然后 text 参数传入 html 字符串。例子：

  ```

  message = '''

      <!DOCTYPE html>

      <html lang="en">

      <head>

          <meta charset="UTF-8">

          <title>test</title>

      </head>

      <body>

          <h1>google</h1>

          <a href="https://www.google.com/">点我</a>

      </body>

      </html>'''

  msg = MIMEText(message, 'html', 'utf-8')

  ```

  ![](/madao.github.io/database/images/articles/python/send_email/image5.png)

- 发送图片

  图片可以作为邮件本身的内容，也可以作为附件发送。先看作为附件怎么发送：

  ```

  from email.mime.base import MIMEBase

  from email.mime.multipart import MIMEMultipart  # 带多个部分的邮件，比如既有图片又有文本

  from email.mime.text import MIMEText  # html格式和文本格式的对象

  from email.mime.image import MIMEImage  # 附件为图片格式的对象

  from email.mime.audio import MIMEAudio  # 附件为音频格式的对象

  from email import encoders  # 编码器

  ```

  把图片作为附件发送

  ```

  # 因为既有文本又有图片附件，所以用MIMEMultipart创建实例

  msg = MIMEMultipart()

  # attach方法相当于给msg对象，添加不同的部分

  msg.attach(MIMEText('发个图片', 'plain', 'utf-8'))

  # 从本地读取一个图片添加为附件:

  with open('./test.jpeg', 'rb') as f:

      mes_image = MIMEImage(f.read())

      # 给图片一个id，如果图片放入html中，那么就要用到它

      mes_image.add_header('Content-ID', 'img_0')

      # 给附件一个名字

      mes_image.add_header('Content-Disposition', 'attachment', filename='test.jpeg')

      # 添加到MIMEMultipart

      msg.attach(mes_image)

  ```

  结果：

  ![](/madao.github.io/database/images/articles/python/send_email/image6.png)

  完整代码：

  ```

  import smtplib

  from email.header import Header

  from email.mime.text import MIMEText

  from email.utils import parseaddr, formataddr

  from email.mime.multipart import MIMEMultipart

  from email.mime.image import MIMEImage

  def format_addr(str):

      name, address = parseaddr(str)

      return formataddr([Header(name, 'utf-8').encode(), address])

  try:

      qq_mail = smtplib.SMTP_SSL('smtp.qq.com', 465)

      from_address = input('请输入邮箱账号：')

      password = input('请输入邮箱密码：')

      qq_mail.login(from_address, password)

      to_address = input('请输入收件人地址：')

      # 因为既有文本又有图片附件，所以用MIMEMultipart创建实例

      msg = MIMEMultipart()

      # attach方法相当于给msg对象，添加不同的部分

      msg.attach(MIMEText('发个图片', 'plain', 'utf-8'))

      # 从本地读取一个图片添加为附件:

      with open('./test.jpeg', 'rb') as f:

          mes_image = MIMEImage(f.read())

          # 给图片一个id，如果图片放入html中，那么就要用到它

          mes_image.add_header('Content-ID', 'img_0')

          # 给附件一个名字

          mes_image.add_header('Content-Disposition', 'attachment', filename='test.jpeg')

          # 添加到MIMEMultipart

          msg.attach(mes_image)

      msg['From'] = format_addr('神秘人 <%s>' % from_address)

      msg['To'] = format_addr('另一个神秘人 <%s>' % to_address)

      msg['Subject'] = Header('今天吃了吗？', 'utf-8').encode()

      qq_mail.sendmail(from_address, [to_address], msg.as_string())

      print('发送成功')

      qq_mail.quit()

  except smtplib.SMTPException as e:

      print('邮件发送失败，原因:%s' % e)

  ```

  作为邮件内容发送，作为邮件内容发送，需要使用 html 格式的内容，然后将图片的 id 写在 img 标签的 src 属性里就可以了：

  ```

  msg.attach(MIMEText('''

      <!DOCTYPE html>

      <html lang="en">

      <head>

          <meta charset="UTF-8">

          <meta name="viewport" content="width=device-width, initial-scale=1.0">

          <meta http-equiv="X-UA-Compatible" content="ie=edge">

          <title>Document</title>

      </head>

      <body>

          <p>发个图片</p>

          <!-- 这里的id，就是mes_image.add_header('Content-ID', 'img_0')这里设置的id-->

          <img src="cid:img_0"

      </body>

      </html>

  ''', 'html', 'utf-8'))

  ```

  ![](/madao.github.io/database/images/articles/python/send_email/image7.png)
