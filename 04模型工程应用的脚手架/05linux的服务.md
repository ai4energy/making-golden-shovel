## Linux 服务与 NGINX 实践：工程部署的核心

大家好。通过上一期的学习，我们已经将工作环境切换到了高效且无缝集成的 WSL Ubuntu。

虽然我们有像 cmder 或 msys2 这样的工具可以在 Windows 下体验命令行，但我们使用 WSL 的核心目的，并非仅仅为了敲命令，而是为了深入理解和实践 **Linux 的系统服务**。因为任何工程部署和平台运行，最终都依赖于这些后台服务的稳定运行。

今天，我们就来揭开这些系统服务的面纱，并从最常用的 **NGINX** 服务入手，构建我们平台部署的基础。

-----

### 一、Linux 服务的核心思想：守护进程

在 Linux 世界中，那些需要长时间运行、提供核心功能的程序，被称为**守护进程（Daemon）或系统服务（Service）**。它们通常在系统启动时自动运行，并在后台默默工作，不依赖于任何用户界面。

在现代 Linux 发行版（包括我们的 WSL Ubuntu）中，管理这些服务的核心工具是 **systemd**，我们通过 `systemctl` 命令与它进行交互。

-----

### 二、实践案例一：高性能 Web 服务器 NGINX

无论是作为 Web 服务器、反向代理（Reverse Proxy）还是负载均衡器，NGINX 都是构建现代网络服务的核心组件。我们的平台部署也离不开它。

#### 1\. 服务的安装与启停

在我们的 Ubuntu 环境中，安装 NGINX 只需要使用我们熟悉的包管理工具 APT：

```bash
# 安装 NGINX
sudo apt install nginx

# 启动 NGINX 服务
sudo systemctl start nginx

# 设置 NGINX 开机自启（在 WSL 中通常不需此步，但云服务器上必需）
sudo systemctl enable nginx
```

安装完成后，我们可以使用 `systemctl` 来控制服务：

```bash
# 查看服务状态（看它是否处于 active (running)）
sudo systemctl status nginx

# 停止服务
sudo systemctl stop nginx

# 重启服务（常用于修改配置后）
sudo systemctl restart nginx
```

#### 2\. NGINX 的配置文件

服务的生命在于配置。NGINX 的配置集中在 `/etc/nginx/` 目录下。

  * **核心主配置文件：** `/etc/nginx/nginx.conf`，这个文件定义了 NGINX 的全局行为。
  * **站点配置：** 在 `/etc/nginx/sites-available/` 目录下，存放着所有可用的虚拟主机（Virtual Host）或站点配置。我们通常修改或创建这里的文件。

> 例如，如果你想让 NGINX 代理我们的应用，你需要修改或创建一个新的配置，指定它监听的端口，以及将请求转发（代理）给后台应用（如 Python 或 Java 服务）的地址。每次修改配置后，务必使用 `sudo systemctl restart nginx` 来重新加载。

-----

### 三、实践案例二：远程管理与 SFTP

在实际的云服务器管理中，我们不可能直接坐在服务器面前。**SSH (Secure Shell)** 服务是远程管理和安全通信的行业标准。

#### 1\. SSH 服务的安装与配置

SSH 服务通常由 OpenSSH Server 提供。

```bash
# 安装 SSH 服务器端
sudo apt install openssh-server

# 启动 SSH 服务
sudo systemctl start ssh
```

SSH 的核心配置文件位于：`/etc/ssh/sshd_config`。

  * 在这里，我们可以修改默认的 **22 端口**，以提高安全性；
  * 可以配置是否允许 Root 用户直接登录；
  * 或者启用基于**公钥/私钥**的身份验证，这也是在工程部署中比密码登录更安全的标准做法。

修改配置后，同样需要使用 `sudo systemctl restart ssh` 来应用更改。

#### 2\. SFTP 服务的简单介绍

**SFTP（SSH File Transfer Protocol）** 并不是一个独立的服务，它是 SSH 协议的一部分。这意味着，只要你的 SSH 服务启动并运行正常，你就自动拥有了一个 SFTP 服务器。

SFTP 允许你通过加密通道安全地上传和下载文件，它是工程部署中将代码和配置文件传输到服务器的最常用、最安全的方式。

-----

### 四、WSL 与宿主机的网络类型与端口映射

这是我们在 WSL 中部署服务的关键。在传统虚拟机中，你需要手动进行端口转发。但在 WSL 中，特别是 **WSL 2**，它实现了几乎无缝的网络集成：

  * **直连访问：** 在 WSL 中运行的任何服务，例如 NGINX 监听的 80 端口，都可以直接通过 Windows 宿主机上的 `localhost` 或 `127.0.0.1` 来访问。
  * **自动映射：** 微软的 WSL 架构会自动处理必要的端口映射和网络转发。当你用浏览器在 Windows 端访问 `http://localhost` 时，流量会被无感地路由到 WSL 中的 NGINX 服务。

> 这意味着，部署在 WSL Linux 环境中的 NGINX、后台应用或数据库服务，可以直接被你的 Windows 浏览器或 Windows 上的其他开发工具访问，就像它们是 Windows 原生程序一样。

-----

**总结：** 掌握了 NGINX 这样的 Web 服务和 SSH 这样的管理服务，以及 WSL 的网络特性，我们就具备了在 Linux 环境中部署和管理微服务的基础能力。