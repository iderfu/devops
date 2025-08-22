---
date: 2020-03-20
category: 自动化工具
tag: Ansible
---

# 3.2 Ansible lineinfile模块详解

## 1 简介

之所以专门说一说这个模块，是因为lineinfile在实际使用中非常有用。

lineinfile模块用于在源文件中插入、删除、替换行，和sed命令的功能类似，也支持正则表达式匹配和替换。

实际上，在大多数时候，我们在linux上的操作，就是针对文件的操作，通过配置管理工具对配置文件作统一的配置修改是一个非常酷的功能。

下面是官方针对该模块的说明：

```
lineinfile - Ensure a particular line is in a file, or replace an existing line using a back-referenced regular expression
```

简单讲，这个模块就是针对一个文件中行内容的操作。

下面我们详细说一说其具体可以做的事情。

## 2 修改匹配行

下面是一个简单的task示例：

```yaml
# 将/etc/selinux/config中匹配到以'SELINUX='开头的行，将其替换为'SELINUX=disabled'
- name: modify selinux to disabled
  lineinfile:
    path: /etc/selinux/config
    regex: '^SELINUX='
    line: 'SELINUX=disabled'
```

## 3 在匹配行前或后添加内容

> insertbefore和insertafter指定的正则表达式如果匹配了多行，则默认选中最后一个匹配行，然后在被选中的行前、行后插入。如果明确要指定选中第一次匹配的行，则指定参数firstmatch=yes：

示例文件如下：

```
# cat /etc/http.conf

Listen 127.0.0.1:80
Listen 80
Port
```

### 3.1 在匹配行前添加

在http.conf文件的`Listen 80`前面添加一行`Listen 8080`，task示例如下：

```
- name: add line before Listen 80
  lineinfile:
    dest: /etc/http.conf
    insertbefore: '^Listen 80'
    line: 'Listen 8080'
```

### 3.2 在匹配行后添加

在http.conf文件的`Port`后面添加一行`just a test`，task示例如下：

```
- name: add line before Listen 80
  lineinfile:
    dest: /etc/http.conf
    insertafter: '^Port'
    line: 'just a test'
```

## 4 修改文件内容及权限

示例文件：

```sh
#cat /etc/hosts

127.0.0.1       localhost.localdomain localhost ::1       localhost6.localdomain6 localhost6
10.1.61.130     hub.dz11.com
```

修改/etc/hosts，将以`127.0.0.1`开头的行替换为`127.0.0.1 localhost`，并将/etc/hosts的属主和属组都修改为root，权限改为644，如下：

```
- name: modify hosts
  lineinfile:
    dest: /etc/hosts
    regex: '^127\.0\.0\.1'
    line: '127.0.0.1 localhost'
    owner: root
    group: root
    mode: 0644
```

## 5 删除行内容

regexp结合state=absent时，表示删除所有匹配的行。

示例原文件：

```sh
#cat /etc/hosts

127.0.0.1       localhost.localdomain localhost ::1       localhost6.localdomain6 localhost6
10.1.61.130     hub.dz11.com
```

删除以`10.1.61.130`开头的行：

```
- name: delete a line
  lineinfile:
    dest: /etc/hosts
    regex: '^10\.1\.61'
    state: absent
```

## 6 文件存在则添加一行内容

往/etc/hosts里添加一行`10.1.61.131 test.dz11.com`（多次执行，不会重复添加），示例如下：

如果再次执行，则不会再次追加此行。因为lineinfile模块的state参数默认值为present，它能保证幂等性，当要插入的行已经存在时则不会再插入。

```yaml
- name: add a line
  lineinfile:
    dest: /etc/hosts
    line: '10.1.61.131 test.dz11.com'
```

## 7 如果有匹配的行则修改该行，如果不匹配则添加

示例原文件/tmp/test.txt内容如下：

```sh
# %wheel   ALL=(ALL)   ALL
```

下面的示例task中，匹配以`%wheel`开头的行，匹配到，则执行替换，未匹配，则添加。因为原文件中，没有以`%wheel`开头的行，所以会添加一行：

```yaml
- name: add or modify a line
  lineinfile: 
    dest: /tmp/test.txt
    regex: '^%wheel'
    line: '%wheel  ALL=(ALL)       NOPASSWD: ALL'
```

修改后的文件如下：

```sh
#cat /tmp/text.txt

# %wheel   ALL=(ALL)   ALL
%wheel  ALL=(ALL)       NOPASSWD: ALL
```

## 8 参数backrefs，backup说明

- backup： 是否备份原文件，默认为no
- backrefs：
  - 当backrefs为no时，如果regex没有匹配到行，则添加一行，如果Regx匹配到行，则修改该行
  - 当backrefs为yes时，如果regex没有匹配到行，则保持原文件不变，如果regex匹配到行，则修改该行
  - backrefs默认为no，所以上面那个示例中，我们没有配置backrefs，而默认没有匹配，则修改。

下面我们看一看backrefs为yes时匹配到行的示例：

示例原文件：

```sh
# cat /tmp/testfile

# %wheel   ALL=(ALL)   ALL
%wheel  ALL=(ALL)       NOPASSWD: ALL
#?bar
```

task示例：

```
 - name: test backrefs
  lineinfile:
      backup: yes
      state: present
      dest: /tmp/testfile
      regexp: '^#\?bar'
      backrefs: yes
      line: 'bar'
```

修改后的文件：

```sh
# cat /tmp/testfile

# %wheel   ALL=(ALL)   ALL
%wheel  ALL=(ALL)       NOPASSWD: ALL
bar
```

## 9 使用validate验证文件是否正确修改

在一些场景下，我们修改完文件后，需要对文件做一下测试，用以检查文件修改之后，是否能正常运行。如http.conf、nginx.conf等，一旦改错，而不加以测试，可能会直接导致http服务挂掉。

可以使用validate关键字，在修改完成以后，对文件执行检测：

```
- name: test validate
  lineinfile:
      dest: /etc/sudoers
      state: present
      regexp: '^%ADMIN ALL='
      line: '%ADMIN ALL=(ALL)'
      validate: 'visudo -cf %s'
  tags:
    - testsudo
```

## 10 regexp和insertXXX结合

lineinfile最后一个比较常用的功能是regepx结合insertbefore或结合insertafter。这时候的行将根据insertXXX的位置来插入，而regexp参数则充当幂等性判断参数：只有regepx匹配失败时，insertXXX才会插入行。

例如：

```
- lineinfile:
path: "a.txt"
line: "hello line"
regexp: '^hello'
insertbefore: '^para.* 2'
```

这表示将"hello line"插入在paragraph 2行的前面，但如果再次执行，则不会再次插入，因为regexp参数指定的正则表达式已经能够已经存在的"hello line"行。所以，当regepx结合insertXXX使用时，regexp的参数通常都会设置为能够匹配插入之后的行的正则表达式，以便实现幂等性。

> 参考链接：
>
> https://www.cnblogs.com/breezey/p/9297252.html
>
> https://blog.51cto.com/cloumn/blog/1544
