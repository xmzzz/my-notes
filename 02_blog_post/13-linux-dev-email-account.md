

Mailbox activated. Please allow few minutes for changes to take effect.

# Mailbox Setup

mingzheng.xing@linux.dev
Your mailbox has now been activated. Use the settings below to configure your email clients and start using your new email address.

ProTip: You may want to save or print this page for later reference.


Incoming Mail
Protocol	IMAP
Server		imap.migadu.com
Port		993
Security	TLS
Authentication	Password
Username	mingzheng.xing@linux.dev
Password	(mailbox password)

Outgoing Mail
Protocol	SMTP
Server		smtp.migadu.com
Port		465
Security	TLS
Authentication	Password
Username	mingzheng.xing@linux.dev
Password	(mailbox password)

Webmail Access
Address		https://webmail.migadu.com
Username	mingzheng.xing@linux.dev
Password	(mailbox password)

ManageSieve
Protocol	ManageSieve
Server		imap.migadu.com
Port		4190
Security	StartTLS
Authentication	Password
Username	mingzheng.xing@linux.dev
Password	(mailbox password)
WARNING: By default, ManageSieve is disabled. Sieve is a simple language but faulty scripts may cause lost messages. Use with care.


# Alternative Access Possibilities

If your email client or application does not support our recommended settings above, you can try the alternatives below.


POP3 Access
Protocol	POP3
Server		pop.migadu.com
Port		995
Security	TLS
Authentication	Password
Username	mingzheng.xing@linux.dev
Password	(mailbox password)
IMPORTANT: POP3 is nowadays considered an archaic protocol. It does not support folders or multi-device access. It is mostly used for picking up messages from a server. Please note that without folders support, messages filtered into the Junk folder are not visible over POP3 protocol.


SMTP / StartTLS
Protocol	SMTP
Server		smtp.migadu.com
Port		587
Security	StartTLS
Authentication	Password
Username	mingzheng.xing@linux.dev
Password	(mailbox password)
NOTE: The current recommendation is to use implicit TLS on port 465 as shown above for message submission. However, we also support StartTLS on port 587. There is no practical security difference though as TLS is always required on our end for submission.
