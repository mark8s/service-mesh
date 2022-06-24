# openssl

验证证书有效期
```shell
$ openssl x509 -in ca-test.yaml -noout -dates
notBefore=Jun  8 03:06:19 2022 GMT
notAfter=Jun  5 03:06:19 2032 GMT
```
- in: in代表输入文件
- dates: 表示显示时间，以上表示，证书有效期从 2022年6月8日 到 2032年6月5日 

还可以使用 -subject 显示证书的对象，如

```shell
$ openssl x509 -in ca-test.yaml -noout -subject
subject= /O=Istio/CN=Intermediate CA/L=mark
```



