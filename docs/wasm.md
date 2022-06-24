# wasm

`WebAssembly`，我们都知道，是一种新的字节码格式，目前被应用于 web 中，由于其可移植、体积小，安全性的等优点被渐渐广泛认可，但是其主要是运行在浏览器中。

一些天才们，想让 `WebAssembly` 也可以运行在非浏览器环境中，这就产生了 `WASI`。

因此，这样会有一些问题，需要解决之。我们知道，一些语言，比如 rust 都有自己的标准库，标准库对系统调用进行了封装，让我们写的代码可以跨平台，这实际上是源代码的可移植性。

`wasi` 这里需要可移植的二进制文件（`.wasm`）和一个跨平台的 runtime，也就是说，我们在某一个平台上生成了`.wasm`，直接拿到其他平台上，也可以直接使用。

## 编写wasm
参考 [proxy-wasm-go-sdk](https://github.com/mark8s/proxy-wasm-go-sdk)

### 环境搭建

#### Requirements

- [Go](https://go.dev/dl/) 1.17 or higher.
- [TinyGo](https://tinygo.org/) - This SDK depends on TinyGo and leverages its [WASI](https://github.com/WebAssembly/WASI) (WebAssembly System Interface) target. Please follow the official instruction [here](https://tinygo.org/getting-started/) for installing TinyGo.
- [Envoy](https://www.envoyproxy.io) - To run compiled examples, you need to have Envoy binary. We recommend using [func-e](https://func-e.io) as the easiest way to get started with Envoy. Alternatively, you can follow [the official instruction](https://www.envoyproxy.io/docs/envoy/latest/start/install).

#### Installation

Install `go1.17` or higher
```yaml
tar -C /usr/local/ -xzf go1.17.1.linux-amd64.tar.gz
 
vim /etc/profile.d/go.sh
### 加入以下内容
 
export GOROOT=/usr/local/go
export GOPATH=/data/go
export PATH=$PATH:$GOROOT/bin:$GOPATH
export GO111MODULE="on" # 开启 Go moudles 特性
export GOPROXY=https://goproxy.cn,direct # 安装 Go 模块时，国内代理服务器设置
 
# 让配置生效
source /etc/profile

```
```shell
$ go version
go version go1.17.1 linux/amd64
```

Ubuntu install `tinygo`
```shell
wget https://github.com/tinygo-org/tinygo/releases/download/v0.21.0/tinygo_0.21.0_amd64.deb
sudo dpkg -i tinygo_0.21.0_amd64.deb

export PATH=$PATH:/usr/local/bin
``` 
```shell
$ tinygo version
tinygo version 0.21.0 linux/amd64 (using go version go1.17.1 and LLVM version 11.0.0)
```

Install ``envoy``
```yaml
sudo apt update
sudo apt install apt-transport-https gnupg2 curl lsb-release
curl -sL 'https://deb.dl.getenvoy.io/public/gpg.8115BA8E629CC074.key' | sudo gpg --dearmor -o /usr/share/keyrings/getenvoy-keyring.gpg
Verify the keyring - this should yield "OK"
echo a077cb587a1b622e03aa4bf2f3689de14658a9497a9af2c427bba5f4cc3c4723 /usr/share/keyrings/getenvoy-keyring.gpg | sha256sum --check
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/getenvoy-keyring.gpg] https://deb.dl.getenvoy.io/public/deb/ubuntu $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/getenvoy.list
sudo apt update
sudo apt install -y getenvoy-envoy
```

#### Run Examples
```shell
### 编译得到wasm文件
tinygo build -o main.wasm -scheduler=none -target=wasi main.go
### 运行envoy
envoy -c envoy.yaml
### 访问envoy expose listener
curl -v localhost:port ## example/helloworld curl -v localhost:18000
```
## 使用wasm
- EnvoyFilter
- WasmPlugins

## istio-proxy中输出的wasm日志样例
```shell
2022-04-24T07:59:01.094651Z	debug	envoy http	[C44][S4314132064810061585] request headers complete (end_stream=true):
':authority', 'mall-admin:8080'
':path', '/order/12'
':method', 'GET'
'accept', 'application/json, text/plain, */*'
'authorization', 'Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJtYXJrcyIsInNjb3BlIjpbImFsbCJdLCJpZCI6MTAsImV4cCI6MTY1MDg3MTMyNywiYXV0aG9yaXRpZXMiOlsiNV_otoXnuqfnrqHnkIblkZgiXSwianRpIjoiN2RjZTRlMTQtMGRhMS00OGMxLWFmY2YtODYwMjg3N2I4YjczIiwiY2xpZW50X2lkIjoiYWRtaW4tYXBwIn0.YvbAcorKuSfLA8XeTdT74rSndmjim1dX6E0SncgXtt0L95ZQvOtWIO9SpG-vTA92UKC4EU80E-LU7w6CeOHmV_V5i_4BDUFtAaL2GvzFO362UPa1_ofzEHASqU-hx-Rz8CNqhuExmxcXMla_5U2IF6jRR96a2DUn1w_bJZl6Jtw'
'user-agent', 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/100.0.4896.127 Safari/537.36'
'origin', 'http://localhost:8090'
'referer', 'http://localhost:8090/'
'accept-encoding', 'gzip, deflate'
'accept-language', 'zh-CN,zh;q=0.9'
'x-request-id', 'a9699a23-4518-99f4-b0ff-db85295ce1e3'
'user', '{"user_name":"marks","scope":["all"],"id":10,"exp":1650871327,"authorities":["5_?????"],"jti":"7dce4e14-0da1-48c1-afcf-8602877b8b73","client_id":"admin-app"}'
'forwarded', 'proto=http;host="10.10.13.48:30201";for="127.0.0.6:36795"'
'x-forwarded-for', '127.0.0.6'
'x-forwarded-proto', 'http,http'
'x-forwarded-prefix', '/mall-admin'
'x-forwarded-port', '30201'
'x-forwarded-host', '10.10.13.48:30201'
'content-length', '0'
'x-envoy-decorator-operation', 'mall-admin.mall.svc.cluster.local:8080/*'
'x-envoy-peer-metadata', 'CiAKDkFQUF9DT05UQUlORVJTEg4aDG1hbGwtZ2F0ZXdheQoaCgpDTFVTVEVSX0lEEgwaCkt1YmVybmV0ZXMKGQoNSVNUSU9fVkVSU0lPThIIGgYxLjEzLjAK3wEKBkxBQkVMUxLUASrRAQoVCgNhcHASDhoMbWFsbC1nYXRld2F5CiEKEXBvZC10ZW1wbGF0ZS1oYXNoEgwaCjdkYzY0Zjk2YmIKJAoZc2VjdXJpdHkuaXN0aW8uaW8vdGxzTW9kZRIHGgVpc3RpbwoxCh9zZXJ2aWNlLmlzdGlvLmlvL2Nhbm9uaWNhbC1uYW1lEg4aDG1hbGwtZ2F0ZXdheQorCiNzZXJ2aWNlLmlzdGlvLmlvL2Nhbm9uaWNhbC1yZXZpc2lvbhIEGgJ2MQoPCgd2ZXJzaW9uEgQaAnYxChoKB01FU0hfSUQSDxoNY2x1c3Rlci5sb2NhbAonCgROQU1FEh8aHW1hbGwtZ2F0ZXdheS03ZGM2NGY5NmJiLXc3Mjc4ChMKCU5BTUVTUEFDRRIGGgRtYWxsChcKEVBMQVRGT1JNX01FVEFEQVRBEgIqAA=='
'x-envoy-peer-metadata-id', 'sidecar~10.44.0.3~mall-gateway-7dc64f96bb-w7278.mall~mall.svc.cluster.local'
'x-envoy-attempt-count', '1'
'x-b3-traceid', '2f528dc824864e870b0d0595c69a21df'
'x-b3-spanid', '90ff7f7650da40ca'
'x-b3-parentspanid', '0b0d0595c69a21df'
'x-b3-sampled', '1'

2022-04-24T07:59:01.094661Z	debug	envoy http	[C44][S4314132064810061585] request end stream
2022-04-24T07:59:01.094754Z	trace	envoy wasm	[host->vm] proxy_on_context_create(13, 2)
2022-04-24T07:59:01.094772Z	trace	envoy wasm	[vm->host] env.proxy_log(2, 81444, 35)
2022-04-24T07:59:01.094777Z	info	envoy wasm	wasm log clean-mall-admin: <---- pluginCx NewHttpContext ---->
2022-04-24T07:59:01.094779Z	trace	envoy wasm	[vm<-host] env.proxy_log return: 0
2022-04-24T07:59:01.094783Z	trace	envoy wasm	[host<-vm] proxy_on_context_create return: void
2022-04-24T07:59:01.094786Z	trace	envoy wasm	[host->vm] proxy_on_request_headers(13, 28, 1)
2022-04-24T07:59:01.094790Z	trace	envoy wasm	[host<-vm] proxy_on_request_headers return: 0
2022-04-24T07:59:01.094792Z	trace	envoy http	[C44][S4314132064810061585] decode headers called: filter=0x5629cf509730 status=0
2022-04-24T07:59:01.094818Z	trace	envoy http	[C44][S4314132064810061585] decode headers called: filter=0x5629cfd2ce70 status=0
2022-04-24T07:59:01.094833Z	trace	envoy http	[C44][S4314132064810061585] decode headers called: filter=0x5629cf05c3f0 status=0
2022-04-24T07:59:01.094836Z	trace	envoy http	[C44][S4314132064810061585] decode headers called: filter=0x5629cf508d20 status=0
2022-04-24T07:59:01.094842Z	trace	envoy http	[C44][S4314132064810061585] decode headers called: filter=0x5629cf508930 status=0
2022-04-24T07:59:01.094851Z	debug	envoy router	[C44][S4314132064810061585] cluster 'inbound|8080||' match for URL '/order/12'
2022-04-24T07:59:01.094863Z	debug	envoy upstream	Using existing host 10.36.0.11:8080.
2022-04-24T07:59:01.094893Z	debug	envoy router	[C44][S4314132064810061585] router decoding headers:
':authority', 'mall-admin:8080'
':path', '/order/12'
':method', 'GET'
':scheme', 'https'
'accept', 'application/json, text/plain, */*'
'authorization', 'Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJtYXJrcyIsInNjb3BlIjpbImFsbCJdLCJpZCI6MTAsImV4cCI6MTY1MDg3MTMyNywiYXV0aG9yaXRpZXMiOlsiNV_otoXnuqfnrqHnkIblkZgiXSwianRpIjoiN2RjZTRlMTQtMGRhMS00OGMxLWFmY2YtODYwMjg3N2I4YjczIiwiY2xpZW50X2lkIjoiYWRtaW4tYXBwIn0.YvbAcorKuSfLA8XeTdT74rSndmjim1dX6E0SncgXtt0L95ZQvOtWIO9SpG-vTA92UKC4EU80E-LU7w6CeOHmV_V5i_4BDUFtAaL2GvzFO362UPa1_ofzEHASqU-hx-Rz8CNqhuExmxcXMla_5U2IF6jRR96a2DUn1w_bJZl6Jtw'
'user-agent', 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/100.0.4896.127 Safari/537.36'
'origin', 'http://localhost:8090'
'referer', 'http://localhost:8090/'
'accept-encoding', 'gzip, deflate'
'accept-language', 'zh-CN,zh;q=0.9'
'x-request-id', 'a9699a23-4518-99f4-b0ff-db85295ce1e3'
'user', '{"user_name":"marks","scope":["all"],"id":10,"exp":1650871327,"authorities":["5_?????"],"jti":"7dce4e14-0da1-48c1-afcf-8602877b8b73","client_id":"admin-app"}'
'forwarded', 'proto=http;host="10.10.13.48:30201";for="127.0.0.6:36795"'
'x-forwarded-for', '127.0.0.6'
'x-forwarded-proto', 'http,http'
'x-forwarded-prefix', '/mall-admin'
'x-forwarded-port', '30201'
'x-forwarded-host', '10.10.13.48:30201'
'content-length', '0'
'x-envoy-attempt-count', '1'
'x-forwarded-client-cert', 'By=spiffe://cluster.local/ns/mall/sa/default;Hash=28e164521389dd25c0f21d777b5b6004b98bcdfea1d75fbeb49d5507ccacd7bc;Subject="";URI=spiffe://cluster.local/ns/mall/sa/default'
'x-b3-traceid', '2f528dc824864e870b0d0595c69a21df'
'x-b3-spanid', '0e41801e8d0be68f'
'x-b3-parentspanid', '90ff7f7650da40ca'
'x-b3-sampled', '1'

2022-04-24T07:59:01.094904Z	debug	envoy pool	[C1160] using existing connection
2022-04-24T07:59:01.094907Z	debug	envoy pool	[C1160] creating stream
2022-04-24T07:59:01.094915Z	debug	envoy router	[C44][S4314132064810061585] pool ready
2022-04-24T07:59:01.094937Z	trace	envoy connection	[C1160] writing 1623 bytes, end_stream false
2022-04-24T07:59:01.094949Z	trace	envoy pool	not creating a new connection, shouldCreateNewConnection returned false.
2022-04-24T07:59:01.094952Z	trace	envoy http	[C44][S4314132064810061585] decode headers called: filter=0x5629cfd2dc00 status=1
2022-04-24T07:59:01.094956Z	trace	envoy http	[C44] parsed 2200 bytes
2022-04-24T07:59:01.094964Z	trace	envoy connection	[C44] socket event: 2
2022-04-24T07:59:01.094966Z	trace	envoy connection	[C44] write ready
2022-04-24T07:59:01.094970Z	trace	envoy connection	[C1160] socket event: 2
2022-04-24T07:59:01.094971Z	trace	envoy connection	[C1160] write ready
2022-04-24T07:59:01.095006Z	trace	envoy connection	[C1160] write returns: 1623
2022-04-24T07:59:01.099050Z	trace	envoy connection	[C1161] socket event: 3
2022-04-24T07:59:01.099065Z	trace	envoy connection	[C1161] write ready
2022-04-24T07:59:01.099069Z	trace	envoy connection	[C1161] read ready. dispatch_buffered_data=false
2022-04-24T07:59:01.099080Z	trace	envoy connection	[C1161] read returns: 965
2022-04-24T07:59:01.099088Z	trace	envoy connection	[C1161] read error: Resource temporarily unavailable
2022-04-24T07:59:01.099095Z	trace	envoy filter	[C1161] downstream connection received 965 bytes, end_stream=false
2022-04-24T07:59:01.099100Z	trace	envoy connection	[C1162] writing 965 bytes, end_stream false
2022-04-24T07:59:01.099109Z	trace	envoy connection	[C1162] socket event: 2
2022-04-24T07:59:01.099113Z	trace	envoy connection	[C1162] write ready
2022-04-24T07:59:01.099155Z	trace	envoy connection	[C1162] write returns: 965
2022-04-24T07:59:01.100354Z	trace	envoy connection	[C1162] socket event: 3
2022-04-24T07:59:01.100363Z	trace	envoy connection	[C1162] write ready
2022-04-24T07:59:01.100367Z	trace	envoy connection	[C1162] read ready. dispatch_buffered_data=false
2022-04-24T07:59:01.100379Z	trace	envoy connection	[C1162] read returns: 13637
2022-04-24T07:59:01.100386Z	trace	envoy connection	[C1162] read error: Resource temporarily unavailable
2022-04-24T07:59:01.100391Z	trace	envoy filter	[C1161] upstream connection received 13637 bytes, end_stream=false
2022-04-24T07:59:01.100396Z	trace	envoy connection	[C1161] writing 13637 bytes, end_stream false
2022-04-24T07:59:01.100403Z	trace	envoy connection	[C1162] socket event: 2
2022-04-24T07:59:01.100404Z	trace	envoy connection	[C1162] write ready
2022-04-24T07:59:01.100407Z	trace	envoy connection	[C1161] socket event: 2
2022-04-24T07:59:01.100409Z	trace	envoy connection	[C1161] write ready
2022-04-24T07:59:01.100440Z	trace	envoy connection	[C1161] write returns: 13637
2022-04-24T07:59:01.124546Z	trace	envoy connection	[C1160] socket event: 3
2022-04-24T07:59:01.124563Z	trace	envoy connection	[C1160] write ready
2022-04-24T07:59:01.124569Z	trace	envoy connection	[C1160] read ready. dispatch_buffered_data=false
2022-04-24T07:59:01.124584Z	trace	envoy connection	[C1160] read returns: 4690
2022-04-24T07:59:01.124594Z	trace	envoy connection	[C1160] read error: Resource temporarily unavailable
2022-04-24T07:59:01.124601Z	trace	envoy http	[C1160] parsing 4690 bytes
2022-04-24T07:59:01.124606Z	trace	envoy http	[C1160] message begin
2022-04-24T07:59:01.124621Z	trace	envoy http	[C1160] completed header: key=Content-Type value=application/json
2022-04-24T07:59:01.124628Z	trace	envoy http	[C1160] completed header: key=Transfer-Encoding value=chunked
2022-04-24T07:59:01.124633Z	trace	envoy http	[C1160] onHeadersCompleteBase
2022-04-24T07:59:01.124635Z	trace	envoy http	[C1160] completed header: key=Date value=Sun, 24 Apr 2022 07:59:01 GMT
2022-04-24T07:59:01.124641Z	trace	envoy http	[C1160] status_code 200
2022-04-24T07:59:01.124643Z	trace	envoy http	[C1160] Client: onHeadersComplete size=3
2022-04-24T07:59:01.124656Z	debug	envoy router	[C44][S4314132064810061585] upstream headers complete: end_stream=false
2022-04-24T07:59:01.124682Z	trace	envoy http	[C44][S4314132064810061585] encode headers called: filter=0x5629cf05c850 status=0
2022-04-24T07:59:01.124687Z	trace	envoy http	[C44][S4314132064810061585] encode headers called: filter=0x5629cf509b20 status=0
2022-04-24T07:59:01.124689Z	trace	envoy http	[C44][S4314132064810061585] encode headers called: filter=0x5629cf05cd90 status=0
2022-04-24T07:59:01.124703Z	trace	envoy http	[C44][S4314132064810061585] encode headers called: filter=0x5629cf508b60 status=0
2022-04-24T07:59:01.124708Z	trace	envoy wasm	[host->vm] proxy_on_response_headers(13, 7, 0)
2022-04-24T07:59:01.124718Z	trace	envoy wasm	[host<-vm] proxy_on_response_headers return: 0
2022-04-24T07:59:01.124720Z	trace	envoy http	[C44][S4314132064810061585] encode headers called: filter=0x5629cf41be30 status=0
2022-04-24T07:59:01.124737Z	debug	envoy http	[C44][S4314132064810061585] encoding headers via codec (end_stream=false):
':status', '200'
'content-type', 'application/json'
'date', 'Sun, 24 Apr 2022 07:59:01 GMT'
'x-envoy-upstream-service-time', '29'
'x-envoy-peer-metadata', 'Ch4KDkFQUF9DT05UQUlORVJTEgwaCm1hbGwtYWRtaW4KGgoKQ0xVU1RFUl9JRBIMGgpLdWJlcm5ldGVzChkKDUlTVElPX1ZFUlNJT04SCBoGMS4xMy4wCtsBCgZMQUJFTFMS0AEqzQEKEwoDYXBwEgwaCm1hbGwtYWRtaW4KIQoRcG9kLXRlbXBsYXRlLWhhc2gSDBoKNTQ5OGY0ZmI3OQokChlzZWN1cml0eS5pc3Rpby5pby90bHNNb2RlEgcaBWlzdGlvCi8KH3NlcnZpY2UuaXN0aW8uaW8vY2Fub25pY2FsLW5hbWUSDBoKbWFsbC1hZG1pbgorCiNzZXJ2aWNlLmlzdGlvLmlvL2Nhbm9uaWNhbC1yZXZpc2lvbhIEGgJ2MQoPCgd2ZXJzaW9uEgQaAnYxChoKB01FU0hfSUQSDxoNY2x1c3Rlci5sb2NhbAolCgROQU1FEh0aG21hbGwtYWRtaW4tNTQ5OGY0ZmI3OS1qY2dxcQoTCglOQU1FU1BBQ0USBhoEbWFsbAoXChFQTEFURk9STV9NRVRBREFUQRICKgA='
'x-envoy-peer-metadata-id', 'sidecar~10.36.0.11~mall-admin-5498f4fb79-jcgqq.mall~mall.svc.cluster.local'
'server', 'istio-envoy'

2022-04-24T07:59:01.124749Z	trace	envoy connection	[C44] writing 863 bytes, end_stream false
2022-04-24T07:59:01.124760Z	trace	envoy http	[C44][S4314132064810061585] encode data called: filter=0x5629cf05c850 status=0
2022-04-24T07:59:01.124762Z	trace	envoy http	[C44][S4314132064810061585] encode data called: filter=0x5629cf509b20 status=0
2022-04-24T07:59:01.124764Z	trace	envoy http	[C44][S4314132064810061585] encode data called: filter=0x5629cf05cd90 status=0
2022-04-24T07:59:01.124766Z	trace	envoy http	[C44][S4314132064810061585] encode data called: filter=0x5629cf508b60 status=0
2022-04-24T07:59:01.124769Z	trace	envoy wasm	[host->vm] proxy_on_response_body(13, 4568, 0)
2022-04-24T07:59:01.124780Z	trace	envoy wasm	[vm->host] env.proxy_log(2, 92592, 30)
2022-04-24T07:59:01.124785Z	info	envoy wasm	wasm log clean-mall-admin: endOfStream: %!(EXTRA T=false)
2022-04-24T07:59:01.124787Z	trace	envoy wasm	[vm<-host] env.proxy_log return: 0
2022-04-24T07:59:01.124791Z	trace	envoy wasm	[vm->host] env.proxy_get_buffer_bytes(1, 4568, 1000, 65208, 65160)
2022-04-24T07:59:01.124794Z	trace	envoy wasm	[vm<-host] env.proxy_get_buffer_bytes return: 0
2022-04-24T07:59:01.124797Z	trace	envoy wasm	[host<-vm] proxy_on_response_body return: 0
2022-04-24T07:59:01.124799Z	trace	envoy http	[C44][S4314132064810061585] encode data called: filter=0x5629cf41be30 status=0
2022-04-24T07:59:01.124802Z	trace	envoy http	[C44][S4314132064810061585] encoding data via codec (size=4568 end_stream=false)
2022-04-24T07:59:01.124806Z	trace	envoy connection	[C44] writing 4576 bytes, end_stream false
2022-04-24T07:59:01.124809Z	trace	envoy http	[C1160] parsed 4690 bytes
2022-04-24T07:59:01.124815Z	trace	envoy connection	[C44] socket event: 2
2022-04-24T07:59:01.124817Z	trace	envoy connection	[C44] write ready
2022-04-24T07:59:01.124919Z	trace	envoy connection	[C44] ssl write returns: 5439
2022-04-24T07:59:01.124928Z	trace	envoy connection	[C1160] socket event: 3
2022-04-24T07:59:01.124930Z	trace	envoy connection	[C1160] write ready
2022-04-24T07:59:01.124932Z	trace	envoy connection	[C1160] read ready. dispatch_buffered_data=false
2022-04-24T07:59:01.124938Z	trace	envoy connection	[C1160] read returns: 5
2022-04-24T07:59:01.124943Z	trace	envoy connection	[C1160] read error: Resource temporarily unavailable
2022-04-24T07:59:01.124946Z	trace	envoy http	[C1160] parsing 5 bytes
2022-04-24T07:59:01.124949Z	trace	envoy http	[C1160] message complete
2022-04-24T07:59:01.124951Z	trace	envoy http	[C1160] message complete
2022-04-24T07:59:01.124954Z	debug	envoy client	[C1160] response complete
2022-04-24T07:59:01.124957Z	trace	envoy main	item added to deferred deletion list (size=1)
2022-04-24T07:59:01.124971Z	trace	envoy http	[C44][S4314132064810061585] encode data called: filter=0x5629cf05c850 status=0
2022-04-24T07:59:01.124973Z	trace	envoy http	[C44][S4314132064810061585] encode data called: filter=0x5629cf509b20 status=0
2022-04-24T07:59:01.124975Z	trace	envoy http	[C44][S4314132064810061585] encode data called: filter=0x5629cf05cd90 status=0
2022-04-24T07:59:01.124977Z	trace	envoy http	[C44][S4314132064810061585] encode data called: filter=0x5629cf508b60 status=0
2022-04-24T07:59:01.124980Z	trace	envoy wasm	[host->vm] proxy_on_response_body(13, 0, 1)
2022-04-24T07:59:01.124986Z	trace	envoy wasm	[vm->host] env.proxy_log(2, 92816, 29)
2022-04-24T07:59:01.124989Z	info	envoy wasm	wasm log clean-mall-admin: endOfStream: %!(EXTRA T=true)
2022-04-24T07:59:01.124991Z	trace	envoy wasm	[vm<-host] env.proxy_log return: 0
2022-04-24T07:59:01.124993Z	trace	envoy wasm	[vm->host] env.proxy_get_buffer_bytes(1, 0, 1000, 65208, 65160)
2022-04-24T07:59:01.124995Z	trace	envoy wasm	[vm<-host] env.proxy_get_buffer_bytes return: 0
2022-04-24T07:59:01.124998Z	trace	envoy wasm	[host<-vm] proxy_on_response_body return: 0
2022-04-24T07:59:01.125000Z	trace	envoy http	[C44][S4314132064810061585] encode data called: filter=0x5629cf41be30 status=0
2022-04-24T07:59:01.125002Z	trace	envoy http	[C44][S4314132064810061585] encoding data via codec (size=0 end_stream=true)
2022-04-24T07:59:01.125006Z	trace	envoy connection	[C44] writing 5 bytes, end_stream false
2022-04-24T07:59:01.125009Z	trace	envoy main	item added to deferred deletion list (size=2)
2022-04-24T07:59:01.125092Z	trace	envoy wasm	[host->vm] proxy_on_log(13)
2022-04-24T07:59:01.125099Z	trace	envoy wasm	[host<-vm] proxy_on_log return: void
2022-04-24T07:59:01.125186Z	debug	envoy wasm	wasm log stats_inbound stats_inbound: [extensions/stats/plugin.cc:640]::report() metricKey cache hit , stat=12
2022-04-24T07:59:01.125192Z	debug	envoy wasm	wasm log stats_inbound stats_inbound: [extensions/stats/plugin.cc:640]::report() metricKey cache hit , stat=6
2022-04-24T07:59:01.125195Z	debug	envoy wasm	wasm log stats_inbound stats_inbound: [extensions/stats/plugin.cc:640]::report() metricKey cache hit , stat=10
2022-04-24T07:59:01.125198Z	debug	envoy wasm	wasm log stats_inbound stats_inbound: [extensions/stats/plugin.cc:640]::report() metricKey cache hit , stat=14
2022-04-24T07:59:01.125201Z	trace	envoy wasm	[host->vm] proxy_on_done(13)
2022-04-24T07:59:01.125205Z	trace	envoy wasm	[host<-vm] proxy_on_done return: 1
2022-04-24T07:59:01.125207Z	trace	envoy wasm	[host->vm] proxy_on_delete(13)
2022-04-24T07:59:01.125210Z	trace	envoy wasm	[host<-vm] proxy_on_delete return: void
2022-04-24T07:59:01.125216Z	trace	envoy main	item added to deferred deletion list (size=3)
2022-04-24T07:59:01.125219Z	trace	envoy misc	enableTimer called on 0x5629cfd50100 for 3600000ms, min is 3600000ms
2022-04-24T07:59:01.125223Z	debug	envoy pool	[C1160] response complete
2022-04-24T07:59:01.125226Z	debug	envoy pool	[C1160] destroying stream: 0 remaining
2022-04-24T07:59:01.125233Z	trace	envoy http	[C1160] parsed 5 bytes
2022-04-24T07:59:01.125236Z	trace	envoy main	clearing deferred deletion list (size=3)
2022-04-24T07:59:01.125240Z	trace	envoy connection	[C44] readDisable: disable=false disable_count=1 state=0 buffer_length=0
2022-04-24T07:59:01.125268Z	trace	envoy connection	[C44] socket event: 2
2022-04-24T07:59:01.125270Z	trace	envoy connection	[C44] write ready
2022-04-24T07:59:01.125312Z	trace	envoy connection	[C44] ssl write returns: 5
[2022-04-24T07:59:01.094Z] "GET /order/12 HTTP/1.1" 200 - via_upstream - "-" 0 4568 30 29 "127.0.0.6" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/100.0.4896.127 Safari/537.36" "a9699a23-4518-99f4-b0ff-db85295ce1e3" "mall-admin:8080" "10.36.0.11:8080" inbound|8080|| 127.0.0.6:39054 10.36.0.11:8080 127.0.0.6:0 outbound_.8080_._.mall-admin.mall.svc.cluster.local default
2022-04-24T07:59:03.002368Z	trace	envoy misc	enableTimer called on 0x5629cf7d4480 for 3600000ms, min is 3600000ms
2022-04-24T07:59:03.002394Z	debug	envoy conn_handler	[C1175] new connection from 10.36.0.0:37160
2022-04-24T07:59:03.002409Z	trace	envoy connection	[C1175] socket event: 3
2022-04-24T07:59:03.002412Z	trace	envoy connection	[C1175] write ready
2022-04-24T07:59:03.002415Z	trace	envoy connection	[C1175] read ready. dispatch_buffered_data=false
2022-04-24T07:59:03.002427Z	trace	envoy connection	[C1175] read returns: 116
2022-04-24T07:59:03.002436Z	trace	envoy connection	[C1175] read error: Resource temporarily unavailable
2022-04-24T07:59:03.002448Z	trace	envoy http	[C1175] parsing 116 bytes
2022-04-24T07:59:03.002462Z	trace	envoy http	[C1175] message begin
2022-04-24T07:59:03.002467Z	debug	envoy http	[C1175] new stream
2022-04-24T07:59:03.002475Z	trace	envoy misc	enableTimer called on 0x5629cf7d5480 for 300000ms, min is 300000ms
2022-04-24T07:59:03.002486Z	trace	envoy http	[C1175] completed header: key=Host value=10.36.0.11:15021
2022-04-24T07:59:03.002492Z	trace	envoy http	[C1175] completed header: key=User-Agent value=kube-probe/1.21
2022-04-24T07:59:03.002498Z	trace	envoy http	[C1175] completed header: key=Accept value=*/*
2022-04-24T07:59:03.002502Z	trace	envoy http	[C1175] onHeadersCompleteBase
2022-04-24T07:59:03.002504Z	trace	envoy http	[C1175] completed header: key=Connection value=close
2022-04-24T07:59:03.002510Z	trace	envoy http	[C1175] Server: onHeadersComplete size=4
2022-04-24T07:59:03.002517Z	trace	envoy http	[C1175] message complete
2022-04-24T07:59:03.002521Z	trace	envoy connection	[C1175] readDisable: disable=true disable_count=0 state=0 buffer_length=116
2022-04-24T07:59:03.002536Z	debug	envoy http	[C1175][S4613903178435323885] request headers complete (end_stream=true):
':authority', '10.36.0.11:15021'
':path', '/healthz/ready'
':method', 'GET'
'user-agent', 'kube-probe/1.21'
'accept', '*/*'
'connection', 'close'

2022-04-24T07:59:03.002540Z	debug	envoy http	[C1175][S4613903178435323885] request end stream
2022-04-24T07:59:03.002571Z	debug	envoy router	[C1175][S4613903178435323885] cluster 'agent' match for URL '/healthz/ready'
2022-04-24T07:59:03.002600Z	debug	envoy router	[C1175][S4613903178435323885] router decoding headers:
':authority', '10.36.0.11:15021'
':path', '/healthz/ready'
':method', 'GET'
':scheme', 'http'
'user-agent', 'kube-probe/1.21'
'accept', '*/*'
'x-forwarded-proto', 'http'
'x-request-id', '4183b024-9c21-4c7e-a180-973e768a834a'
'x-envoy-expected-rq-timeout-ms', '15000'

2022-04-24T07:59:03.002608Z	debug	envoy pool	[C8] using existing connection
2022-04-24T07:59:03.002611Z	debug	envoy pool	[C8] creating stream
2022-04-24T07:59:03.002618Z	debug	envoy router	[C1175][S4613903178435323885] pool ready
2022-04-24T07:59:03.002631Z	trace	envoy connection	[C8] writing 213 bytes, end_stream false
2022-04-24T07:59:03.002637Z	trace	envoy pool	not creating a new connection, shouldCreateNewConnection returned false.
2022-04-24T07:59:03.002642Z	trace	envoy http	[C1175][S4613903178435323885] decode headers called: filter=0x5629cfd2caf0 status=1
2022-04-24T07:59:03.002645Z	trace	envoy misc	enableTimer called on 0x5629cf7d5480 for 300000ms, min is 300000ms
2022-04-24T07:59:03.002649Z	trace	envoy http	[C1175] parsed 116 bytes
2022-04-24T07:59:03.002656Z	trace	envoy connection	[C1175] socket event: 2
2022-04-24T07:59:03.002658Z	trace	envoy connection	[C1175] write ready
2022-04-24T07:59:03.002661Z	trace	envoy connection	[C8] socket event: 2
2022-04-24T07:59:03.002663Z	trace	envoy connection	[C8] write ready
2022-04-24T07:59:03.002693Z	trace	envoy connection	[C8] write returns: 213
2022-04-24T07:59:03.002884Z	trace	envoy connection	[C8] socket event: 3
2022-04-24T07:59:03.002897Z	trace	envoy connection	[C8] write ready
2022-04-24T07:59:03.002900Z	trace	envoy connection	[C8] read ready. dispatch_buffered_data=false
2022-04-24T07:59:03.002907Z	trace	envoy connection	[C8] read returns: 75
2022-04-24T07:59:03.002912Z	trace	envoy connection	[C8] read error: Resource temporarily unavailable
2022-04-24T07:59:03.002916Z	trace	envoy http	[C8] parsing 75 bytes
2022-04-24T07:59:03.002919Z	trace	envoy http	[C8] message begin
2022-04-24T07:59:03.002927Z	trace	envoy http	[C8] completed header: key=Date value=Sun, 24 Apr 2022 07:59:03 GMT
2022-04-24T07:59:03.002931Z	trace	envoy http	[C8] onHeadersCompleteBase
2022-04-24T07:59:03.002933Z	trace	envoy http	[C8] completed header: key=Content-Length value=0
2022-04-24T07:59:03.002939Z	trace	envoy http	[C8] status_code 200
2022-04-24T07:59:03.002941Z	trace	envoy http	[C8] Client: onHeadersComplete size=2
2022-04-24T07:59:03.002945Z	trace	envoy http	[C8] message complete
2022-04-24T07:59:03.002947Z	trace	envoy http	[C8] message complete
2022-04-24T07:59:03.002949Z	debug	envoy client	[C8] response complete
2022-04-24T07:59:03.002953Z	trace	envoy main	item added to deferred deletion list (size=1)
2022-04-24T07:59:03.002960Z	debug	envoy router	[C1175][S4613903178435323885] upstream headers complete: end_stream=true
2022-04-24T07:59:03.002980Z	trace	envoy misc	enableTimer called on 0x5629cf7d5480 for 300000ms, min is 300000ms
2022-04-24T07:59:03.002991Z	debug	envoy http	[C1175][S4613903178435323885] closing connection due to connection close header
2022-04-24T07:59:03.002997Z	debug	envoy http	[C1175][S4613903178435323885] encoding headers via codec (end_stream=true):
':status', '200'
'date', 'Sun, 24 Apr 2022 07:59:03 GMT'
'content-length', '0'
'x-envoy-upstream-service-time', '0'
'server', 'envoy'
'connection', 'close'

2022-04-24T07:59:03.003004Z	trace	envoy connection	[C1175] writing 143 bytes, end_stream false
2022-04-24T07:59:03.003007Z	trace	envoy main	item added to deferred deletion list (size=2)
2022-04-24T07:59:03.003012Z	trace	envoy main	item added to deferred deletion list (size=3)
2022-04-24T07:59:03.003014Z	trace	envoy misc	enableTimer called on 0x5629cf7d4480 for 3600000ms, min is 3600000ms
2022-04-24T07:59:03.003018Z	debug	envoy connection	[C1175] closing data_to_write=143 type=2
2022-04-24T07:59:03.003021Z	debug	envoy connection	[C1175] setting delayed close timer with timeout 1000 ms
2022-04-24T07:59:03.003028Z	debug	envoy pool	[C8] response complete
2022-04-24T07:59:03.003031Z	debug	envoy pool	[C8] destroying stream: 0 remaining
2022-04-24T07:59:03.003035Z	trace	envoy http	[C8] parsed 75 bytes
2022-04-24T07:59:03.003038Z	trace	envoy main	clearing deferred deletion list (size=3)


```

过滤了指定服务后的日志样例

```shell
$ kubectl logs mall-admin-5498f4fb79-dcx9p -n mall -c istio-proxy  | grep "wasm log clean-mall-admin"

2022-04-27T03:59:58.385387Z	info	envoy wasm	wasm log clean-mall-admin: <---- 新Http连接 ---->
2022-04-27T03:59:58.385649Z	info	envoy wasm	wasm log clean-mall-admin: 请求头中的share data: 7359900035059749376
2022-04-27T03:59:58.749601Z	info	envoy wasm	wasm log clean-mall-admin: 返回体中获取shareData: 7359900035059749376
2022-04-27T03:59:58.749672Z	info	envoy wasm	wasm log clean-mall-admin: response body: {"code":200,"message":"操作成功","data":{"id":12,"memberId":1,"couponId":2,"orderSn":"201809150101000001","createTime":"2018-09-15T04:24:27.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":20.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":10.00,"payType":0,"sourceType":1,"status":4,"orderType":0,"deliveryCompany":"","deliverySn":"","autoConfirmDay":15,"integration":13284,"growth":13284,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"江苏省","receiverCity":"常州市","receiverRegion":"天宁区","receiverDetailAddress":"东晓街道","note":"1558000000","confirmStatus":0,"deleteStatus":0,"useIntegration":null,"paymentTime":null,"deliveryTime":null,"receiveTime":null,"commentTime":null,"modifyTime":"2022-04-25T02:47:34.000+00:00","orderItemList":[{"id":21,"orderId":null,"orderSn":null,"productId":26,"productPic":"http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/images/20180607/5ac1bf58Ndefaac16.jpg","productName":"华为 HUAWEI P20","productBrand":"华为","productSn":"6946605","productPrice":3788.00,"productQuantity":1,"productSkuId":null,"productSkuCode":null,"productCategoryId":null,"promotionName":null,"promotionAmount":null,"couponAmount":null,"integrationAmount":null,"realAmount":null,"giftIntegration":null,"giftGrowth":null,"productAttr":"[{\"key\":\"颜色\",\"value\":\"金色\"},{\"key\":\"容量\",\"value\":\"16G\"}]"},{"id":22,"orderId":null,"orderSn":null,"productId":27,"productPic":"http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/images/20180615/xiaomi.jpg","productName":"小米8","productBrand":"小米","productSn":"7437788","productPrice":2699.00,"productQuantity":3,"productSkuId":null,"productSkuCode":null,"productCategoryId":null,"promotionName":null,"promotionAmount":null,"couponAmount":null,"integrationAmount":null,"realAmount":null,"giftIntegration":null,"giftGrowth":null,"productAttr":"[{\"key\":\"颜色\",\"value\":\"黑色\"},{\"key\":\"容量\",\"value\":\"32G\"}]"},{"id":23,"orderId":null,"orderSn":null,"productId":28,"productPic":"http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/images/20180615/5a9d248cN071f4959.jpg","productName":"红米5A","productBrand":"小米","productSn":"7437789","productPrice":649.00,"productQuantity":1,"productSkuId":null,"productSkuCode":null,"productCategoryId":null,"promotionName":null,"promotionAmount":null,"couponAmount":null,"integrationAmount":null,"realAmount":null,"giftIntegration":null,"giftGrowth":null,"productAttr":"[{\"key\":\"颜色\",\"value\":\"金色\"},{\"key\":\"容量\",\"value\":\"16G\"}]"},{"id":24,"orderId":null,"orderSn":null,"productId":28,"productPic":"http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/images/20180615/5a9d248cN071f4959.jpg","productName":"红米5A","productBrand":"小米","productSn":"7437789","productPrice":699.00,"productQuantity":1,"productSkuId":null,"productSkuCode":null,"productCategoryId":null,"promotionName":null,"promotionAmount":null,"couponAmount":null,"integrationAmount":null,"realAmount":null,"giftIntegration":null,"giftGrowth":null,"productAttr":"[{\"key\":\"颜色\",\"value\":\"金色\"},{\"key\":\"容量\",\"value\":\"32G\"}]"},{"id":25,"orderId":null,"orderSn":null,"productId":29,"productPic":"http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/images/20180615/5acc5248N6a5f81cd.jpg","productName":"Apple iPhone 8 Plus","productBrand":"苹果","productSn":"7437799","productPrice":5499.00,"productQuantity":1,"productSkuId":null,"productSkuCode":null,"productCategoryId":null,"promotionName":null,"promotionAmount":null,"couponAmount":null,"integrationAmount":null,"realAmount":null,"giftIntegration":null,"giftGrowth":null,"productAttr":"[{\"key\":\"颜色\",\"value\":\"金色\"},{\"key\":\"容量\",\"value\":\"32G\"}]"}],"historyList":[{"id":29,"orderId":null,"operateMan":"后台管理员","createTime":"2022-04-25T02:47:34.000+00:00","orderStatus":4,"note":"修改备注信息：1558000000"},{"id":28,"orderId":null,"operateMan":"后台管理员","createTime":"2022-04-25T02:47:19.000+00:00","orderStatus":4,"note":"修改备注信息：15580000000"},{"id":27,"orderId":null,"operateMan":"后台管理员","createTime":"2022-04-25T02:39:42.000+00:00","orderStatus":4,"note":"修改备注信息：12345678901"},{"id":23,"orderId":null,"operateMan":"后台管理员","createTime":"2019-11-09T08:50:28.000+00:00","orderStatus":4,"note":"修改备注信息：111"},{"id":7,"orderId":null,"operateMan":"后台管理员","createTime":"2018-10-12T06:13:10.000+00:00","orderStatus":4,"note":"订单关闭:买家退货"},{"id":5,"orderId":null,"operateMan":"后台管理员","createTime":"2018-10-12T06:01:29.000+00:00","orderStatus":2,"note":"完成发货"}]}}
2022-04-27T03:59:58.749685Z	info	envoy wasm	wasm log clean-mall-admin: response body 是否结束: false
2022-04-27T03:59:58.754180Z	info	envoy wasm	wasm log clean-mall-admin: 返回体中获取shareData: 7359900035059749376
2022-04-27T04:00:01.142802Z	info	envoy wasm	wasm log clean-mall-admin: <---- 新Http连接 ---->
2022-04-27T04:00:01.142831Z	info	envoy wasm	wasm log clean-mall-admin: 请求头中的share data: 7420214226221166336
2022-04-27T04:00:01.327442Z	info	envoy wasm	wasm log clean-mall-admin: 返回体中获取shareData: 7420214226221166336
2022-04-27T04:00:01.327895Z	info	envoy wasm	wasm log clean-mall-admin: response body: {"code":200,"message":"操作成功","data":{"pageNum":1,"pageSize":10,"totalPage":2,"total":15,"list":[{"id":12,"memberId":1,"couponId":2,"orderSn":"201809150101000001","createTime":"2018-09-15T04:24:27.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":20.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":10.00,"payType":0,"sourceType":1,"status":4,"orderType":0,"deliveryCompany":"","deliverySn":"","autoConfirmDay":15,"integration":13284,"growth":13284,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"江苏省","receiverCity":"常州市","receiverRegion":"天宁区","receiverDetailAddress":"东晓街道","note":"1558000000","confirmStatus":0,"deleteStatus":0,"useIntegration":null,"paymentTime":null,"deliveryTime":null,"receiveTime":null,"commentTime":null,"modifyTime":"2022-04-25T02:47:34.000+00:00"},{"id":13,"memberId":1,"couponId":2,"orderSn":"201809150102000002","createTime":"2018-09-15T06:24:29.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":1,"sourceType":1,"status":1,"orderType":0,"deliveryCompany":"","deliverySn":"","autoConfirmDay":15,"integration":13284,"growth":13284,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":1000,"paymentTime":"2018-10-11T06:04:19.000+00:00","deliveryTime":null,"receiveTime":null,"commentTime":null,"modifyTime":null},{"id":14,"memberId":1,"couponId":2,"orderSn":"201809130101000001","createTime":"2018-09-13T08:57:40.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":2,"sourceType":1,"status":2,"orderType":0,"deliveryCompany":"顺丰快递","deliverySn":"201707196398345","autoConfirmDay":15,"integration":13284,"growth":13284,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":null,"paymentTime":"2018-10-13T05:44:04.000+00:00","deliveryTime":"2018-10-16T05:43:41.000+00:00","receiveTime":null,"commentTime":null,"modifyTime":null},{"id":15,"memberId":1,"couponId":2,"orderSn":"201809130102000002","createTime":"2018-09-13T09:03:00.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":1,"sourceType":1,"status":3,"orderType":0,"deliveryCompany":"顺丰快递","deliverySn":"201707196398346","autoConfirmDay":15,"integration":13284,"growth":13284,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":1,"deleteStatus":0,"useIntegration":null,"paymentTime":"2018-10-13T05:44:54.000+00:00","deliveryTime":"2018-10-16T05:45:01.000+00:00","receiveTime":"2018-10-18T06:05:31.000+00:00","commentTime":null,"modifyTime":null},{"id":16,"memberId":1,"couponId":2,"orderSn":"201809140101000001","createTime":"2018-09-14T08:16:16.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":2,"sourceType":1,"status":4,"orderType":0,"deliveryCompany":null,"deliverySn":null,"autoConfirmDay":15,"integration":13284,"growth":13284,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":null,"paymentTime":null,"deliveryTime":null,"receiveTime":null,"commentTime":null,"modifyTime":null},{"id":17,"memberId":1,"couponId":2,"orderSn":"201809150101000003","createTime":"2018-09-15T04:24:27.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":0,"sourceType":1,"status":4,"orderType":0,"deliveryCompany":"顺丰快递","deliverySn":"201707196398345","autoConfirmDay":15,"integration":null,"growth":null,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":null,"paymentTime":null,"deliveryTime":"2018-10-12T06:01:28.000+00:00","receiveTime":null,"commentTime":null,"modifyTime":null},{"id":18,"memberId":1,"couponId":2,"orderSn":"201809150102000004","createTime":"2018-09-15T06:24:29.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":1,"sourceType":1,"status":1,"orderType":0,"deliveryCompany":"圆通快递","deliverySn":"xx","autoConfirmDay":15,"integration":null,"growth":null,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":1000,"paymentTime":null,"deliveryTime":"2018-10-16T06:42:17.000+00:00","receiveTime":null,"commentTime":null,"modifyTime":null},{"id":19,"memberId":1,"couponId":2,"orderSn":"201809130101000003","createTime":"2018-09-13T08:57:40.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":2,"sourceType":1,"status":2,"orderType":0,"deliveryCompany":null,"deliverySn":null,"autoConfirmDay":15,"integration":null,"growth":null,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":null,"paymentTime":null,"deliveryTime":null,"receiveTime":null,"commentTime":null,"modifyTime":null},{"id":20,"memberId":1,"couponId":2,"orderSn":"201809130102000004","createTime":"2018-09-13T09:03:00.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":1,"sourceType":1,"status":3,"orderType":0,"deliveryCompany":null,"deliverySn":null,"autoConfirmDay":15,"integration":null,"growth":null,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":null,"paymentTime":null,"deliveryTime":null,"receiveTime":null,"commentTime":null,"modifyTime":null},{"id":22,"memberId":1,"couponId":2,"orderSn":"201809150101000005","createTime":"2018-09-15T04:24:27.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":0,"sourceType":1,"status":4,"orderType":0,"deliveryCompany":"顺丰快递","deliverySn":"201707196398345","autoConfirmDay":15,"integration":0,"growth":0,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":null,"paymentTime":null,"deliveryTime":"2018-10-12T06:01:28.000+00:00","receiveTime":null,"commentTime":null,"modifyTime":null}]}}
2022-04-27T04:00:01.327911Z	info	envoy wasm	wasm log clean-mall-admin: response body 是否结束: false
2022-04-27T04:00:01.327976Z	info	envoy wasm	wasm log clean-mall-admin: 返回体中获取shareData: 7420214226221166336
2022-04-27T04:00:02.965062Z	info	envoy wasm	wasm log clean-mall-admin: <---- 新Http连接 ---->
2022-04-27T04:00:02.965090Z	info	envoy wasm	wasm log clean-mall-admin: 请求头中的share data: 12191905453584448768
2022-04-27T04:00:02.994514Z	info	envoy wasm	wasm log clean-mall-admin: 返回体中获取shareData: 12191905453584448768
2022-04-27T04:00:02.994572Z	info	envoy wasm	wasm log clean-mall-admin: response body: {"code":200,"message":"操作成功","data":{"id":13,"memberId":1,"couponId":2,"orderSn":"201809150102000002","createTime":"2018-09-15T06:24:29.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":1,"sourceType":1,"status":1,"orderType":0,"deliveryCompany":"","deliverySn":"","autoConfirmDay":15,"integration":13284,"growth":13284,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":1000,"paymentTime":"2018-10-11T06:04:19.000+00:00","deliveryTime":null,"receiveTime":null,"commentTime":null,"modifyTime":null,"orderItemList":[{"id":26,"orderId":null,"orderSn":null,"productId":26,"productPic":"http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/images/20180607/5ac1bf58Ndefaac16.jpg","productName":"华为 HUAWEI P20","productBrand":"华为","productSn":"6946605","productPrice":3788.00,"productQuantity":1,"productSkuId":null,"productSkuCode":null,"productCategoryId":null,"promotionName":null,"promotionAmount":null,"couponAmount":null,"integrationAmount":null,"realAmount":null,"giftIntegration":null,"giftGrowth":null,"productAttr":"[{\"key\":\"颜色\",\"value\":\"金色\"},{\"key\":\"容量\",\"value\":\"16G\"}]"},{"id":27,"orderId":null,"orderSn":null,"productId":27,"productPic":"http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/images/20180615/xiaomi.jpg","productName":"小米8","productBrand":"小米","productSn":"7437788","productPrice":2699.00,"productQuantity":3,"productSkuId":null,"productSkuCode":null,"productCategoryId":null,"promotionName":null,"promotionAmount":null,"couponAmount":null,"integrationAmount":null,"realAmount":null,"giftIntegration":null,"giftGrowth":null,"productAttr":"[{\"key\":\"颜色\",\"value\":\"黑色\"},{\"key\":\"容量\",\"value\":\"32G\"}]"},{"id":28,"orderId":null,"orderSn":null,"productId":28,"productPic":"http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/images/20180615/5a9d248cN071f4959.jpg","productName":"红米5A","productBrand":"小米","productSn":"7437789","productPrice":649.00,"productQuantity":1,"productSkuId":null,"productSkuCode":null,"productCategoryId":null,"promotionName":null,"promotionAmount":null,"couponAmount":null,"integrationAmount":null,"realAmount":null,"giftIntegration":null,"giftGrowth":null,"productAttr":"[{\"key\":\"颜色\",\"value\":\"金色\"},{\"key\":\"容量\",\"value\":\"16G\"}]"},{"id":29,"orderId":null,"orderSn":null,"productId":28,"productPic":"http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/images/20180615/5a9d248cN071f4959.jpg","productName":"红米5A","productBrand":"小米","productSn":"7437789","productPrice":699.00,"productQuantity":1,"productSkuId":null,"productSkuCode":null,"productCategoryId":null,"promotionName":null,"promotionAmount":null,"couponAmount":null,"integrationAmount":null,"realAmount":null,"giftIntegration":null,"giftGrowth":null,"productAttr":"[{\"key\":\"颜色\",\"value\":\"金色\"},{\"key\":\"容量\",\"value\":\"32G\"}]"},{"id":30,"orderId":null,"orderSn":null,"productId":29,"productPic":"http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/images/20180615/5acc5248N6a5f81cd.jpg","productName":"Apple iPhone 8 Plus","productBrand":"苹果","productSn":"7437799","productPrice":5499.00,"productQuantity":1,"productSkuId":null,"productSkuCode":null,"productCategoryId":null,"promotionName":null,"promotionAmount":null,"couponAmount":null,"integrationAmount":null,"realAmount":null,"giftIntegration":null,"giftGrowth":null,"productAttr":"[{\"key\":\"颜色\",\"value\":\"金色\"},{\"key\":\"容量\",\"value\":\"32G\"}]"}],"historyList":[{"id":16,"orderId":null,"operateMan":"后台管理员","createTime":"2018-10-16T06:42:17.000+00:00","orderStatus":2,"note":"完成发货"},{"id":8,"orderId":null,"operateMan":"后台管理员","createTime":"2018-10-12T06:13:10.000+00:00","orderStatus":4,"note":"订单关闭:买家退货"},{"id":6,"orderId":null,"operateMan":"后台管理员","createTime":"2018-10-12T06:01:29.000+00:00","orderStatus":2,"note":"完成发货"}]}}
2022-04-27T04:00:02.994584Z	info	envoy wasm	wasm log clean-mall-admin: response body 是否结束: false
2022-04-27T04:00:02.995514Z	info	envoy wasm	wasm log clean-mall-admin: 返回体中获取shareData: 12191905453584448768
2022-04-27T04:00:04.511393Z	info	envoy wasm	wasm log clean-mall-admin: <---- 新Http连接 ---->
2022-04-27T04:00:04.511436Z	info	envoy wasm	wasm log clean-mall-admin: 请求头中的share data: 6628819207911744256
2022-04-27T04:00:04.539071Z	info	envoy wasm	wasm log clean-mall-admin: 返回体中获取shareData: 6628819207911744256
2022-04-27T04:00:04.539144Z	info	envoy wasm	wasm log clean-mall-admin: response body: {"code":200,"message":"操作成功","data":{"pageNum":1,"pageSize":10,"totalPage":2,"total":15,"list":[{"id":12,"memberId":1,"couponId":2,"orderSn":"201809150101000001","createTime":"2018-09-15T04:24:27.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":20.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":10.00,"payType":0,"sourceType":1,"status":4,"orderType":0,"deliveryCompany":"","deliverySn":"","autoConfirmDay":15,"integration":13284,"growth":13284,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"江苏省","receiverCity":"常州市","receiverRegion":"天宁区","receiverDetailAddress":"东晓街道","note":"1558000000","confirmStatus":0,"deleteStatus":0,"useIntegration":null,"paymentTime":null,"deliveryTime":null,"receiveTime":null,"commentTime":null,"modifyTime":"2022-04-25T02:47:34.000+00:00"},{"id":13,"memberId":1,"couponId":2,"orderSn":"201809150102000002","createTime":"2018-09-15T06:24:29.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":1,"sourceType":1,"status":1,"orderType":0,"deliveryCompany":"","deliverySn":"","autoConfirmDay":15,"integration":13284,"growth":13284,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":1000,"paymentTime":"2018-10-11T06:04:19.000+00:00","deliveryTime":null,"receiveTime":null,"commentTime":null,"modifyTime":null},{"id":14,"memberId":1,"couponId":2,"orderSn":"201809130101000001","createTime":"2018-09-13T08:57:40.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":2,"sourceType":1,"status":2,"orderType":0,"deliveryCompany":"顺丰快递","deliverySn":"201707196398345","autoConfirmDay":15,"integration":13284,"growth":13284,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":null,"paymentTime":"2018-10-13T05:44:04.000+00:00","deliveryTime":"2018-10-16T05:43:41.000+00:00","receiveTime":null,"commentTime":null,"modifyTime":null},{"id":15,"memberId":1,"couponId":2,"orderSn":"201809130102000002","createTime":"2018-09-13T09:03:00.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":1,"sourceType":1,"status":3,"orderType":0,"deliveryCompany":"顺丰快递","deliverySn":"201707196398346","autoConfirmDay":15,"integration":13284,"growth":13284,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":1,"deleteStatus":0,"useIntegration":null,"paymentTime":"2018-10-13T05:44:54.000+00:00","deliveryTime":"2018-10-16T05:45:01.000+00:00","receiveTime":"2018-10-18T06:05:31.000+00:00","commentTime":null,"modifyTime":null},{"id":16,"memberId":1,"couponId":2,"orderSn":"201809140101000001","createTime":"2018-09-14T08:16:16.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":2,"sourceType":1,"status":4,"orderType":0,"deliveryCompany":null,"deliverySn":null,"autoConfirmDay":15,"integration":13284,"growth":13284,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":null,"paymentTime":null,"deliveryTime":null,"receiveTime":null,"commentTime":null,"modifyTime":null},{"id":17,"memberId":1,"couponId":2,"orderSn":"201809150101000003","createTime":"2018-09-15T04:24:27.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":0,"sourceType":1,"status":4,"orderType":0,"deliveryCompany":"顺丰快递","deliverySn":"201707196398345","autoConfirmDay":15,"integration":null,"growth":null,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":null,"paymentTime":null,"deliveryTime":"2018-10-12T06:01:28.000+00:00","receiveTime":null,"commentTime":null,"modifyTime":null},{"id":18,"memberId":1,"couponId":2,"orderSn":"201809150102000004","createTime":"2018-09-15T06:24:29.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":1,"sourceType":1,"status":1,"orderType":0,"deliveryCompany":"圆通快递","deliverySn":"xx","autoConfirmDay":15,"integration":null,"growth":null,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":1000,"paymentTime":null,"deliveryTime":"2018-10-16T06:42:17.000+00:00","receiveTime":null,"commentTime":null,"modifyTime":null},{"id":19,"memberId":1,"couponId":2,"orderSn":"201809130101000003","
2022-04-27T04:00:04.539156Z	info	envoy wasm	wasm log clean-mall-admin: response body 是否结束: false
2022-04-27T04:00:04.539227Z	info	envoy wasm	wasm log clean-mall-admin: 返回体中获取shareData: 6628819207911744256
2022-04-27T04:00:04.539262Z	info	envoy wasm	wasm log clean-mall-admin: response body: createTime":"2018-09-13T08:57:40.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":2,"sourceType":1,"status":2,"orderType":0,"deliveryCompany":null,"deliverySn":null,"autoConfirmDay":15,"integration":null,"growth":null,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":null,"paymentTime":null,"deliveryTime":null,"receiveTime":null,"commentTime":null,"modifyTime":null},{"id":20,"memberId":1,"couponId":2,"orderSn":"201809130102000004","createTime":"2018-09-13T09:03:00.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":1,"sourceType":1,"status":3,"orderType":0,"deliveryCompany":null,"deliverySn":null,"autoConfirmDay":15,"integration":null,"growth":null,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":null,"paymentTime":null,"deliveryTime":null,"receiveTime":null,"commentTime":null,"modifyTime":null},{"id":22,"memberId":1,"couponId":2,"orderSn":"201809150101000005","createTime":"2018-09-15T04:24:27.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":0,"sourceType":1,"status":4,"orderType":0,"deliveryCompany":"顺丰快递","deliverySn":"201707196398345","autoConfirmDay":15,"integration":0,"growth":0,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":null,"paymentTime":null,"deliveryTime":"2018-10-12T06:01:28.000+00:00","receiveTime":null,"commentTime":null,"modifyTime":null}]}}
2022-04-27T04:00:04.539271Z	info	envoy wasm	wasm log clean-mall-admin: response body 是否结束: false
2022-04-27T04:00:04.539508Z	info	envoy wasm	wasm log clean-mall-admin: 返回体中获取shareData: 6628819207911744256
2022-04-27T04:00:09.054218Z	info	envoy wasm	wasm log clean-mall-admin: <---- 新Http连接 ---->
2022-04-27T04:00:09.054259Z	info	envoy wasm	wasm log clean-mall-admin: 请求头中的share data: 15745463481192591616
2022-04-27T04:00:09.066593Z	info	envoy wasm	wasm log clean-mall-admin: 返回体中获取shareData: 15745463481192591616
2022-04-27T04:00:09.067141Z	info	envoy wasm	wasm log clean-mall-admin: response body: {"code":200,"message":"操作成功","data":{"id":14,"memberId":1,"couponId":2,"orderSn":"201809130101000001","createTime":"2018-09-13T08:57:40.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":2,"sourceType":1,"status":2,"orderType":0,"deliveryCompany":"顺丰快递","deliverySn":"201707196398345","autoConfirmDay":15,"integration":13284,"growth":13284,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":null,"paymentTime":"2018-10-13T05:44:04.000+00:00","deliveryTime":"2018-10-16T05:43:41.000+00:00","receiveTime":null,"commentTime":null,"modifyTime":null,"orderItemList":[{"id":31,"orderId":null,"orderSn":null,"productId":26,"productPic":"http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/images/20180607/5ac1bf58Ndefaac16.jpg","productName":"华为 HUAWEI P20","productBrand":"华为","productSn":"6946605","productPrice":3788.00,"productQuantity":1,"productSkuId":null,"productSkuCode":null,"productCategoryId":null,"promotionName":null,"promotionAmount":null,"couponAmount":null,"integrationAmount":null,"realAmount":null,"giftIntegration":null,"giftGrowth":null,"productAttr":"[{\"key\":\"颜色\",\"value\":\"金色\"},{\"key\":\"容量\",\"value\":\"16G\"}]"},{"id":32,"orderId":null,"orderSn":null,"productId":27,"productPic":"http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/images/20180615/xiaomi.jpg","productName":"小米8","productBrand":"小米","productSn":"7437788","productPrice":2699.00,"productQuantity":3,"productSkuId":null,"productSkuCode":null,"productCategoryId":null,"promotionName":null,"promotionAmount":null,"couponAmount":null,"integrationAmount":null,"realAmount":null,"giftIntegration":null,"giftGrowth":null,"productAttr":"[{\"key\":\"颜色\",\"value\":\"黑色\"},{\"key\":\"容量\",\"value\":\"32G\"}]"},{"id":33,"orderId":null,"orderSn":null,"productId":28,"productPic":"http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/images/20180615/5a9d248cN071f4959.jpg","productName":"红米5A","productBrand":"小米","productSn":"7437789","productPrice":649.00,"productQuantity":1,"productSkuId":null,"productSkuCode":null,"productCategoryId":null,"promotionName":null,"promotionAmount":null,"couponAmount":null,"integrationAmount":null,"realAmount":null,"giftIntegration":null,"giftGrowth":null,"productAttr":"[{\"key\":\"颜色\",\"value\":\"金色\"},{\"key\":\"容量\",\"value\":\"16G\"}]"},{"id":34,"orderId":null,"orderSn":null,"productId":28,"productPic":"http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/images/20180615/5a9d248cN071f4959.jpg","productName":"红米5A","productBrand":"小米","productSn":"7437789","productPrice":699.00,"productQuantity":1,"productSkuId":null,"productSkuCode":null,"productCategoryId":null,"promotionName":null,"promotionAmount":null,"couponAmount":null,"integrationAmount":null,"realAmount":null,"giftIntegration":null,"giftGrowth":null,"productAttr":"[{\"key\":\"颜色\",\"value\":\"金色\"},{\"key\":\"容量\",\"value\":\"32G\"}]"},{"id":35,"orderId":null,"orderSn":null,"productId":29,"productPic":"http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/images/20180615/5acc5248N6a5f81cd.jpg","productName":"Apple iPhone 8 Plus","productBrand":"苹果","productSn":"7437799","productPrice":5499.00,"productQuantity":1,"productSkuId":null,"productSkuCode":null,"productCategoryId":null,"promotionName":null,"promotionAmount":null,"couponAmount":null,"integrationAmount":null,"realAmount":null,"giftIntegration":null,"giftGrowth":null,"productAttr":"[{\"key\":\"颜色\",\"value\":\"金色\"},{\"key\":\"容量\",\"value\":\"32G\"}]"}],"historyList":[]}}
2022-04-27T04:00:09.067153Z	info	envoy wasm	wasm log clean-mall-admin: response body 是否结束: false
2022-04-27T04:00:09.067829Z	info	envoy wasm	wasm log clean-mall-admin: 返回体中获取shareData: 15745463481192591616
2022-04-27T04:00:10.948641Z	info	envoy wasm	wasm log clean-mall-admin: <---- 新Http连接 ---->
2022-04-27T04:00:10.948675Z	info	envoy wasm	wasm log clean-mall-admin: 请求头中的share data: 11867813742393546752
2022-04-27T04:00:10.964548Z	info	envoy wasm	wasm log clean-mall-admin: 返回体中获取shareData: 11867813742393546752
2022-04-27T04:00:10.964625Z	info	envoy wasm	wasm log clean-mall-admin: response body: {"code":200,"message":"操作成功","data":{"pageNum":1,"pageSize":10,"totalPage":2,"total":15,"list":[{"id":12,"memberId":1,"couponId":2,"orderSn":"201809150101000001","createTime":"2018-09-15T04:24:27.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":20.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":10.00,"payType":0,"sourceType":1,"status":4,"orderType":0,"deliveryCompany":"","deliverySn":"","autoConfirmDay":15,"integration":13284,"growth":13284,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"江苏省","receiverCity":"常州市","receiverRegion":"天宁区","receiverDetailAddress":"东晓街道","note":"1558000000","confirmStatus":0,"deleteStatus":0,"useIntegration":null,"paymentTime":null,"deliveryTime":null,"receiveTime":null,"commentTime":null,"modifyTime":"2022-04-25T02:47:34.000+00:00"},{"id":13,"memberId":1,"couponId":2,"orderSn":"201809150102000002","createTime":"2018-09-15T06:24:29.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":1,"sourceType":1,"status":1,"orderType":0,"deliveryCompany":"","deliverySn":"","autoConfirmDay":15,"integration":13284,"growth":13284,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":1000,"paymentTime":"2018-10-11T06:04:19.000+00:00","deliveryTime":null,"receiveTime":null,"commentTime":null,"modifyTime":null},{"id":14,"memberId":1,"couponId":2,"orderSn":"201809130101000001","createTime":"2018-09-13T08:57:40.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":2,"sourceType":1,"status":2,"orderType":0,"deliveryCompany":"顺丰快递","deliverySn":"201707196398345","autoConfirmDay":15,"integration":13284,"growth":13284,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":null,"paymentTime":"2018-10-13T05:44:04.000+00:00","deliveryTime":"2018-10-16T05:43:41.000+00:00","receiveTime":null,"commentTime":null,"modifyTime":null},{"id":15,"memberId":1,"couponId":2,"orderSn":"201809130102000002","createTime":"2018-09-13T09:03:00.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":1,"sourceType":1,"status":3,"orderType":0,"deliveryCompany":"顺丰快递","deliverySn":"201707196398346","autoConfirmDay":15,"integration":13284,"growth":13284,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":1,"deleteStatus":0,"useIntegration":null,"paymentTime":"2018-10-13T05:44:54.000+00:00","deliveryTime":"2018-10-16T05:45:01.000+00:00","receiveTime":"2018-10-18T06:05:31.000+00:00","commentTime":null,"modifyTime":null},{"id":16,"memberId":1,"couponId":2,"orderSn":"201809140101000001","createTime":"2018-09-14T08:16:16.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":2,"sourceType":1,"status":4,"orderType":0,"deliveryCompany":null,"deliverySn":null,"autoConfirmDay":15,"integration":13284,"growth":13284,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":null,"paymentTime":null,"deliveryTime":null,"receiveTime":null,"commentTime":null,"modifyTime":null},{"id":17,"memberId":1,"couponId":2,"orderSn":"201809150101000003","createTime":"2018-09-15T04:24:27.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":0,"sourceType":1,"status":4,"orderType":0,"deliveryCompany":"顺丰快递","deliverySn":"201707196398345","autoConfirmDay":15,"integration":null,"growth":null,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":null,"paymentTime":null,"deliveryTime":"2018-10-12T06:01:28.000+00:00","receiveTime":null,"commentTime":null,"modifyTime":null},{"id":18,"memberId":1,"couponId":2,"orderSn":"201809150102000004","createTime":"2018-09-15T06:24:29.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":1,"sourceType":1,"status":1,"orderType":0,"deliveryCompany":"圆通快递","deliverySn":"xx","autoConfirmDay":15,"integration":null,"growth":null,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":1000,"paymentTime":null,"deliveryTime":"2018-10-16T06:42:17.000+00:00","receiveTime":null,"commentTime":null,"modifyTime":null},{"id":19,"memberId":1,"couponId":2,"orderSn":"201809130101000003","
2022-04-27T04:00:10.964637Z	info	envoy wasm	wasm log clean-mall-admin: response body 是否结束: false
2022-04-27T04:00:10.964707Z	info	envoy wasm	wasm log clean-mall-admin: 返回体中获取shareData: 11867813742393546752
2022-04-27T04:00:10.964802Z	info	envoy wasm	wasm log clean-mall-admin: response body: createTime":"2018-09-13T08:57:40.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":2,"sourceType":1,"status":2,"orderType":0,"deliveryCompany":null,"deliverySn":null,"autoConfirmDay":15,"integration":null,"growth":null,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":null,"paymentTime":null,"deliveryTime":null,"receiveTime":null,"commentTime":null,"modifyTime":null},{"id":20,"memberId":1,"couponId":2,"orderSn":"201809130102000004","createTime":"2018-09-13T09:03:00.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":1,"sourceType":1,"status":3,"orderType":0,"deliveryCompany":null,"deliverySn":null,"autoConfirmDay":15,"integration":null,"growth":null,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":null,"paymentTime":null,"deliveryTime":null,"receiveTime":null,"commentTime":null,"modifyTime":null},{"id":22,"memberId":1,"couponId":2,"orderSn":"201809150101000005","createTime":"2018-09-15T04:24:27.000+00:00","memberUsername":"test","totalAmount":18732.00,"payAmount":16377.75,"freightAmount":0.00,"promotionAmount":2344.25,"integrationAmount":0.00,"couponAmount":10.00,"discountAmount":0.00,"payType":0,"sourceType":1,"status":4,"orderType":0,"deliveryCompany":"顺丰快递","deliverySn":"201707196398345","autoConfirmDay":15,"integration":0,"growth":0,"promotionInfo":"单品促销,打折优惠：满3件，打7.50折,满减优惠：满1000.00元，减120.00元,满减优惠：满1000.00元，减120.00元,无优惠","billType":null,"billHeader":null,"billContent":null,"billReceiverPhone":null,"billReceiverEmail":null,"receiverName":"大梨","receiverPhone":"18033441849","receiverPostCode":"518000","receiverProvince":"广东省","receiverCity":"深圳市","receiverRegion":"福田区","receiverDetailAddress":"东晓街道","note":null,"confirmStatus":0,"deleteStatus":0,"useIntegration":null,"paymentTime":null,"deliveryTime":"2018-10-12T06:01:28.000+00:00","receiveTime":null,"commentTime":null,"modifyTime":null}]}}
2022-04-27T04:00:10.964811Z	info	envoy wasm	wasm log clean-mall-admin: response body 是否结束: false
2022-04-27T04:00:10.965037Z	info	envoy wasm	wasm log clean-mall-admin: 返回体中获取shareData: 11867813742393546752

```