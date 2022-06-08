# go 问题解决

## This download does NOT match the one reported by the checksum server

大致错误如下

![go-checksum](../images/go-checksum.png)

解决
```shell
$ go clean -modcache
$ go mod tidy
```


