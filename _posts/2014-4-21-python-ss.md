---
ahtour: Fish
layout: post
title: Python 邮件发送脚本
categories: Python
tags: email
---
{% highlight python %}

#!/usr/bin/python
#coding : utf-8
import sys
import smtplib
#init server
USAGE = ''
The usages are blow:

python mailto **@xx.com -t 'this is title' -c 'this is content'
or
python mailto **@xx.com -c 'this is content'
''

SERVER = 'smtp.gmail.com'
PORT = 587
USER = 'f7@gmail.com'
GF = '4@qq.com'
PASSWD = '03'

def send_email(receiver, msg):

   # print PASSWD, USER
    try:
        smtp = smtplib.SMTP(SERVER, PORT) 
        smtp.ehlo()
        smtp.starttls()
        smtp.login(USER, PASSWD)
        smtp.sendmail(USER, receiver, msg) 
        print 'Success!'
    except: 
        print 'failed to send'

def main():
    length = len(sys.argv)
#    print length
    if not (length == 6 or length == 4):
        print USAGE
        return 

    if sys.argv[1] == 'me':
        receiver = USER
    elif sys.argv[1] == 'love':
        receiver = GF
    else:
        receiver = sys.argv[1]

#    print receiver
    if length == 4 and sys.argv[2] == '-c':
        headers  = '\r\n'.join([
            'from: ' + USER,
            'subject: ' + 'An email from ' + USER,
            'to: ' + receiver,
            'mime-version: 1.0',
            'content-type: text/html'
            ])
        content = headers + '\r\n\r\n' + sys.argv[3] 
    if length == 6 and sys.argv[2] == '-t':
        headers  = '\r\n'.join([
            'from: ' + USER,
            'subject: ' + sys.argv[3],
            'to: ' + receiver,
            'mime-version: 1.0',
            'content-type: text/html'
            ])
        content = headers + '\r\n\r\n' + sys.argv[5] 
    try:
        send_email(receiver, content) 
    except:
        print 'failed to send email'
        print USAGE
if __name__ == '__main__':
    main()        

{% endhighlight %}


#END
