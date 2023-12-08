# procmail

两个配置文件： `$HOME/.procmailrc` 和 `/etc/procmailrc`

```
LOGFILE=$HOME/procmail.log
VERBOSE = yes
# 上面是设置 log，可以参考 man mailstat 格式化 log

PATH=/usr/local/bin:/usr/bin:/bin
MAILDIR=$HOME/Mail      #you'd better make sure it exists
DEFAULT=$MAILDIR/mbox   #completely optional
LOGFILE=$MAILDIR/from   #recommended

INCLUDERC = spam.procmailrc
INCLUDERC = lists.procmailrc

```

## procmail recipe

```
:0 <flags>
* <pattern>
<action>
```

```
:0
!me@some.other.host
```

action:

!		转发邮件
|		pipe 到后面的程序
filename	放进邮箱文件夹

## recipe 例子

```
:0
* ^From:.*@126.com
* ^To: me@m126.com
* < 25600
!my.other.email.address@other.host
```

所有从 126 发给我 126 邮箱的邮件，小于 25K ，转发到另一个邮箱

```
:0
* ^TO_webmaster@melonfire.com
WEBMASTER
```

^TO_		匹配 "To", "Cc"，等其他 Header

```
:0:
* From:.*redhat-list@redhat.com
LISTS

:0:
* From:.*php-general@lists.php.net
LISTS

:0:
* From:.*oracle-request@cs.indiana.edu
FUN
```

:0:		加锁，避免冲突


```
:0
* ^From: .*joe@annoyances.domain.com
/dev/null
```

屏蔽邮箱

```
:0 c
BACKUP

:0
!my.other.email.address@some.other.host
```

c		carbon copy, 复制一份邮件进行下面的 recipe

```
:0
* ^From:.*@known-bad-domain.com
SPAM

:0
* ^From:.*clouded_mind_99@hotmail.com
SPAM

:0
* ^Subject:.*make money fast
SPAM
```

过滤已知垃圾邮件

```
:0
* ? egrep -is -f /path/to/black-list.txt
SPAM

$ cat black-list.txt
angie76@spammer.com
clouded_mind_99@hotmail.com

:0
* !? egrep -is -f /home/me/white-list.txt
SPAM
```

```
:0
| (/usr/bin/formail -r ; cat autoresponse.txt) | /usr/sbin/sendmail -oi -t
```

仅做举例用，用 formail + sendmail 自动回复邮件，没考虑邮件列表等情景

```
:0fw:
| /usr/local/bin/spamassassin

:0:
* ^X-Spam-Status: Yes
SPAM
```

使用 [spamassassin](http://www.spamassassin.org/) 过滤垃圾邮件

# 常见的邮件相关协议

POP3 (Post Office Protocol 3)

POP3协议允许 MUA 下载服务器上的邮件，但是在 MUA 的操作（如移动邮件、标记已读等），不会反馈到服务器上。

IMAP (Internet Mail Access Protocol)

IMAP MUA收取的邮件仍然保留在服务器上，同时在MUA上的操作都会反馈到服务器上。

SMTP (Simple Mail Transfer Protocol)

用来发送邮件。
是一组用于从源地址到目的地址传输邮件的规范，通过它来控制邮件的中转方式。

# 名词解释

blind carbon-copy (BCC)

暗抄送

carbon-copy (Cc) 

抄送

# mutt

## mutt 常用快捷键

下一行	回车
上一行	backspace

下一页	space
上一页	减号

alias展开	tab

entry:
select-entry	Enter
exit		q

菜单：

j, k		上下一个条目
z, Z		上下一页
=, *		跳转至第一个或最后一个条目
q		退出菜单
?		列出当前所有键绑定

编辑器：

^A		行首
^B		向后一个字符
^F		向前一个字符

Esc B		向前一个单词
Esc F		向后一个单词

<Tab>		补全文件名或别名

Esc c		大写首字母
Esc u		单词变大写
Esc l		单词变小写

阅读邮件：

Index

c	转到其他邮箱

d	删除当前邮件
u	取消删除

o O	改变排序方法
<Tab>	跳转下一个新的或未读邮件
@	显示作者email

/
Esc /	反向搜索

q	保存更改并退出
x	放弃更改并退出

t	标记
Esc t	标记整个 thread

Pager

回车	向下一行
<backspace?	向上一行
<space>	向下一页
-	向上一页
^	跳转至邮件顶部

S	跳过引用
T	切换是否显示引用

\	突出显示搜索匹配项

Thread

^D	删除 thread
^U	取消删除 thread

^N	跳转到下一个 thread
^P	跳转到上一个 thread

^R	标记 thread 为已读

Esc v	折叠当前 thread
Esc V	折叠全部 thread


# thunderbird

Toggle Line Wrap扩展，可以在邮件编辑器界面中增加一键自动换行的功能。

该扩展安装后将在邮件编辑器右上角新增"Line Wrap"按钮，该按钮可以启动和关闭
自动换行模式。

- Line Wrap 快捷键为 Ctrl + Shift + W
- 每行的字符数在下面提到的mailnews.wraplength参数指定。
- Line Wrap 只对英文正文或者汉字有效，杂乱的随机字母组合不换行。
- 经测试，网址链接在邮件接收端不换行，因此不会被截断。
- 仅支持纯文本模式。

需要注意的是，该按钮可能是灰色不可用状态，则需要对thunderbird进行以下设置：

Thunderbird Settings -> General -> Config Editor...
在打开的Advanced Preferences页面中
  搜索 "mailnews.send_plaintext_flowed" 并设置为 false
  搜索 "mailnews.wraplength" 设置为 72

Thunderbird Settings -> Account Settings
  指定账户的 Composition & Addressing
    <unchecked> Compose messages in HTML format

# thunderbird + public-inbox-imapd

## public-inbox-config 参考

```
[publicinbox "linux-riscv-xmz"]
        address = linux-riscv@lists.infradead.org
        url = http://lore.kernel.org/linux-riscv
        inboxdir = /home/xmz/archive/mirror-lists/linux-riscv
        newsgroup = lists.linux-riscv
[publicinbox "tools-xmz"]
        address = tools@linux.kernel.org
        url = https://lore.kernel.org/tools
        inboxdir = /home/xmz/archive/mirror-lists/tools
        newsgroup = lists.tools
```

- PS: 以上config中的 newsgroup 参数值在IMAP服务中表示server端的存储目录

```
$ public-inbox-imapd
# bound imap://0.0.0.0:143
PID=13846 is worker[0]
```
## thunderbird 中操作

Settings -> Account Settings -> Account Actions -> Add Mail Account

  Your full name: xxx		; 必填
  Email Address: lists@tools	; 该项内容需满足邮箱格式才能进行下一步
  点击下方出现的 `Configure manually` -> 点击下侧的 `Advanced config`
  弹出提示菜单"...Do you want to proceed?" -> OK

PS:
1. 以下步骤建议按顺序执行，以免自动开启IMAP服务器端所有邮件的下载任务
2. 建议按如下方法每个mailling list单独开一个Account，否则目录显示会很乱

Synchronisation & Storage -> 
  < > Keep messages in all folders for this account on this computer (unchecked)
  该选项会自动检索IMAP服务器所有邮件并下载到本地

Disk Space 按需选择
  经测试 "Synchronise the most recent xx Days" 选项无效
  仍会自动下载文件夹内全部邮件到本地
  "Don't download messages larger than xx kB"  选项有效

Server Settings ->
  Server Settings ->
    Advanced.. ->
      IMAP server directory: lists.linux-riscv	(填public-inbox-config中的newsgroup值)
      < > Show only subscribed folders 		(unchecked)
    OK

Server Settings ->
  Server Name: 192.168.0.11			(localhost IP or IMAP server IP)
  Port: 143
  -> Restart

IMAP password 任意填

public-inbox 会将大的邮件列表分成多个文件夹存储，编号最大的文件夹为最新的邮件仓库。

PS: 
  预览邮件文件夹会自动下载该文件夹内的全部Header，并开始indexing
  大的邮件列表中该过程会耗时并占用CPU。
  https://support.mozilla.org/en-US/kb/rebuilding-global-database

## 配合 public-inbox-watch 自动更新mailing list镜像

### 邮件正文常见英文缩写

AFAIK	as far as I know.		据我所知
IOW	in other words.			换句话说
FWIW	for what it’s worth.		不管是否有价值
YMMV	your mileage may vary.		见仁见智
IIRC	if I remember correctly.	如果我没记错的话
IMHO	in my humble opinion.		管见所及

更多内容可参考：
http://vger.kernel.org/lkml/#contributors
