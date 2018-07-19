---
layout: post
title: 2018-07-19-ansible-variables
date: 2018-07-19 17:57:22.000000000 +08:00
---

前段时间面试卡在了一道dict和list混合的变量问题下，记录如下，变量文件如下：

```yaml
users_with_items:
  - name: "alice"
    personal_directories:
      - "bob"
      - "carol"
      - "dan"
  - name: "bob"
    personal_directories:
      - "alice"
      - "carol"
      - "dan"

common_directories:
  - ".ssh"
  - "loops"
```

先分析一下这个变量文件，由两个list组成，而第一个list里面又包含了两个dictionary,dictionary里面又有一个list , 当时的变量虽然不是这样的，但是大致也是这种类型。首先因为这是一个变量文件，所以需要使用到 var_files 来引用这个变量文件。如果使用 with_items 的话，那只能遍历一个 list ，如下所示：

```yaml
---
- hosts: localhost
  gather_facts: no

  vars_files:
    - /home/nick/ansible/newvars.yml

  tasks:
    - name: "test users_with_items result"
      debug: msg="Hello {{ item.name }} with {{ item.personal_directories}}"
      with_items:
        - "{{ users_with_items }}"
    - name: "test common_directories"
      debug: msg="This is common_directories {{ item }}"
      with_items:
        - "{{ common_directories }}"
```

这段 playbook 遍历变量文件里的两个 list , 因为上面分析过第一个 list 里面包含两个 dictionary , 而 dictionary 是由 key 和value 组成的。因此第一个 task 会打印出来名字和目录 list ，结果如下：

```
PLAY [localhost] ***************************************************************************************************************************

TASK [test users_with_items result] ********************************************************************************************************
ok: [localhost] => (item={u'personal_directories': [u'bob',u'carol', u'dan'], u'name': u'alice'}) => {
    "msg": "Hello alice with [u'bob', u'carol', u'dan']"
}
ok: [localhost] => (item={u'personal_directories': [u'alice',u'carol', u'dan'], u'name': u'bob'}) => {
    "msg": "Hello bob with [u'alice', u'carol', u'dan']"
}

TASK [test common_directories] *************************************************************************************************************
ok: [localhost] => (item=.ssh) => {
    "msg": "This is common_directories .ssh"
}
ok: [localhost] => (item=loops) => {
    "msg": "This is common_directories loops"
}
```

  这个示例是针对读取一个变量文件的，ansible 也支持直接在 playbook 里面定义循环 list ，比如：

```yaml
    - name: "pre-define list variables"
      debug: msg="test {{ item }}"
      with_items:
        - a
        - b
        - c
      tags:
        - list-var
```

执行结果如下：

```
PLAY [localhost] ***************************************************************************************************************************

TASK [pre-define list variables] ***********************************************************************************************************
ok: [localhost] => (item=a) => {
    "msg": "test a"
}
ok: [localhost] => (item=b) => {
    "msg": "test b"
}
ok: [localhost] => (item=c) => {
    "msg": "test c"
}
```

注意，YAML格式里面的 list 一定前面要有 - 这个符号，如果没有这个符合，只是当作空格隔开的几个字符而已。

然后其实上面的例子只是取出了 list 里面的 dictionary , 但是 dictionary 里面的 list 怎么办？先看看我们这个变量文件里面， dictionary 嵌套 list 是在 key 为 personal_directories 这里，所以我们可以使用 with_subelements 来读取嵌套的内容：

```yaml
    - name: "test with_subelements"
      debug: msg="Hello {{item.0.name}} with {{ item.1 }}"
      with_subelements:
        - "{{users_with_items}}"
        - personal_directories
      tags:
        - "subelements"
```

看看 with_subelements 里面，其实是一个 list ，这个 list 第一个内容是这个大的 list ，也就是包含我们两个 dictionary 的这个 list ，然后第二个是我们的子 list 的在这个 dictionary 里的 key , 看看执行结果：

```
PLAY [localhost] ***************************************************************************************************************************

TASK [test with_subelements] ***************************************************************************************************************
ok: [localhost] => (item=[{u'name': u'alice'}, u'bob']) => {
    "msg": "Hello alice with bob"
}
ok: [localhost] => (item=[{u'name': u'alice'}, u'carol']) => {
    "msg": "Hello alice with carol"
}
ok: [localhost] => (item=[{u'name': u'alice'}, u'dan']) => {
    "msg": "Hello alice with dan"
}
ok: [localhost] => (item=[{u'name': u'bob'}, u'alice']) => {
    "msg": "Hello bob with alice"
}
ok: [localhost] => (item=[{u'name': u'bob'}, u'carol']) => {
    "msg": "Hello bob with carol"
}
ok: [localhost] => (item=[{u'name': u'bob'}, u'dan']) => {
    "msg": "Hello bob with dan"
}
```

改天再拓展多级嵌套怎么解决。