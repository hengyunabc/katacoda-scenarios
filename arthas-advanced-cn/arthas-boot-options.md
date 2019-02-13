

`arthas-boot.jar` 支持很多参数。，可以执行 `java -jar arthas-boot.jar -h`{{execute T2}} 来查看。

### 允许远程连接

默认情况下 arthas server只侦听 `127.0.0.1`，可以通过`--target-ip`参数显式侦听所有IP，这样用户可以远程访问。

`java -jar arthas-boot.jar --target-ip 0.0.0.0`{{execute T2}}

### 指定侦听的端口

默认情况下，Arthas会侦听两个端口，一个是telnet端口 3658，另一个是http端口 8563。

可以通过参数显式指定，比如下面指定telnet端口为 9999，http端口为`-1`，即不启动http。

`java -jar arthas-boot.jar --telnet-port 9999 --http-port -1`{{execute T2}}