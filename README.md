# Kyma_Ubuntu_Docker_Deployment 归档项目 (轻量版)

## 一、项目概况
- **平台**: SAP Kyma (AWS 托管)
- **核心功能**: 部署一个带持久化存储的 Ubuntu 环境，并配置 SSH 访问与网络放行。
- **关键技术**: 使用 AWS NLB 处理 UDP 流量，解决 K8s 默认负载均衡器不支持 UDP 的限制。

---

## 二、部署配置文件 (zvps-ubuntu-only.yaml)

你可以直接复制以下内容并在 Kyma UI 中应用：

apiVersion: apps/v1
kind: Deployment
metadata:
  name: zvps-super
  namespace: zvps-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: zvps-super
  template:
    metadata:
      labels:
        app: zvps-super
    spec:
      securityContext:
        fsGroup: 1000
      containers:
        - name: ubuntu-node
          image: ghcr.io/zv201413/zvps:latest 
          imagePullPolicy: Always
          env:
            - name: SSH_USER
              value: "zv"
            - name: SSH_PWD
              value: "设置你的自定义密码"
            - name: GB
              value: "true"
          resources:
            requests:
              cpu: "100m"
              memory: "512Mi"
          volumeMounts:
            - name: data-storage
              mountPath: /home/zv
      volumes:
        - name: data-storage
          persistentVolumeClaim:
            claimName: [你的实际PVC名称] # 请在 Kyma Storage 界面确认 PVC 名称

---

# TCP 服务：处理 SSH (端口 22) 和 Web 终端 (端口 80)
apiVersion: v1
kind: Service
metadata:
  name: zvps-svc-tcp
  namespace: zvps-system
spec:
  type: LoadBalancer
  selector:
    app: zvps-super
  ports:
    - name: ssh-port
      protocol: TCP
      port: 22
      targetPort: 22
    - name: ttyd-port
      protocol: TCP
      port: 80
      targetPort: 7681

---

# UDP 服务：专门为需要 UDP 的应用准备 (强制开启 NLB)
apiVersion: v1
kind: Service
metadata:
  name: zvps-svc-udp
  namespace: zvps-system
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb" # 必须加此注解以支持 UDP
spec:
  type: LoadBalancer
  selector:
    app: zvps-super
  ports:
    - name: port-2325
      protocol: UDP
      port: 2325
      targetPort: 2325
    - name: port-2326
      protocol: UDP
      port: 2326
      targetPort: 2326
  externalTrafficPolicy: Cluster

---

# 网络策略：确保集群防火墙放行指定端口
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-ingress
  namespace: zvps-system
spec:
  podSelector:
    matchLabels:
      app: zvps-super
  policyTypes:
    - Ingress
  ingress:
    - ports:
        - protocol: UDP
          port: 2325
        - protocol: UDP
          port: 2326
        - protocol: TCP
          port: 22
        - protocol: TCP
          port: 80

---

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

## 四、归档检查清单
- [x] Deployment 已关联正确的 PVC
- [x] Service 注解已添加，确保支持 UDP
- [x] NetworkPolicy 已放行 22, 80, 2325, 2326 端口