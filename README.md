# MR_KOOK_BOT

CT Club KOOK 机器人 Linux 服务端客户端与版本发布。

## 一键部署到服务器

当前发布包适用于 **Ubuntu/Debian + Linux x86_64**。首次部署需要一台干净服务器，并通过 SSH 登录执行命令；不需要把 Windows 的 `CT-Club.exe` 上传到服务器。

要求：

- Ubuntu 22.04/24.04 或 Debian 12
- root 权限
- 服务器可以访问 GitHub、KOOK 和 `https://kooktx.mod.top`
- 对外只需开放 80/443；程序内部默认监听 `127.0.0.1:8000`

在服务器 SSH 终端直接粘贴下面整段命令，即可下载当前 Release、校验 SHA-256、解压并安装到 `/www/wwwroot/ct-club`：

```bash
set -Eeuo pipefail
VERSION="2026.10.02"
INSTALL_DIR="/www/wwwroot/ct-club"
WORK_DIR="$(mktemp -d -t ct-club-install-XXXXXX)"
trap 'rm -rf "$WORK_DIR"' EXIT
cd "$WORK_DIR"
BASE_URL="https://github.com/BOTBAKS/MR_KOOK_BOT/releases/download/v${VERSION}"
curl -fL --retry 3 "$BASE_URL/ct-club-server-${VERSION}.zip" -o ct-club-server.zip
curl -fL --retry 3 "$BASE_URL/SHA256SUMS-${VERSION}.txt" -o SHA256SUMS.txt
grep "  ct-club-server-${VERSION}.zip$" SHA256SUMS.txt > ct-club.sha256
sha256sum -c ct-club.sha256
unzip -q ct-club-server.zip -d package
sudo bash package/scripts/install.sh "$INSTALL_DIR"
```

如果当前 SSH 账号本身就是 root，也可以去掉最后一行命令前的 `sudo`。

安装脚本会自动：

1. 安装 Python、虚拟环境、FFmpeg、Supervisor 和服务端依赖；
2. 创建加密配置和 `/etc/ct-club/master.key`；
3. 创建 `ct-club` Supervisor 服务并启动；
4. 检查 `http://127.0.0.1:8000/healthz`，确认网页后台已就绪。

安装完成后访问：

```text
http://服务器IP:8000/
```

首次进入会创建一级管理员。随后在“在线授权”区域填写自己的 KOOK Bot Token 和卡密，激活成功后再配置服务器、管理员身份和插件。Bot Token、卡密和主密钥不会写入 GitHub Release。

如果要安装到其他目录，修改命令中的 `INSTALL_DIR`，例如 `/opt/ct-club`。不要将全新安装包直接覆盖已有线上目录；已有实例请使用后台“系统更新”，或先完整备份 `config/`、`data/`、插件配置和 `/etc/ct-club/master.key`。

## 宝塔/Nginx 部署

一键脚本会自动创建 Supervisor 服务。使用宝塔时：

1. 网站配置反向代理到 `http://127.0.0.1:8000`；
2. 外部只开放 80/443，不要把 8000 暴露到公网；
3. 申请 HTTPS 后通过域名访问后台；
4. 不要再用宝塔 Python 项目管理器启动第二个实例。

安装后可检查：

```bash
supervisorctl status ct-club
curl --fail http://127.0.0.1:8000/healthz
```

## 后续升级

安装完成后，登录网页控制台首页，系统会优先检查 GitHub 最新 Release；GitHub 不可达或附件下载失败时会尝试新服务器备用源。出现新版本时点击“立即更新”即可自动完成下载、SHA-256/Ed25519 校验、备份、替换、回滚保护和重启。

也可以手动查看版本：

```bash
cat /www/wwwroot/ct-club/release.json
```

## 重要说明

- 首次安装包是全新初始化包，不包含旧账号、数据库、插件配置、Bot Token 或授权文件。
- 插件业务数据和配置属于敏感数据，请定期备份；网页中的配置导出仅限一级管理员使用，可一键导出全部配置，也可按平台配置及每个插件的启停状态、设置和卡片文案分别选择。导出文件可能包含插件访问密钥。
- 只能运行一个服务进程和一个 worker，否则会重复连接 KOOK、重复执行定时任务。
- 如果机器人未上线，先检查卡密状态、Supervisor 日志和服务器到 KOOK/授权服务的出站网络。

详细的宝塔、Nginx、Playwright、数据迁移和故障排查说明见 [`docs/宝塔部署.md`](docs/宝塔部署.md)。

## Release

- 最新版本：[CT Club v2026.10.02](https://github.com/BOTBAKS/MR_KOOK_BOT/releases/tag/v2026.10.02)
- 仓库：[BOTBAKS/MR_KOOK_BOT](https://github.com/BOTBAKS/MR_KOOK_BOT)
