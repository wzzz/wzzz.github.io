在部署gcloud的app时，需要执行gcloud auth login，遇到以下一些问题：

1. redirect_url为localhost:8085, 由于当前xxnet的ui监控占用了这个端口，需要先改掉这个端口再重启

2. 认证成功后报错：
    httplib2.SSLHandshakeError: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:590) cacerts.txt
此处需要从浏览器的证书中导出GoAgent的pem文件，然后拷贝其中的内容追加到：
C:\Program Files (x86)\Google\Cloud SDK\google-cloud-sdk\lib\third_party\httplib2\cacerts.txt

