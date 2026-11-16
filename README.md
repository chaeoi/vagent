# vagent

给哪吒面板用的虚拟探针，当前按 Docker 使用。

使用时只读 `/config/config.yml`，不读取 `NZ_CLIENT_SECRET`、`NZ_SERVER` 这类环境变量。

## 运行

```bash
cat > config.yml <<'EOF'
Linux:
  client_secret: "replace_with_your_client_secret"
  server: "dashboard.example.com:443"
  tls: true
EOF

docker run -d \
  --name vagent \
  --restart unless-stopped \
  -v "$PWD/config.yml:/config/config.yml:ro" \
  niudea/vagent:latest
```

## 配置示例

```yaml
Linux:
  client_secret: "replace_with_your_client_secret"
  server: "dashboard.example.com:443"
  tls: true
  host:
    platform: Linux
    platform_version: "6.1"
    arch: amd64
    cpu: ["AMD EPYC 7B12 64-Core Processor"]
    mem_total: 4GB
    disk_total: 40GB

Windows:
  client_secret: "replace_with_your_client_secret"
  server: "dashboard.example.com:443"
  tls: true
  host:
    platform: Windows
    platform_version: "Server 2022"
    arch: amd64
    gpu: ["NVIDIA GeForce RTX 4090"]
    mem_total: 8GB
    disk_total: 80GB
```

完整示例见 [config.example.yml](/opt/codeserver/project/vagent/config.example.yml)。

## 版本与发布

推送到 `main` 后，GitHub Actions 自动编号、创建标签并发布，版本格式为 `年.月.发布序号`。
年月取 commit 提交时间的 UTC 年月；没有同月标签时从 `.1` 开始，后续取同月最大序号加一。
一次推送多个提交只为最后一个提交分配一次编号，首次发布前的历史提交不计数。
同一 commit 重跑沿用原编号，构建失败预留的编号不回收，因此未完成的发布可能留下空号。

发布任务排队执行。后续 commit 已有版本标签时，尚未编号的旧任务会跳过；已编号的旧版本可以重跑，
但不会覆盖更新的 `latest`。中断的草稿发布可继续完成，已公开的附件和已有版本镜像不覆盖。
Release 名称采用“项目名 版本号”，正文包含提交说明、使用入口和相邻版本变更；附件提供 `SHA256SUMS`。
构建时写入程序或包版本，不自动回写源码提交。开发副本使用开发版本标识。
