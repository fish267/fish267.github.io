---
author: Fish
layout: post
title: 近期笔记
categories: work
tags: python
---

[网商工具平台][1]  将近期Coding时需要谷歌的点，记录一下，主要是 Python Flask Jquery 方面

## Python

### Python 正则

Python 自1.5版本起增加了re 模块，它提供 Perl 风格的正则表达式模式。

#### 正则表达式实例

1. 字符类

| 实例 |   描述 |
| --------   | -----:  | :----:  |
| [Pp]ython |   匹配 "Python" 或 "python"  |
| rub[ye] | 匹配 "ruby" 或 "rube"  |
| [aeiou] | 匹配中括号内的任意一个字母  |
| [0-9]   | 匹配任何数字。类似于 [0123456789]  |
| [a-z]   | 匹配任何小写字母  |
| [A-Z]   | 匹配任何大写字母  |
| [a-zA-Z0-9] |  匹配任何字母及数字  |
| [^aeiou]   |  除了aeiou字母以外的所有字符  |
| [^0-9] |  匹配除了数字外的字符  |

2. 特殊字符类

| 实例 |   描述 |
| --------   | -----:  | :----:  |
| .  |  匹配除 "\n" 之外的任何单个字符。要匹配包括 '\n' 在内的任何字符，请使用象 '[.\n]' 的模式 |
| \d |  匹配一个数字字符。等价于 [0-9] |
| \D |  匹配一个非数字字符。等价于 [^0-9] |
| \s |  匹配任何空白字符，包括空格、制表符、换页符等等。等价于 [ \f\n\r\t\v] |
| \S |  匹配任何非空白字符。等价于 [^ \f\n\r\t\v] |
| \w |  匹配包括下划线的任何单词字符。等价于'[A-Za-z0-9_]' |
| \W  匹配任何非单词字符。等价于 '[^A-Za-z0-9_]' |

#### re.match && re.search

    re.match(pattern, string, flags=0)

    re.search(pattern, string, flags=0)

<!--more-->
{% highlight python %}
import re

line = 'Doge is a friend of mine and we are good friends.'

dog = re.search('friend', line)
print(dog.group())

dogs = re.search(r'(.*) and (.*)', line)
print(dogs.groups())

cat = re.match('Doge', line)
print(cat.group())

cats = re.match(r'(.*) are (.*)', line)
print(cats.groups())

{% endhighlight %}

output:

    friend
    ('Doge is a friend of mine', 'we are good friends.')
    Doge
    ('Doge is a friend of mine and we', 'good friends.')

re.match只匹配字符串的开始，如果字符串开始不符合正则表达式，则匹配失败，函数返回None；而re.search匹配整个字符串，直到找到一个匹配。

### csv

    csv.reader(csvfile, dialect='excel', **fmtparams)

按列读取CSV文件，返回值当做一个大list

{% highlight python %}

>>> import csv, os
>>> os.chdir('/Users/fish/Git/test/')
>>> content = csv.reader(open('test.csv', encoding = 'utf-8'))
>>> for line in content:
...     print(line)
...
['GREAL', 'Great Lakes Food Market', 'Howard Snyder', 'Marketing Manager', '(503) 555-7555', '2732 Baker Blvd.', 'Eugene', 'OR', '97403', 'USA']
['HUNGC', 'Hungry Coyote Import Store', 'Yoshi Latimer', 'Sales Representative', '(503) 555-6874', 'City Center Plaza 516 Main St.', 'Elgin', 'OR', '97827', 'USA']
['LAZYK', 'Lazy K Kountry Store', 'John Steel', 'Marketing Manager', '(509) 555-7969', '12 Orchestra Terrace', 'Walla Walla', 'WA', '99362', 'USA']
['LETSS', "Let's Stop N Shop", 'Jaime Yorres', 'Owner', '(415) 555-5938', '87 Polk St. Suite 5', 'San Francisco', 'CA', '94117', 'USA']

{% endhighlight %}

### config

和csv不同，一般系统的配置文件就是 K-V 表，python 中用 configparser 模块来处理。
比如系统system.config 文件如下：

    [dev]
    traceid=sh /home/log/tools/traceidCheck/whereru.sh
    sql=sh /home/log/tools/traceidCheck/exeSql.sh
    traceidQuick=sh /home/log/tools/traceidCheck/whereru_fast.sh on

    [test]
    sit_status = cat /Users/fish/Git/test/sit.txt
    system_detail = cat /Users/fish/Git/test/dec.py

{% highlight python %}

class Config:
    def __init__(self, path):
        self.config = configparser.ConfigParser()
        self.config.read(path + '/config/command.config')

    def get_config(self, session, name):
        return self.config[session][name]
{% endhighlight %}

比如要获取 dev 配置下 sql 命令对应的 value, 用 config['dev']['sql']  来获取。


### Database sqlite3

python 自带了个 "DB", sqlite3 中的 DB 其实就是个支持 sql 命令的小文件。 和 PHP 等其他语言连接 DB 差不多，有个 "游标" 的概念。简单操作如下:

{% highlight python %}


class DB:
    def __init__(self, file):
        self.db= file

    def executeDB(self, sql):
        conn = sqlite3.connect(self.db)
        cur = conn.cursor()
        try:
            print('Executing command: ' + sql)
            cur.execute(sql)
            conn.commit()
            output = cur.fetchall()
            return output
        except:
            print('Error executing sql: %s' % sql)
{% endhighlight %}

## Bootstrap


## Jquery

### 元素选取：

1. 直接选取  <code>$('div')</code>
2. #id 选择 <code>$('#divid')</code>
3. .class 选择 <code>$('.active')</code>
4. 复杂的  <code>$("p.intro")</code>   <code>$("p:first")</code>
            <code> $('#serverTable tr');</code>
### 事件操作

用的最多的还是 <code>$('#id').click(...)</code>

{% highlight javascript %}

$("#executeButton").click(function () {
    $.post("/submitExecute", {cmd: $('#executeCmd').val()}, function (data) {
        $('#textOutput').text(data['contents']);
    })
});

{% endhighlight %}
常见 DOM 事件：


| 鼠标事件  |   键盘事件 |    表单事件 |    文档/窗口事件 |
| --------   | -----:  | ----:  | :------: |
| click  |  keypress |    submit |  load |
| dblclick |    keydown | change |  resize |
| mouseenter |  keyup   | focus  |  scroll |
| mouseleave |      blur |    |  unload |

### 元素绑定回车

{% highlight javascript %}

$('#executeCmd').bind('keyup', function (event) {
    if (event.keyCode == "13") {
        $('#executeButton').click();
    }
});

{% endhighlight %}

### Ajax

{% highlight javascript %}

$('#register').click(function () {
    $.post('/register', {username: $('#username').val(), password: $('#password').val()},
            function (data) {
                alert(data);
                window.location.reload();
            });
});

{% endhighlight %}

### 获取页面Cookie

{% highlight javascript %}

function getCookie(c_name) {
    if (document.cookie.length > 0) {
        c_start = document.cookie.indexOf(c_name + "=");
        if (c_start != -1) {
            c_start = c_start + c_name.length + 1;
            c_end = document.cookie.indexOf(";", c_start);
            if (c_end == -1) c_end = document.cookie.length;
            return unescape(document.cookie.substring(c_start, c_end))
        }
    }
    return ""
}
{% endhighlight %}

### 链路图 Gojs

### 画大饼图  Achart

### 其他

- 页面刷新   <code> window.location.reload()</code>
- 添加class <code>$('#id').addClass('active')
- 改变值    <code>$('#id').text('xxxx')
- <a>改变href   <code>$('#a').href('/')
- 显示、隐藏 <code>$('#id').hide() $('#id').show()</code>

##  Flask

### Jinja2

{{   }}  中用于获取 py 文件传过来的值，{{ data|length}} {{ data|tojson }}, 是比较常用的属性。其实前端界面比较繁琐，需要一步步调试出来。

{% highlight python %}

{% for button in button_details %}
    {% if  button|length  !=0 %}
        <div class="col-xs-6 col-sm-3 placeholder">
            <button class="btn btn-primary" id='{{ button[0] }}' style="background: #1EB0F2"
                    value='{{ button[1] }}'>{{ button[0] }}</button>
        </div>
    {% endif %}
{% endfor %}

{% endhighlight %}

### flask_login

账户系统，需要 <code>pip install flask_login</code> 来利用这个模块，非常方便。


{% highlight python %}
import flask.ext.login as flask_login

login_manager = flask_login.LoginManager()

login_manager.init_app(app)
app.secret_key = 'super secret string'

@app.route('/login', methods=['GET', 'POST'])
def login():
    users = getUsers(app.root_path + '/script/bank.db')
    if flask.request.method == 'GET':
        return render_template('login.html', flag='false')

    try:
        username = flask.request.form['username']
        if flask.request.form['password'] == users[username]:
            user = User()
            user.id = username
            flask_login.login_user(user, remember=True)
            return flask.redirect(flask.url_for('sql'))
        else:
            return render_template('login.html', flag='true')
    except:
        return render_template('login.html', flag='true')


class User(flask_login.UserMixin):
    pass


@login_manager.user_loader
def user_loader(username):
    if username not in users:
        return

    user = User()
    user.id = username
    return user


@login_manager.request_loader
def request_loader(request):
    username = request.form.get('username')
    if username not in users:
        return

    user = User()
    user.id = username
    # user.is_authenticated = request.form['password'] == users[username]

    return user


@login_manager.unauthorized_handler
def unauthorized_handler():
    return flask.redirect(flask.url_for('login'))

{% endhighlight %}


在需要账户登录的界面， 添加 <code>@flask_login.login_required</code>

{% highlight python %}


@app.route('/sql', methods=['get', 'post'])
@flask_login.login_required
def sql():
    if request.method == 'POST':
        traceid = request.form['traceid']
        username = request.form['username']
        gmt = time.ctime()
        # filter sql, delete/select is not allowed
        if 'delete' in traceid:
            output = {'contents': '只能执行update/select命令'}
            return jsonify(output)
        # get system command on file command.config
        config = Config(app.root_path)
        cmd = config.get_config('default', 'sql') + ' "' + traceid + '"'
        print(cmd)
        output = os.popen(cmd)
        # write sql to db
        db = DB(app.root_path + '/script/bank.db')
        db.executeDB('insert into sqlHistory values("%s", "%s", "%s")' % (username, traceid, gmt))
        output = {'contents': output.read()}
        return jsonify(output)
    else:
        db = DB(app.root_path + '/script/bank.db')
        sqlHistory = db.executeDB('select * from sqlHistory order by gmt_modified desc limit 50')
        output = ''
        for sql in sqlHistory:
            output = output + str(sql)[1:-1] + '\n'
        return render_template('sql.html', sqlHistory=output)

{% endhighlight %}

# 参考资料


  [Bootstrap][2]

  [Flask 文档][3]


  [BUI 官网][4]


  [Gojs 官网][5]


[1]: http://bktools.mayibank.net
[2]: http://www.runoob.com/bootstrap/bootstrap-tutorial.html
[3]: http://flask.pocoo.org
[4]: http://builive.com
[5]: http://www.nwoods.com
