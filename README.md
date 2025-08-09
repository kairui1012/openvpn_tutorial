# OpenVPN 部署教程

一个详细的 OpenVPN 服务器部署指南，帮助您从零开始搭建自己的 VPN 服务。

## 简介

本项目提供了一个完整的 OpenVPN 服务器部署教程，适合初学者和有经验的系统管理员。教程涵盖了从服务器选择到完整配置的所有步骤。

## 特色

- **新手友好** - 详细的步骤说明，适合Linux初学者
- **全面覆盖** - 从服务器准备到最终配置的完整流程
- **实用导向** - 基于真实部署经验，提供实际可用的配置
- **多平台支持** - 主要针对Ubuntu，同时提供其他系统的适配说明

## 教程内容

### 第一步：准备云服务器
- 云服务商选择指南
- 服务器配置要求
- 端口设置和安全配置

### 第二步：配置 OpenVPN 服务器
- 软件安装
- 证书生成
- 服务器配置
- 服务启动和管理

### 涵盖的主题
- ✅ Ubuntu 22.04 LTS 系统配置
- ✅ DigitalOcean 云服务器设置
- ✅ 防火墙和端口配置
- ✅ OpenVPN 和 Easy-RSA 安装
- ✅ 服务启动和状态检查
- 🚧 证书生成和管理（开发中）
- 🚧 客户端配置（开发中）

## 快速开始

1. **查看完整教程**
   ```bash
   # 阅读主教程文件
   cat tutorial.md
   ```

2. **系统要求**
   - Ubuntu 22.04 LTS（推荐）
   - 最低 512MB 内存
   - 10GB 存储空间
   - 公网 IP 地址

3. **主要命令概览**
   ```bash
   # 更新系统
   sudo apt update
   
   # 安装 OpenVPN
   sudo apt install openvpn easy-rsa
   
   # 启动服务
   sudo systemctl start openvpn@server
   ```

## 文件结构

```
openvpn/
├── README.md           # 项目说明文档
├── tutorial.md         # 完整部署教程
├── tutorial.txt        # 教程草稿版本
├── docker-compose.yml  # Docker 容器化配置
└── Dockerfile          # Docker 镜像构建文件
```

## 支持的云服务商

| 云服务商 | 支持状态 | 说明 |
|----------|----------|------|
| DigitalOcean | ✅ 完全支持 | 主要教学平台 |
| AWS | ✅ 兼容 | 需要调整安全组设置 |
| 阿里云 | ✅ 兼容 | 需要调整安全组规则 |
| 腾讯云 | ✅ 兼容 | 需要调整防火墙设置 |
| Vultr | ✅ 兼容 | 配置方式类似 |

## 端口要求

确保以下端口在您的服务器上开放：

| 端口 | 协议 | 用途 |
|------|------|------|
| 22 | TCP | SSH 远程管理 |
| 80 | TCP | HTTP 服务（可选） |
| 443 | TCP | HTTPS 服务（可选） |
| 1194 | UDP | OpenVPN 数据传输 |

## Docker 部署（可选）

本项目也支持使用 Docker 进行快速部署：

```bash
# 使用 Docker Compose 启动
docker-compose up -d

# 查看服务状态
docker-compose ps
```

## 贡献指南

欢迎提交问题和改进建议！

1. Fork 本项目
2. 创建您的特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交您的修改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 开启一个 Pull Request

## 常见问题

### Q: 教程适合哪些用户？
A: 适合希望搭建个人 VPN 服务的用户，特别是 Linux 初学者。

### Q: 需要什么基础知识？
A: 基本的 Linux 命令行操作知识即可，教程会详细说明每个步骤。

### Q: 支持哪些操作系统？
A: 主要支持 Ubuntu 22.04 LTS，其他 Debian 系统也可参考使用。

### Q: 安装过程大约需要多长时间？
A: 完整安装通常需要 30-60 分钟，取决于网络速度和熟练程度。

## 许可证

本项目采用 MIT 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情。

## 联系方式

如果您有任何问题或建议，请通过以下方式联系：

- 创建 GitHub Issue
- 提交 Pull Request

## 致谢

感谢所有为本教程提供反馈和建议的用户。

---

**注意**: 本教程仅供学习和个人使用。请确保遵守您所在地区的相关法律法规。

