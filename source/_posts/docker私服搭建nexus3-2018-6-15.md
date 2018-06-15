---
title: docker私服搭建nexus3
date: 2018-06-15 17:53:47
categories:
 - nexus
tags:
 - nexus
 - docker
 - Docker
 - nexus3
---
docker私服搭建有官方的registry镜像，也有改版后的NexusOss3.x，因为maven的原因搭建了nexus，所以一并将docker私服也搭建到nexus上。

nexus的安装过程就单独说了，如果是2.x系列需要升级到2.14版本再升级到3.y系列，如果3.x到3.y直接升级就可以。
>从3.0版本开始，nexus不再只是一个maven仓库，还可以是docker、npm、bower的私有仓库。

## 配置SSL
docker的仓库链接是基于HTTPS的，故一般情况下需要将nexus的访问方式改为支持https。
配置SSL主要的难点在于证书，证书可以用公网证书，而一般情况下，私服部署在内网，没有域名，用内网IP访问，这种情况下用自签名的证书是最好的选择。证书生成工具用jdk自带的就可以。
配置过程,进入${nexus}/etc/ssl目录下面，执行命令过程中会输入多次密码，记下密码，后面有用处：
``` shell
$ keytool -genkeypair -keystore example.jks -storepass password -alias example.com \
>  -keyalg RSA -keysize 2048 -validity 5000 -keypass password \
>  -dname 'CN=*.example.com, OU=Sonatype, O=Sonatype, L=Unspecified, ST=Unspecified, C=US' \
>  -ext 'SAN=DNS:nexus.example.com,DNS:clm.example.com,DNS:repo.example.com,DNS:www.example.com'

Warning:
The JKS keystore uses a proprietary format. It is recommended to migrate to PKCS12 which is an industry standard format using "keytool -importkeystore -srckeystore example.jks -destkeystore example.jks -deststoretype pkcs12".
$ keytool -exportcert -keystore example.jks -alias example.com -rfc > example.cert
Enter keystore password:  password

Warning:
The JKS keystore uses a proprietary format. It is recommended to migrate to PKCS12 which is an industry standard format using "keytool -importkeystore -srckeystore example.jks -destkeystore example.jks -deststoretype pkcs12".
$ keytool -importkeystore -srckeystore example.jks -destkeystore example.p12 -deststoretype PKCS12
Importing keystore example.jks to example.p12...
Enter destination keystore password:  
Re-enter new password: 
Enter source keystore password:  
Entry for alias example.com successfully imported.
Import command completed:  1 entries successfully imported, 0 entries failed or cancelled
$ openssl pkcs12 -nocerts -nodes -in example.p12 -out example.key
Enter Import Password:
MAC verified OK

```

修改配置文件:etc/nexus.properties
``` 
原：
nexus-args=${jetty.etc}/jetty.xml,${jetty.etc}/jetty-http.xml,${jetty.etc}/jetty-requestlog.xml
修改后：
application-port-ssl=8433
nexus-args=${jetty.etc}/jetty.xml,${jetty.etc}/jetty-http.xml,${jetty.etc}/jetty-https.xml,${jetty.etc}/jetty-requestlog.xml
```
修改配置文件：etc/jetty/jetty-https.xml
``` xml
 <New id="sslContextFactory" class="org.eclipse.jetty.util.ssl.SslContextFactory">
    <Set name="KeyStorePath"><Property name="ssl.etc"/>/keystore.jks</Set> // 文件名和前面证书的一致
    <Set name="KeyStorePassword">password</Set> // 密码用前面的
    <Set name="KeyManagerPassword">password</Set>
    <Set name="TrustStorePath"><Property name="ssl.etc"/>/keystore.jks</Set>
    <Set name="TrustStorePassword">password</Set>
    <Set name="EndpointIdentificationAlgorithm"></Set>
    <Set name="NeedClientAuth"><Property name="jetty.ssl.needClientAuth" default="false"/></Set>
    <Set name="WantClientAuth"><Property name="jetty.ssl.wantClientAuth" default="false"/></Set>
    <Set name="ExcludeCipherSuites">
      <Array type="String">
        <Item>SSL_RSA_WITH_DES_CBC_SHA</Item>
        <Item>SSL_DHE_RSA_WITH_DES_CBC_SHA</Item>
        <Item>SSL_DHE_DSS_WITH_DES_CBC_SHA</Item>
        <Item>SSL_RSA_EXPORT_WITH_RC4_40_MD5</Item>
        <Item>SSL_RSA_EXPORT_WITH_DES40_CBC_SHA</Item>
        <Item>SSL_DHE_RSA_EXPORT_WITH_DES40_CBC_SHA</Item>
        <Item>SSL_DHE_DSS_EXPORT_WITH_DES40_CBC_SHA</Item>
      </Array>
    </Set>
  </New>

```
重启nexus，打开浏览器访问，https://ip:8443/ 会提示出来证书签名有问题，直接略过就行。

## 创建仓库(私有，代理，组合)
创建三个仓库，如果需要代理多个来源，可以创建多个代理库。
私有仓库：注意将http或者https的端口打开。

代理中央库的注意地址：
remote: https://registry-1.docker.io
docer index : use Docker hub

组合仓库将刚建立的Hosted和Proxy仓库都加入就可以了。

##　配置docker本地

本地配置最主要的是配置insecure-registries和docker　login。

### 配置daemon.json
/etc/docker/daemon.json
``` json
{
  "registry-mirrors": ["https://IP:18443"],
  "insecure-registries":["IP:18443","IP:18444"]
}
```
为什么需要两个insecure-registry呢？因为group仓库不可以作为push仓库，如果单纯的进行pull，可以只配置一个。
### docker login
分别login两个insecure-registry。
``` shell
$ docker login IP:18444
Username (admin): admin
Password: 
Login Succeeded
$ docker login IP:18443
Username (admin): admin
Password: 
Login Succeeded
```
这样可以愉快的进行push和pull操作了。

## push和pull的坑
nexus有一个设定，push只能对Hosted，pull从三种仓库都可以。因为push的操作给proxy的，是无法推送给被代理库的。group的仓库接受push后，无法确定是给proxy还是hosted。
但是nexus可以通过push给hosted的仓库，但是通过group仓库pull。
``` shell
$ docker images
REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE
alpine                               latest              3fd9065eaf02        5 months ago        4.15MB
IP:18443/alpine            latest              3fd9065eaf02        5 months ago        4.15MB
$ docker tag alpine IP:18444/alpine
$ docker push IP:18444/alpine
The push refers to repository [IP:18444/alpine]
cd7100a72410: Layer already exists 
latest: digest: sha256:8c03bb07a531c53ad7d0f6e7041b64d81f99c6e493cb39abba56d956b40eacbc size: 528
$ docker images
REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE
alpine                               latest              3fd9065eaf02        5 months ago        4.15MB
IP:18444/alpine            latest              3fd9065eaf02        5 months ago        4.15MB
$ docker pull IP:18443/alpine
Using default tag: latest
latest: Pulling from alpine
Digest: sha256:8c03bb07a531c53ad7d0f6e7041b64d81f99c6e493cb39abba56d956b40eacbc
Status: Image is up to date for IP:18443/alpine:latest
$ docker images
REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE
alpine                               latest              3fd9065eaf02        5 months ago        4.15MB
IP:18443/alpine            latest              3fd9065eaf02        5 months ago        4.15MB
IP:18444/alpine            latest              3fd9065eaf02        5 months ago        4.15MB
```
完成。
