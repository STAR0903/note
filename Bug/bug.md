### localhost

当你通过 `localhost:8500/hello` 进行连接时，操作系统的域名解析器（DNS Resolver）会将 `localhost` 解析为本地回环地址。

根据系统配置或网络环境的不同，解析器可能优先选择 IPv6 地址（即 `::1`），而非 IPv4 地址（即 `127.0.0.1`）。这意味着你的程序实际上尝试连接的是 `[::1]:8500`，而不是 `127.0.0.1:8500`。如果目标服务仅监听在 IPv4 地址上，这种地址族不匹配会导致连接失败或报错。

### ERROR 1045 (28000):Access denied for user 'canal'@'localhost' (using password: YES)

你可能同时运行了本地安装的 MySQL（Windows 服务） 和 Docker 容器中的 MySQL

`host.docker.internal:3306` 或者 `localhost:3306` 可能被路由到了本地 MySQL 服务 ，而不是你的 Docker 容器。
