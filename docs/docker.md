## 使用 Alpine 作为基础镜像时，遇到的No such file or directory

现象：copy一个可执行文件到镜像中，执行的时候，发现执行不了。该镜像的基础镜像为Alpine

```shell
bash-4.4# dlv 
bash: /usr/local/bin/dlv: No such file or directory
```

解决办法：
容器内执行或重新编写dockerfile：`apk add libc6-compat`

```shell
bash-4.4# apk add libc6-compat
fetch http://mirrors.tuna.tsinghua.edu.cn/alpine/v3.7/main/x86_64/APKINDEX.tar.gz
ERROR: http://mirrors.tuna.tsinghua.edu.cn/alpine/v3.7/main: Permission denied
fetch http://mirrors.tuna.tsinghua.edu.cn/alpine/v3.7/community/x86_64/APKINDEX.tar.gz
ERROR: http://mirrors.tuna.tsinghua.edu.cn/alpine/v3.7/community: Permission denied
(1/1) Installing libc6-compat (1.1.18-r4)
OK: 39 MiB in 38 packages
bash-4.4# 
bash-4.4# 
bash-4.4# 
bash-4.4# dlv 
Delve is a source level debugger for Go programs.

Delve enables you to interact with your program by controlling the execution of the process,
evaluating variables, and providing information of thread / goroutine state, CPU register state and more.

The goal of this tool is to provide a simple yet powerful interface for debugging Go programs.

Pass flags to the program you are debugging using `--`, for example:

`dlv exec ./hello -- server --config conf/config.toml`

Usage:
  dlv [command]

Available Commands:
  attach      Attach to running process and begin debugging.
  connect     Connect to a headless debug server with a terminal client.
  core        Examine a core dump.
  dap         Starts a headless TCP server communicating via Debug Adaptor Protocol (DAP).
  debug       Compile and begin debugging main package in current directory, or the package specified.
  exec        Execute a precompiled binary, and begin a debug session.
  help        Help about any command
  run         Deprecated command. Use 'debug' instead.
  test        Compile test binary and begin debugging program.
  trace       Compile and begin tracing program.
  version     Prints version.
```

Reference：\
[使用 Alpine 作为基础镜像时可能会遇到的常见问题的解决方法](https://mozillazg.com/2020/03/use-alpine-image-common-issues.rst.html)

[Docker Alpine executable binary not found even if in PATH](https://stackoverflow.com/questions/66963068/docker-alpine-executable-binary-not-found-even-if-in-path)

