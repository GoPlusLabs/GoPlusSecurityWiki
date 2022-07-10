# 伪邮件地址

作恶者可以使用和知名公司或组织一模一样的E-mail地址。例如，冒充OpenSea发送钓鱼邮件。

SMTP(Simple Mail Transfer Protocol) 协议定义了E-mail如何工作。该协议是基于TCP的应用层的协议，有三个阶段：建立连接、传输数据、终止连接。

再SMTP中有一些命令, 如：

| 命令   | 描述                                           |
| --------- | ----------------------------------------------------- |
| HELO      | Identify the domain name of the sending host to SMTP. |
| MAIL FROM | Specify the sender of the mail.                       |
| RCPT TO   | Specify the recipients of the mail.                   |
| DATA      | Define the following information as data.             |
| QUIT      | End an SMTP connection.                               |

诈骗者可以将`HELO`设置为任何想冒充的E-mail地址。

通过检查发件人的IP我们可以在某种程度上确定该邮件为仿冒。不过该方法对普通用户而言相当有难度，他们还是容易遭受伪邮件地址攻击。

