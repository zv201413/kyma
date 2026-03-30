# SAP Kyma Ubuntu 持久化环境部署

## 项目简介

本项目用于在 SAP BTP Kyma（AWS 托管 Kubernetes）上快速部署一个带持久化存储的 Ubuntu 环境，支持：
- **Web 终端访问** - 通过浏览器直接访问 Linux 环境
- **SSH 访问** - 标准 SSH 客户端连接
- **UDP 端口放行** - 支持 Hysteria 2、TUIC 等代理协议

---

## 文件说明

| 文件 | 说明 |
|------|------|
| `zvps-deployment.yaml` | 主部署文件，包含 Deployment、Service(TCP)、Service(UDP)、NetworkPolicy |
| `service.yaml` | UDP Service 单独配置（NLB 类型负载均衡器） |
| `NetworkPolicy.yaml` | 网络策略，放行 UDP 端口入站流量 |

---

## 部署步骤

### 1. 下载并修改配置文件

下载本项目文件，打开 `zvps-deployment.yaml` 进行修改：

<img width="934" height="1014" alt="image" src="https://github.com/user-attachments/assets/28a4b3b4-a5fe-43ad-a8a8-9879de9446f9" />

<img width="651" height="919" alt="image" src="https://github.com/user-attachments/assets/eb545d0c-f008-4de7-9f81-d6abd6a46349" />

修改以下环境变量：
```yaml
env:
- name: TTYD_OPTS
  value: "-c admin:yourpassword123"  # Web 终端登录密码
- name: SSH_USER
  value: "your_username"              # SSH 用户名
- name: SSH_PWD
  value: "your_password"              # SSH 密码
```

### 2. 创建 Kyma 命名空间

创建 Kyma 之后进入主控室：

<img width="1143" height="794" alt="image" src="https://github.com/user-attachments/assets/598a3208-8396-41bd-a6b7-bf2f16790672" />

上传 `zvps-deployment.yaml` 文件，等待后台自动化运行：

<img width="1515" height="714" alt="image" src="https://github.com/user-attachments/assets/957f2fd4-87fe-440a-9f60-5fa05684d465" />

等待状态变为 Active，点击 zvps-system：

<img width="987" height="853" alt="image" src="https://github.com/user-attachments/assets/31cad2bf-025b-4a50-b537-6f747ac7e223" />

<img width="1239" height="630" alt="image" src="https://github.com/user-attachments/assets/bc832ee1-06fe-47b3-82e9-00b8633b766d" />

### 3. 配置网络策略

进入之后在网络与发现关注两个子项：网络策略与服务

<img width="1295" height="836" alt="image" src="https://github.com/user-attachments/assets/72ac6890-7777-443a-8248-60e50bfc1870" />

先进入 Service，为了实现图示的最终效果，点击第二项 `zvps-svc-udp`：

<img width="1486" height="365" alt="image" src="https://github.com/user-attachments/assets/673ad2d8-c2cb-49b7-8904-e86f743ffc0b" />

按照图示依次编辑，把下载的 `service.yaml` 文件内容全部替换掉并保存（保存后页面会自动补全内容，继续点击保存即可）：

<img width="1388" height="730" alt="image" src="https://github.com/user-attachments/assets/e1614bbb-5170-489b-aef3-eb5bbdc6d328" />

同理进入 NetworkPolicy 进行编辑替换并保存下载的 `NetworkPolicy.yaml` 文件内容，此处默认开启了两个 UDP 端口，也可自行增删或修改：

<img width="1699" height="876" alt="image" src="https://github.com/user-attachments/assets/6cf6c06b-a22a-4644-b6d6-2991b5388993" />

---

## 访问方式

部署完成后，在 Services 页面查看 External IP：

| 服务 | 端口 | 用途 |
|------|------|------|
| `zvps-svc-tcp` | 22 | SSH 访问 |
| `zvps-svc-tcp` | 80 | Web 终端 |
| `zvps-svc-udp` | 2325 | Hysteria 2 |
| `zvps-svc-udp` | 2326 | TUIC |

**连接示例：**
```bash
# Web 终端
http://<External-IP>

# SSH 连接
ssh <SSH_USER>@<External-IP> -p 22
```

---

## 端口说明

默认放行以下 UDP 端口（可在 NetworkPolicy.yaml 中修改）：
- **2325** - Hysteria 2 协议端口
- **2326** - TUIC 协议端口

如需添加更多端口，编辑 `NetworkPolicy.yaml` 添加新的端口规则，并同步修改 `service.yaml`。

---

## 注意事项

1. UDP 服务必须使用 AWS NLB 负载均衡器（已配置 annotation）
2. PVC 需提前创建并确保状态为 `Bound`
3. 建议在生产环境中修改默认密码
4. External IP 通常为 AWS ELB/NLB 域名格式：`*.elb.amazonaws.com`

---

## 镜像来源

使用镜像：`ghcr.io/zv201413/zvps:latest`

---

## License

MIT
