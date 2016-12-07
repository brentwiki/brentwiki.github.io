本章将利用nginx的目录浏览功能(autoindex)来搭建下载服务器（Nginx不像apache默认是打开目录浏览功能，需要手动开启）

## 环境准备
* 安装Nginx(略)

## fancyindex美化index
* 进入Nginx编译目录
```shell
cd /usr/local/nginx-1.10.1/
```
* 下载fancyindex
```shell
wget https://github.com/aperezdc/ngx-fancyindex/archive/master.zip
```
* 解压
```shell
unzip master.zip
```
解压后的目录为：ngx-fancyindex-master

* 编译nginx
```shell
./configure  --with-http_stub_status_module  --with-http_ssl_module --with-stream --add-module=./ngx-fancyindex-master/
make && make install
```
## 配置
* 配置nginx代理
```shell
vi /usr/local/nginx/conf/http.d/soft.xxx.dev
```
粘贴并修改以下内容
```xml
server {
        listen          80;
        server_name     soft.xxx.dev;

        error_log   logs/soft.xxx.dev.error.log;

        keepalive_timeout    60;  

        fancyindex on;
        fancyindex_exact_size off;
        fancyindex_localtime on;
        fancyindex_footer /.footer.html;
        fancyindex_header /.header.html;

        root    /home/soft;
}
```
* 配置页头和页脚
```shell
mkdir /home/soft
cd /home/soft
vi .header.html
```
粘贴并修改以下内容
```html
<html xmlns="http://www.w3.org/1999/xhtml">
<head><meta http-equiv="content-type" content="text/html; charset=utf-8"/>
<style type="text/css" media="screen">
body,html {background:#fff;font-family: "Lucida Grande",Calibri,Arial;font-size: 13pt;color: #333;background: #f8f8f8;}
tr.e {background:#f4f4f4;}
th,td {padding:0.1em 0.5em;}
th {text-align:left;font-weight:bold;background:#eee;border-bottom:1px solid #aaa;}
#top1 {width:80%; font-size:28px; margin: 0 auto 5px auto;}
#top2 {width:80%; font-size:18px; margin: 0 auto 5px auto;}
#footer {width:80%;margin: 0 auto; padding: 10pt 0;font-size: 10pt;text-align:center;}
#footer a {font-size: 14px; font-weight: normal; text-decoration: underline;}
#list {border:1px solid #aaa;width:80%;margin: 0 auto;padding: 0;}
a {color: #b00;font-size: 11pt;font-weight: bold;text-decoration: none;}
a:hover {color: #000;}
#readme {padding:0;margin:1em 0;border:none;width:100%;}
</style>
<script type="text/javascript">// <![CDATA[function ngx_onload(){var f=document.getElementById('readme');if(!(f&&f.contentDocument))return;f.style.height=f.contentDocument.body.offsetHeight+'px';f.contentDocument.body.style.padding='0';f.contentDocument.body.style.margin='0';}// ]]></script>
<title>XXX研发软件下载中心</title>
</head>
<body onload="ngx_onload()">
<h1 id="top1">XXX研发软件下载中心</h1>
<h1 id="top2">Directory listing of
```
```shell
vi .footer.html
```
粘贴并修改以下内容

```html
<table id="footer" cellpadding="0" cellspacing="1">
<thead><tr><td colspan="3">本页面由XXX提供技术支持（有需要上传到上载目录，请联系XXX）</td></tr><thead>
</table></body></html>
```
## 重启nginx
```shell
/usr/local/nginx/sbin/nginx -s stop
/usr/local/nginx/sbin/nginx
```
注意：/usr/local/nginx/sbin/nginx -s reload 这样重启不好使，我在这里被坑了一把

## 效果预览
![image](https://cloud.githubusercontent.com/assets/10822807/17135256/5a90857a-5361-11e6-956b-70514cefd19a.png)

## 参数解释
* 参见源码https://github.com/aperezdc/ngx-fancyindex README
