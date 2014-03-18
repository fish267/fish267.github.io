---
layout: post
author: Fish
title: Linux下vim bash初始化脚本
categories: Python
tags: tools
---
每次换个操作环境, 都得按自己的习惯重新搭配搭配, 尤其是公司里, 这么多虚拟机, 我使用 vim 有点强迫症.

写了个 Python 小脚本, 针对 ubuntu 系统写的, 可以一次搞定 vim 配置和 bash 下 alias 的设定! 给自己赞一个~

主要是 vim 的括号匹配, tab 键设置, 自动对其, 还有 bash 下省事的操作等功能, 所以说, **世界是靠懒人推动的!**
<!--more-->
{% highlight python %}
#!/usr/bin/python
#coding: utf-8
#get_ready for my linux env
import os
import subprocess
import re
#get vim ready
#suggest you have already installed vim
global vim_backup
global bash_backup
# config .vimrc to suit needs
def config_vim(vim_file):
    #make a copy of that
    global vim_backup
    vim_backup = vim_file + '.backup'
    os.system('cp %s %s' %(vim_file, vim_backup))
    #if these are not in .vimrc, add it!
    add_commands = ['set smartcase', 'set mouse=a', 'set autoindent', 'set smartindent', \
'set nu', 'set ts=4', 'set shiftwidth=4', 'set softtabstop=4',\
'set expandtab', 'set smarttab']
 
    vim_content = open(vim_file).read()
    for command in add_commands:
        if command not in vim_content:
            os.system('echo >> %s "%s"' %(vim_file, command))
    #sure these blow are not in .vimrc, just add that
    map_brackets = '''
inoremap jk <ESC>:w
inoremap ( ()<ESC>i
inoremap ) <c-r>=ClosePair(')')<CR>
#inoremap { {<CR>}<ESC>kA<CR>
inoremap } <c-r>=ClosePair('}')<CR>
inoremap [ []<ESC>i
inoremap ] <c-r>=ClosePair(']')<CR>
function ClosePair(char)
    if getline('.')[col('.')-1 ]== a:char
        return '\<Right>'
    else
        return a:char

    endif
endf
vmap <C-X> :s/^/#/<CR>
vmap XX :s/^##*//<CR>
        '''
#    print map_brackets
    if 'inoremap' not in vim_content:
        os.system('echo >> %s "%s"'  %(vim_file, map_brackets))
config_vim('/home/fish/tmp')
# config .bashrc to suit needs
def config_bash(bash_file):
    global bash_backup
    #make a copy
    bash_backup = bash_file + '.backup'
    os.system('cp %s %s' %(bash_file, bash_backup))

    bash_commands = ['alias ll="ls -alF"', 'alias la="ls -A"', \
   'alias l="ls -CF"', 'alias ..="cd .."', 'alias cp="cp -v"', \
   'alias mv="mv -v"', 'alias py="python"', 'alias df="df -h"', \
   'alias gmt="git commit -m"', 'alias ga="git add ."', \
   'alias gp="git push"'] 
    #add these alias if not in bashrc
    bash_content = open(bash_file).read()
    for command in bash_commands:
        if command not in bash_content:
            os.system("echo >> %s '%s'" %(bash_file, command))
#config_bash('/home/fish/tmp')
def locate_file(filename):
    #this function is used to locate file

    #下面这句代码信息量很高, 使用重定向输出到stdout, 然后*.stdout.read() 获取
    all_file_location = subprocess.Popen('locate %s' %filename, stdout = subprocess.PIPE, \
    shell = True).stdout.read().split('\n')
    #print all_file_location
    for location in all_file_location:
        exact_location = re.search('^\/home.*%s$' %filename, location)
        if exact_location != None and os.path.isfile(exact_location.group(0)):
            return exact_location.group(0)
            break
    print 'can not locate the file %s' %filename
#main
if __name__ == '__main__':
    global vim_backup
    global bash_backup
    vim_location = locate_file('.vimrc')
    bash_location = locate_file('.bashrc')
    print '*' * 22
    print 'you are going to change these files:'
    print vim_location
    print bash_location
    print '*' * 22
    
    check = raw_input('Type yes to modify:')
    if check.upper() == 'YES' or check.upper() == 'Y':
        config_vim(vim_location)
        config_bash(bash_location)

        print 'The files have been modified, and their backup are: \n'
        print vim_backup 
        print bash_backup
        print 'The difference before and after modified are blow:\n'
        print subprocess.Popen('diff %s %s' %(vim_location, vim_backup), \
            stdout = subprocess.PIPE, shell = True).stdout.read()
        print subprocess.Popen('diff %s %s' %(bash_location, bash_backup), \
            stdout = subprocess.PIPE, shell = True).stdout.read()
{% endhighlight %}
