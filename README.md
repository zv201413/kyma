# Kyma_Ubuntu_Docker_Deployment

## 一、项目概况
- **平台**: SAP Kyma (AWS 托管)
- **核心功能**: 部署一个带持久化存储的 Ubuntu 环境，并配置 SSH 访问与网络放行。

---

## 二、准备工作与部署配置文件 (zvps-ubuntu-only.yaml)
创建kyma之后进入主控室
<img width="1143" height="794" alt="image" src="https://github.com/user-attachments/assets/598a3208-8396-41bd-a6b7-bf2f16790672" />
上传yaml，选择修改好的zvps-deployment.yaml文件，等待后台自动化运行
<img width="1515" height="714" alt="image" src="https://github.com/user-attachments/assets/957f2fd4-87fe-440a-9f60-5fa05684d465" />
等待状态变为Active，点击zvps-system
<img width="987" height="853" alt="image" src="https://github.com/user-attachments/assets/31cad2bf-025b-4a50-b537-6f747ac7e223" />

<img width="1239" height="630" alt="image" src="https://github.com/user-attachments/assets/bc832ee1-06fe-47b3-82e9-00b8633b766d" />
进入之后在网络与发现关注两个子项：网络策略与服务
<img width="1295" height="836" alt="image" src="https://github.com/user-attachments/assets/72ac6890-7777-443a-8248-60e50bfc1870" />
先进入service，为了实现图示的最终效果，点击第二项zvps-svc-udp
<img width="1486" height="365" alt="image" src="https://github.com/user-attachments/assets/673ad2d8-c2cb-49b7-8904-e86f743ffc0b" />
按照图示依次编辑，把下载的service.yaml文件内容全部替换掉并保存（保存后页面会自动补全内容，继续点击保存即可，以下同理）
<img width="1388" height="730" alt="image" src="https://github.com/user-attachments/assets/e1614bbb-5170-489b-aef3-eb5bbdc6d328" />
同理进入NetworkPolicy进行编辑替换并保存下载的NetworkPolicy.yaml文件内容，此处我默认开启了两个udp端口，也可自行增删或修改
<img width="1699" height="876" alt="image" src="https://github.com/user-attachments/assets/6cf6c06b-a22a-4644-b6d6-2991b5388993" />






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
