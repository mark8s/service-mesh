# go 问题解决

## This download does NOT match the one reported by the checksum server

大致错误如下

![go-checksum](../images/go-checksum.png)

解决
```shell
$ go clean -modcache
$ go mod tidy
```

## 老项目如何改造成使用go mod 模式

```shell
# 1.在项目于根目录 使用 go mod init 项目名
go mod init github.com/kiali/kiali
# 2.删除项目中vendor目录
rm -rf vendor
# 3. 使用 go mod tidy
go mod tidy
```

## 私有库library无法下载下来
现象
```shell
solar-controller git:(v1.10.4) ✗ go get git.cloud2go.cn/solarmesh/api v1.10.3-0.20220318062648-d5203057ec07
go: git.cloud2go.cn/solarmesh/api@v1.10.3-0.20220318062648-d5203057ec07: unrecognized import path "git.cloud2go.cn/solarmesh/api": https fetch: Get "https://git.cloud2go.cn/solarmesh/api?go-get=1": x509: certificate signed by unknown authority
```
解决：
```shell
$ go env -w GOPRIVATE="git.cloud2go.cn"
$ go env -w GOINSECURE="git.cloud2go.cn"
```

ok.
```shell
➜  solar-controller git:(v1.10.4) ✗ go get git.cloud2go.cn/solarmesh/api@v1.10.3-0.20220318062648-d5203057ec07 
go: downloading git.cloud2go.cn/solarmesh/api v1.10.3-0.20220318062648-d5203057ec07
go: downloading github.com/grpc-ecosystem/grpc-gateway v1.15.0
go: downloading github.com/gogo/protobuf v1.3.1
go: downloading github.com/golang/mock v1.4.4
go: downloading github.com/mwitkow/go-proto-validators v0.3.2
go: downloading github.com/rakyll/statik v0.1.7
go: downloading github.com/golang/glog v0.0.0-20160126235308-23def4e6c14b
go: downloading github.com/golang/protobuf v1.4.2
go: downloading golang.org/x/xerrors v0.0.0-20200804184101-5ec99f83aff1
go: downloading github.com/ghodss/yaml v1.0.0
go: downloading google.golang.org/genproto v0.0.0-20200526211855-cb27e3aa2013
go: downloading golang.org/x/mod v0.4.2
go: downloading gopkg.in/yaml.v2 v2.4.0
```

