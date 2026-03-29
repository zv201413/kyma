# Kyma_Ubuntu_Docker_Deployment

## 一、项目概况
- **平台**: SAP Kyma (AWS 托管)
- **核心功能**: 部署一个带持久化存储的 Ubuntu 环境，并配置 SSH 访问与网络放行。

---

## 二、部署配置文件 (zvps-ubuntu-only.yaml)



## 三、搭建指南

### 1. 准备持久化存储 (PVC)
在 Kyma Dashboard 的 Storage -> Persistent Volume Claims 中，确认已经创建好了一个状态为 Bound 的 PVC。记下它的名称并填入 YAML 的 claimName 处。

### 2. 执行部署
将上述 YAML 内容应用到集群中。你可以使用 kubectl apply -f 或直接在 Kyma UI 界面粘贴。

### 3. 获取连接地址
部署完成后，通过以下 Service 获取外部 IP：
- SSH/Web 控制台: 查看 zvps-svc-tcp 的 External IP。
- UDP 应用服务: 查看 zvps-svc-udp 的 External IP（通常是以 .elb.amazonaws.com 结尾的 AWS 域名）。

### 4. 访问方式
- Web 终端: 浏览器访问 http://[External-IP] 即可进入环境。
- SSH 客户端: 使用命令 ssh zv@[External-IP] -p 22 进行连接。

---
