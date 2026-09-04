# 问题背景
GitHub 服务器在海外，受国内网络环境影响（DNS 污染、国际线路拥堵等），常见问题：
- 网页打开缓慢、超时
- `git clone` 速度极慢或失败
- Release 大文件下载中断
以下为 3 种实测有效的方法，按推荐程度排序。
## 方法一：URL 前缀代理（推荐）

原理：在 GitHub 地址前加代理前缀，请求先进国内加速节点转发。

| # | 站点 | 类型 | 支持范围 | 速度 |
| --- | --- | --- | --- | --- |
| 1 | gh-proxy.com | URL前缀代理 | clone + Release + raw + archive | ★★★ |
| 2 | ghfast.top | URL前缀代理 | clone + Release + raw | ★★★ |
| 3 | mirror.ghproxy.com | URL前缀代理 | clone + Release + raw | ★★ |
| 4 | hub.gitmirror.com | URL前缀代理 | clone + Release | ★★ |

### gh-proxy.com

```
wget https://gh-proxy.com/https://github.com/owner/repo/releases/download/v1.0/file.tar.gz

# 克隆仓库
git clone https://gh-proxy.com/https://github.com/owner/repo.git

# raw 文件
wget https://gh-proxy.com/https://raw.githubusercontent.com/owner/repo/main/README.md
```

支持 Release / raw / archive / git clone 全场景。实测速度 10-50 MB/s（国内移动 / 联通 / 电信均测过，移动最快）。

### ghfast.top

```
wget https://ghfast.top/https://github.com/neovim/neovim/releases/download/stable/nvim-linux-x86_64.tar.gz
```

备用域名集合站：[https://ghproxy.link/](https://ghproxy.link/) —— 主域名失效时自动切换可用镜像。

### mirror.ghproxy.com

```
git clone https://mirror.ghproxy.com/https://github.com/InternLM/InternLM.git
```
功能与 gh-proxy.com 一致，稳定性偶尔波动。

### hub.gitmirror.com

```
git clone https://hub.gitmirror.com/https://github.com/owner/repo.git
```
适合国内服务器拉取镜像。

### 全局配置 url.insteadOf（一次配置永久生效）

无需每次手动加前缀，配置后所有 `git clone https://github.com/...`​ 自动走镜像：

```
git config --global url."https://gitclone.com/".insteadOf "https://"
git config --global url."https://hub.gitmirror.com/https://".insteadOf "https://"
```

按目录隔离配置：

```
# ~/.gitconfig
[includeIf "gitdir:~/work/"]
  path = ~/.gitconfig-work

# ~/.gitconfig-work
[url "https://hub.gitmirror.com/https://"]
  insteadOf = https://
```

> 注意：`insteadOf "https://"`​ 会接管所有 https 地址，仅在确认无冲突的环境使用。

### gitclone.com（仅 clone 场景）

```
git clone https://gitclone.com/github.com/owner/repo.git
# 注意：此处为 gitclone.com/github.com/，非 gitclone.com/https://github.com/
```

## 方法二：DNS 指定（hosts）

原理：将 `github.com`​、`raw.githubusercontent.com`​ 等域名直接绑定到可用 IP，绕过被污染的 DNS 解析。

### GitHub520

macOS / Linux 一次性更新：

```
sudo curl -o /tmp/hosts https://raw.hellogithub.com/hosts
sudo cat /tmp/hosts >> /etc/hosts
```

Windows 使用 [SwitchHosts](https://github.com/oldj/SwitchHosts) 订阅 [https://raw.hellogithub.com/hosts](https://raw.hellogithub.com/hosts)。

注意：hosts 仅对直连流量生效，全局代理下由代理接管，hosts 不参与解析。

## 方法三：Watt Toolkit（原 Steam++）

图形化工具，开箱即用：[瓦特工具箱官网](https://steampp.net/)。

不仅可以加速 Github 还能一键加速 Steam 商店。

## 总结

| 场景 | 推荐 |
| ---- | ---- |
| 临时下载/克隆 | 方法一：URL前缀代码 |
| 全局生效、免手动加前缀 | url.insteadOf配置 |
| 仅 clone | gitclone.com |
| 图形化操作 | 方法三：Watt Toolkit |
| 网页/ git / 下载整体提速 | 方法二：hosts(GitHub520) |