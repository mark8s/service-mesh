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


