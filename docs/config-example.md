```yaml
apiVersion: kubekey.kubesphere.io/v1alpha2
kind: Cluster
metadata:
  name: sample
spec:
  hosts:
  # 假设SSH默认端口为22，否则在IP地址后面加上端口号。 
  # 如果您在 ARM 上安装 Kubernetes，请添加“arch: arm64”。例如，{...用户：ubuntu，密码：Qcloud@123，arch：arm64}。
  - {name: node1, address: 172.16.0.2, internalAddress: 172.16.0.2, port: 8022, user: ubuntu, password: "Qcloud@123"}
  # 对于默认的 root 用户。
  # Kubekey 将解析 `labels` 字段并自动标记节点。
  - {name: node2, address: 172.16.0.3, internalAddress: 172.16.0.3, password: "Qcloud@123", labels: {disk: SSD, role: backend}}
  # 对于使用 SSH 密钥的无密码登录。
  - {name: node3, address: 172.16.0.4, internalAddress: 172.16.0.4, privateKeyPath: "~/.ssh/id_rsa"}
  roleGroups:
    etcd:
    - node1 # 集群中作为 etcd 节点的所有节点。
    master:
    - node1
    - node[2:10] # 从节点 2 到节点 10。集群中充当主节点的所有节点。
    worker:
    - node1
    - node[10:100] # 集群中所有充当工作节点的节点。
  controlPlaneEndpoint:
    #apiservers 的内部负载均衡器。支持：haproxy、kube-vip [Default ""]
    internalLoadbalancer: haproxy 
    domain: lb.kubesphere.local
    # 负载均衡器的 IP 地址。如果您在“kube-vip”模式下使用 internalLoadblancer，则此处需要一个 VIP。
    address: ""      
    port: 6443
  system:
    # chrony 的 ntp 服务器。
    ntpServers:
      - time1.cloud.tencent.com
      - ntp.aliyun.com
      - node1 # 如果没有公共 ntp 服务器访问，则将 `hosts` 中的节点名称设置为 ntp 服务器。
    timezone: "Asia/Shanghai"
    # 指定要安装的附加包。需要工件中包含的 ISO 文件。
    rpms:
      - nfs-utils
    #指定要安装的附加包。需要工件中包含的 ISO 文件。
    debs: 
      - nfs-common
    #preInstall: # 为每个节点指定自定义的init shell脚本，并按照列表顺序执行。
    # -name: 格式化并挂载磁盘
    # bash: /bin/bash -x setup-disk.sh
    # materials: # 脚本可以有一些依赖材料。那些将复制到节点
    # -./setup-disk.sh #shell执行需要的脚本
    # -xxx # 本脚本需要的其他工具材料
    # postInstall: # 在 kubernetes 安装后为每个节点指定自定义完成清理 shell 脚本。
    # -名称：清理 tmps 文件
    # 庆典： |
    # rm -fr /tmp/kubekey/*
    #skipConfigureOS: true # 不要预先配置主机操作系统（例如内核模块、/etc/hosts、sysctl.conf、NTP 服务器等）。在使用 KubeKey 之前，您必须通过其他方法设置这些东西。

  kubernetes:
    version: v1.21.5
    # 可选的额外主题备用名称 (SAN) 用于 API 服务器服务证书。可以是 IP 地址和 DNS 名称。
    apiserverCertExtraSans:  
      - 192.168.8.8
      - lb.kubespheredev.local
    # 容器运行时，支持：containerd、cri-o、isula。 [Default: docker]
    containerManager: docker
    clusterName: cluster.local
    # 是否安装可以自动更新 Kubernetes 控制平面证书的脚本。 [Default: false]
    autoRenewCerts: true
    # 如果使用纯 iptables 代理模式，masqueradeAll 告诉 kube-proxy 对所有内容进行 SNAT。[Default: false].
    masqueradeAll: false
   # maxPods 是可以在这个 Kubelet 上运行的 Pod 的数量。 [Default: 110]
    maxPods: 110
    # podPidsLimit 是任何 pod 中 PID 的最大数量。 [Default: 10000]
    podPidsLimit: 10000
   # 内部网络节点大小分配。这是分配给网络上每个节点的大小。 [Default: 24]
    nodeCidrMaskSize: 24
   # 指定使用哪种代理模式。 [Default: ipvs]
    proxyMode: ipvs
    # 启用功能门, [Default: {"ExpandCSIVolumes":true,"RotateKubeletServerCertificate": true,"CSIStorageCapacity":true, "TTLAfterFinished":true}]
    featureGates: 
      CSIStorageCapacity: true
      ExpandCSIVolumes: true
      RotateKubeletServerCertificate: true
      TTLAfterFinished: true
    # 支持型和 NFD
    # kata:
    #   enabled: true
    # nodeFeatureDiscovery
    #   enabled: true
    # additional kube-proxy configurations
    kubeProxyConfiguration:
      ipvs:
        # 清理 IPVS 规则时要排除的 CIDR。
        # 当 internalLoadbalancer=kube-vip 和 proxyMode=ipvs 时，必须将节点 cidr 放在这里
        # 参考：https://github.com/kubesphere/kubekey/issues/1702
        excludeCIDRs:
          - 172.16.0.2/24
  etcd:
    # 指定集群使用的etcd类型。当集群类型为k3s时，设置该参数为kubeadm无效。 [kubekey | kubeadm | external] [Default: kubekey]
    type: kubekey  
   ## 下面的参数只有当type设置为external时才需要加上。
    ## caFile、certFile 和 keyFile 不需要设置，如果没有为现有的 etcd 启用 TLS 身份验证。
    # external:
    #   endpoints:
    #     - https://192.168.6.6:2379
    #   caFile: /pki/etcd/ca.crt
    #   certFile: /pki/etcd/etcd.crt
    #   keyFile: /pki/etcd/etcd.key
    dataDir: "/var/lib/etcd"
    # 心跳间隔的时间（以毫秒为单位）。
    heartbeatInterval: "250"
    # 选举超时的时间（以毫秒为单位）。
    electionTimeout: "5000"
    # 触发快照到磁盘的已提交事务数。
    snapshotCount: "10000"
    # 以小时为单位的 mvcc 键值存储的自动压缩保留。 0 表示禁用自动压缩。
    autoCompactionRetention: "8"
    # Set level of detail for etcd exported metrics, specify 'extensive' to include histogram metrics.
    metrics: basic
    ## Etcd has a default of 2G for its space quota. If you put a value in etcd_memory_limit which is less than
    ## etcd_quota_backend_bytes, you may encounter out of memory terminations of the etcd cluster. Please check
    ## etcd documentation for more information.
    # 8G is a suggested maximum size for normal environments and etcd warns at startup if the configured value exceeds it.
    quotaBackendBytes: "2147483648" 
    # Maximum client request size in bytes the server will accept.
    # etcd is designed to handle small key value pairs typical for metadata.
    # Larger requests will work, but may increase the latency of other requests
    maxRequestBytes: "1572864"
    # Maximum number of snapshot files to retain (0 is unlimited)
    maxSnapshots: 5
    # Maximum number of wal files to retain (0 is unlimited)
    maxWals: 5
    # Configures log level. Only supports debug, info, warn, error, panic, or fatal.
    logLevel: info
  network:
    plugin: calico
    calico:
      ipipMode: Always  # IPIP Mode to use for the IPv4 POOL created at start up. If set to a value other than Never, vxlanMode should be set to "Never". [Always | CrossSubnet | Never] [Default: Always]
      vxlanMode: Never  # VXLAN Mode to use for the IPv4 POOL created at start up. If set to a value other than Never, ipipMode should be set to "Never". [Always | CrossSubnet | Never] [Default: Never]
      vethMTU: 0  # The maximum transmission unit (MTU) setting determines the largest packet size that can be transmitted through your network. By default, MTU is auto-detected. [Default: 0]
    kubePodsCIDR: 10.233.64.0/18
    kubeServiceCIDR: 10.233.0.0/18
  storage:
    openebs:
      basePath: /var/openebs/local # base path of the local PV provisioner
  registry:
    registryMirrors: []
    insecureRegistries: []
    privateRegistry: ""
    namespaceOverride: ""
    auths: # if docker add by `docker login`, if containerd append to `/etc/containerd/config.toml`
      "dockerhub.kubekey.local":
        username: "xxx"
        password: "***"
        skipTLSVerify: false # Allow contacting registries over HTTPS with failed TLS verification.
        plainHTTP: false # Allow contacting registries over HTTP.
        certsPath: "/etc/docker/certs.d/dockerhub.kubekey.local" # Use certificates at path (*.crt, *.cert, *.key) to connect to the registry.
  addons: [] # You can install cloud-native addons (Chart or YAML) by using this field.

```
