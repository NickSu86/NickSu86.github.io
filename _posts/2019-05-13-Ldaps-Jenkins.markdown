---
layout: post
title: ldap之tls 双向认证要我命
date: Monday, 13. May 2019 11:00PM 
---

先简单介绍一下问题背景，现公司的ldap不止是ldaps还要验证客户端，也就是说在客户端验证服务端的tls证书的同时，服务端也会要求验证客户端的证书，其实并不复杂，平时单向认证见得多了，只要客户端有服务端的ca证书，当拿到服务端自己的证书的时候，就会用ca证书去验证服务端是不是想连接的服务端，所以只要告诉客户端ca证书在哪里就可以。双向认证其实也不难，就加多了让客户端知道自己的证书和私钥在哪里就可以了。这样当服务端请求客户端的证书的时候，客户端知道自己的证书在哪里并且发给服务端之后能拿自己的私钥来解密服务端发过来的随机码。
1. ldap 启用 tls:

- 生成证书和私钥
- 修改 ldap 指定证书和私钥地址
```
dn: cn=config
changetype: modify
add: olcTLSCACertificateFile
olcTLSCACertificateFile: /etc/ldap/sasl2/ca.pem
-
replace: olcTLSCertificateFile
olcTLSCertificateFile: /etc/ldap/sasl2/ldap.pem
-
replace: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/ldap/sasl2/ldap-key.pem
```
- 要求ldap验证客户端证书
```
dn: cn=config
changetype: modify
replace: olcTLSVerifyClient
olcTLSVerifyClient: demand
```

2. 客户端配置ca和自己的证书私钥
java配置truststore和密码，并修改java启动参数
```
root@saltjenkins:~# grep -i keystore /etc/default/jenkins 
JAVA_ARGS="-Djava.awt.headless=true -Djavax.net.ssl.trustStorePassword=changeit -Djavax.net.ssl.trustStore=/var/lib/jenkins/cacerts -Djavax.net.ssl.keyStorePassword=123123 -Djavax.net.ssl.keyStore=/var/lib/jenkins/jenkins.p12"
```
trsutstore和keystore分别是存放ca证书和自己的证书和私钥，因为keystore不支持分别导入证书和私钥，因此需要把pem格式转换成pk12然后再导入
```
cat jenkins.pem jenkins-key.pem > jenkins-p12.txt
openssl pkcs12 -export -in jenkins-p12.txt -out jenkins.pkcs12 -name jenkins.home.kd -noiter -nomaciter
keytool -importkeystore -srckeystore /var/lib/jenkins/jenkins.p12 -destkeystore /var/lib/jenkins/cacerts -srcstoretype pkcs12 -deststoretype jks
```
上面三个命令，前两个命令是把pem格式的证书和私钥转换成pkcs12格式，第三个命令是把pkcs12导入keystore里，然后在java启动的时候指定这个keystore就可以了。
几个有用的命令：
`openssl verify -CAfile $cacert $cliencert`
使用ca证书验证
