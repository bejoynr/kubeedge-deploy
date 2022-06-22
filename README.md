## k8s Cluster deployment
```
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
> net.bridge.bridge-nf-call-iptables  = 1
> net.ipv4.ip_forward                 = 1
> net.bridge.bridge-nf-call-ip6tables = 1
> EOF

 sudo sysctl --system

 mkdir /etc/containerd
 sudo containerd config default | sudo tee /etc/containerd/config.toml

 sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml


kubeadm init --apiserver-advertise-address=172.16.1.7 

`[init] Using Kubernetes version: v1.24.2`
`Your Kubernetes control-plane has initialized successfully!`
`To start using your cluster, you need to run the following as a regular user:`

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

```

## Apply CNI

```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

```

## Add SAN to API server certs - not needed if cluster was deployed with public IP

```
 kubectl get configmap -n kube-system kubeadm-config -o jsonpath='{.data.ClusterConfiguration}' > kubeadm-config.yaml

 vi kubeadm-config.yaml

apiServer:
  certSANs:
  - 10.96.0.1
  - 172.16.1.7
  - 20.xxx.xxx.xxx
  extraArgs:
 .
 .
 mv /etc/kubernetes/pki/apiserver.{crt,key} ~

 kubeadm init phase certs apiserver --config kubeadm-config.yaml

 crictl pods | grep apiserver | cut -d' ' -f1

 export CONTAINER_RUNTIME_ENDPOINT="unix:///run/containerd/containerd.sock"

 crictl stop  -r unix:///run/containerd/containerd.sock d457ac5069aba4dad5901af0a4e140d42e928f5d50c362e41afee3d72f3a62d7


 ```

 ## Taint CP node

 ```
kubectl taint nodes --all node-role.kubernetes.io/control-plane- node-role.kubernetes.io/master-
 ```

 ## Install Kubeedge binaries

 ```
wget https://github.com/kubeedge/kubeedge/releases/download/v1.11.0/keadm-v1.11.0-linux-amd64.tar.gz

 wget https://github.com/kubeedge/kubeedge/releases/download/v1.11.0/kubeedge-v1.11.0-linux-amd64.tar.gz

  git clone https://github.com/kubeedge/kubeedge.git

  wget https://get.helm.sh/helm-v3.9.0-linux-amd64.tar.gz

  root@azvm01:~/Kube-edge/kubeedge/manifests/charts# helm upgrade --install cloudcore ./cloudcore  -f ./cloudcore/values.yaml --namespace kubeedge --debug --dry-run --create-namespace --set cloudCore.modules.cloudHub.advertiseAddress[0]=20.xxx.186.xx
  ```
  ## Helm debug info
<details>
  <summary>Debug info</summary>

```yaml

    NAME: cloudcore
    LAST DEPLOYED: Tue Jun 21 06:27:30 2022
    NAMESPACE: kubeedge
    STATUS: pending-install
    REVISION: 1
    TEST SUITE: None
    USER-SUPPLIED VALUES:
    appVersion: 1.9.1
    cloudCore:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/edge
                operator: DoesNotExist
      annotations: {}
      hostNetWork: "true"
      image:
        pullPolicy: IfNotPresent
        pullSecrets: []
        repository: kubeedge/cloudcore
        tag: v1.9.1
      labels:
        k8s-app: kubeedge
        kubeedge: cloudcore
      modules:
        cloudHub:
          advertiseAddress:
          - 20.223.186.57
          dnsNames:
          - ""
          https:
            enable: "true"
          nodeLimit: "1000"
          quic:
            enable: "false"
            maxIncomingStreams: "10000"
          websocket:
            enable: "true"
        cloudStream:
          enable: "true"
        dynamicController:
          enable: "false"
        router:
          enable: "false"
      nodeSelector: {}
      replicaCount: 1
      resources:
        limits:
          cpu: 200m
          memory: 1Gi
        requests:
          cpu: 100m
          memory: 512Mi
      securityContext:
        privileged: true
      service:
        annotations: {}
        cloudhubHttpsNodePort: "30002"
        cloudhubNodePort: "30000"
        cloudhubQuicNodePort: "30001"
        cloudstreamNodePort: "30003"
        enable: "true"
        tunnelNodePort: "30004"
        type: NodePort
      tolerations: {}
    iptablesManager:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/edge
                operator: DoesNotExist
      annotations: {}
      enable: "true"
      hostNetWork: true
      image:
        pullPolicy: IfNotPresent
        pullSecrets: []
        repository: kubeedge/iptables-manager
        tag: v1.9.1
      labels:
        k8s-app: iptables-manager
        kubeedge: iptables-manager
      mode: internal
      nodeSelector: {}
      resources:
        limits:
          cpu: 200m
          memory: 50Mi
        requests:
          cpu: 100m
          memory: 25Mi
      securityContext:
        capabilities:
          add:
          - NET_ADMIN
          - NET_RAW
      tolerations: {}
    
    COMPUTED VALUES:
    appVersion: 1.9.1
    cloudCore:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/edge
                operator: DoesNotExist
      annotations: {}
      hostNetWork: "true"
      image:
        pullPolicy: IfNotPresent
        pullSecrets: []
        repository: kubeedge/cloudcore
        tag: v1.9.1
      labels:
        k8s-app: kubeedge
        kubeedge: cloudcore
      modules:
        cloudHub:
          advertiseAddress:
          - 20.223.186.57
          dnsNames:
          - ""
          https:
            enable: "true"
          nodeLimit: "1000"
          quic:
            enable: "false"
            maxIncomingStreams: "10000"
          websocket:
            enable: "true"
        cloudStream:
          enable: "true"
        dynamicController:
          enable: "false"
        router:
          enable: "false"
      nodeSelector: {}
      replicaCount: 1
      resources:
        limits:
          cpu: 200m
          memory: 1Gi
        requests:
          cpu: 100m
          memory: 512Mi
      securityContext:
        privileged: true
      service:
        annotations: {}
        cloudhubHttpsNodePort: "30002"
        cloudhubNodePort: "30000"
        cloudhubQuicNodePort: "30001"
        cloudstreamNodePort: "30003"
        enable: "true"
        tunnelNodePort: "30004"
        type: NodePort
      tolerations: {}
    iptablesManager:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/edge
                operator: DoesNotExist
      annotations: {}
      enable: "true"
      hostNetWork: true
      image:
        pullPolicy: IfNotPresent
        pullSecrets: []
        repository: kubeedge/iptables-manager
        tag: v1.9.1
      labels:
        k8s-app: iptables-manager
        kubeedge: iptables-manager
      mode: internal
      nodeSelector: {}
      resources:
        limits:
          cpu: 200m
          memory: 50Mi
        requests:
          cpu: 100m
          memory: 25Mi
      securityContext:
        capabilities:
          add:
          - NET_ADMIN
          - NET_RAW
      tolerations: {}
    
    HOOKS:
    ---
    # Source: cloudcore/templates/secret_cloudcore.yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: cloudcore
      labels:
        k8s-app: kubeedge
        kubeedge: cloudcore
      annotations:
        "helm.sh/hook": "pre-install"
        "helm.sh/hook-delete-policy": "before-hook-creation"
    data:
      streamCA.crt:     
    MANIFEST:
    ---
    # Source: cloudcore/templates/rbac_cloudcore.yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      labels:
        k8s-app: kubeedge
        kubeedge: cloudcore
      name: cloudcore
    ---
    # Source: cloudcore/templates/configmap_cloudcore.yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: cloudcore
      labels:
        k8s-app: kubeedge
        kubeedge: cloudcore
    data:
      cloudcore.yaml: |
        apiVersion: cloudcore.config.kubeedge.io/v1alpha2
        kind: CloudCore
        kubeAPIConfig:
          kubeConfig: ""
          master: ""
        modules:
          cloudHub:
            advertiseAddress:
            - xx.xxx.xxx.xx
            dnsNames:
            -
            nodeLimit: 1000
            tlsCAFile: /etc/kubeedge/ca/rootCA.crt
            tlsCertFile: /etc/kubeedge/certs/edge.crt
            tlsPrivateKeyFile: /etc/kubeedge/certs/edge.key
            unixsocket:
              address: unix:///var/lib/kubeedge/kubeedge.sock
              enable: true
            websocket:
              address: 0.0.0.0
              enable: true
              port: 10000
            quic:
              address: 0.0.0.0
              enable: false
              maxIncomingStreams: 10000
              port: 10001
            https:
              address: 0.0.0.0
              enable: true
              port: 10002
          cloudStream:
            enable: true
            streamPort: 10003
            tunnelPort: 10004
          dynamicController:
            enable: false
          router:
            enable: false
          iptablesManager:
            enable: true
            mode: internal
    ---
    # Source: cloudcore/templates/rbac_cloudcore.yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: cloudcore
      labels:
        k8s-app: kubeedge
        kubeedge: cloudcore
    rules:
    - apiGroups: [""]
      resources: ["nodes", "nodes/status", "serviceaccounts/token", "configmaps",     "pods", "pods/status", "secrets", "endpoints", "services",     "persistentvolumes", "persistentvolumeclaims"]
      verbs: ["get", "list", "watch", "create", "update"]
    - apiGroups: [""]
      resources: ["namespaces"]
      verbs: ["get", "create", "list", "watch"]
    - apiGroups: [""]
      resources: ["nodes", "pods/status"]
      verbs: ["patch"]
    - apiGroups: [""]
      resources: ["pods"]
      verbs: ["delete"]
    - apiGroups: ["coordination.k8s.io"]
      resources: ["leases"]
      verbs: ["get", "list", "watch", "create", "update"]
    - apiGroups: ["devices.kubeedge.io"]
      resources: ["devices", "devicemodels", "devices/status", "devicemodels/    status"]
      verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
    - apiGroups: ["reliablesyncs.kubeedge.io"]
      resources: ["objectsyncs", "clusterobjectsyncs", "objectsyncs/status",     "clusterobjectsyncs/status"]
      verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
    - apiGroups: ["rules.kubeedge.io"]
      resources: ["rules", "ruleendpoints", "rules/status", "ruleendpoints/status"]
      verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
    - apiGroups: ["apiextensions.k8s.io"]
      resources: ["customresourcedefinitions"]
      verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
    - apiGroups: ["networking.istio.io"]
      resources: ["*"]
      verbs: ["get", "list", "watch"]
    ---
    # Source: cloudcore/templates/rbac_cloudcore.yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: cloudcore
      labels:
        k8s-app: kubeedge
        kubeedge: cloudcore
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cloudcore
    subjects:
    - kind: ServiceAccount
      name: cloudcore
      namespace: kubeedge
    ---
    # Source: cloudcore/templates/service_cloudcore.yaml
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        k8s-app: kubeedge
        kubeedge: cloudcore
      name: cloudcore
    spec:
      type: ClusterIP
      ports:
      - port: 10000
        targetPort: 10000
        name: cloudhub
      - port: 10001
        targetPort: 10001
        name: cloudhub-quic
      - port: 10002
        targetPort: 10002
        name: cloudhub-https
      - port: 10003
        targetPort: 10003
        name: cloudstream
      - port: 10004
        targetPort: 10004
        name: tunnelport
      selector:
        k8s-app: kubeedge
        kubeedge: cloudcore
    ---
    # Source: cloudcore/templates/deployment_cloudcore.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        k8s-app: kubeedge
        kubeedge: cloudcore
      name: cloudcore
    spec:
      replicas: 1
      selector:
        matchLabels:
          k8s-app: kubeedge
          kubeedge: cloudcore
      template:
        metadata:
          labels:
            k8s-app: kubeedge
            kubeedge: cloudcore
        spec:
          hostNetwork: true
          dnsPolicy: ClusterFirstWithHostNet
          restartPolicy: Always
          affinity:
            nodeAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                - matchExpressions:
                  - key: node-role.kubernetes.io/edge
                    operator: DoesNotExist
          serviceAccount: cloudcore
          containers:
          - name: cloudcore
            image: kubeedge/cloudcore:v1.9.1
            imagePullPolicy: IfNotPresent
            ports:
            - containerPort: 10000
              name: cloudhub
              protocol: TCP
            - containerPort: 10001
              name: cloudhub-quic
              protocol: TCP
            - containerPort: 10002
              name: cloudhub-https
              protocol: TCP
            - containerPort: 10003
              name: cloudstream
              protocol: TCP
            - containerPort: 10004
              name: tunnelport
              protocol: TCP
            volumeMounts:
            - name: conf
              mountPath: /etc/kubeedge/config
            - name: certs
              mountPath: /etc/kubeedge
            - name: sock
              mountPath: /var/lib/kubeedge
            - mountPath: /etc/localtime
              name: host-time
              readOnly: true
            securityContext:
              privileged: true
            resources:
              limits:
                cpu: 200m
                memory: 1Gi
              requests:
                cpu: 100m
                memory: 512Mi
          volumes:
          - name: conf
            configMap:
              name: cloudcore
          - name: certs
            secret:
              secretName: cloudcore
              items:
              - key: stream.crt
                path: certs/stream.crt
              - key: stream.key
                path: certs/stream.key
              - key: streamCA.crt
                path: ca/streamCA.crt
          - name: sock
            hostPath:
              path: /var/lib/kubeedge
              type: DirectoryOrCreate
          - hostPath:
              path: /etc/localtime
              type: ""
            name: host-time
```
</details>

## Generate the secret and apply

```
certgen.sh buildSecret | tee -a 06-secret.yaml

kubectl  apply -f 06-secret.yaml

keadm gettoken --kube-config /root/.kube/config

```
## On edge node

```
sudo ./keadm join --cloudcore-ipport=20.223.186.57:10000 --token=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx --edgenode-name=pi4-home --kubeedge-version=1.11.0 --tarballpath=./

```
<details>
  <summary>Details of execution</summary>
  ```  
  I0621 13:23:19.107991   23287 command.go:845] 1. Check KubeEdge edgecore     process status
  I0621 13:23:19.163661   23287 command.go:845] 2. Check if the management     directory is clean
  I0621 13:23:19.164064   23287 join.go:100] 3. Create the necessary directories
  I0621 13:23:19.165947   23287 join.go:176] 4. Pull Images
  Pulling kubeedge/installation-package:v1.11.0 ...
  Pulling eclipse-mosquitto:1.6.15 ...
  Pulling kubeedge/pause:3.1 ...
  I0621 13:23:19.220245   23287 join.go:176] 5. Copy resources from the image to     the  management directory
  I0621 13:23:22.674640   23287 join.go:176] 6. Start the default mqtt service
  I0621 13:23:23.849822   23287 join.go:100] 7. Generate systemd service file
  I0621 13:23:23.850465   23287 join.go:100] 8. Generate EdgeCore default     configuration
  I0621 13:23:23.850605   23287 join.go:230] The configuration does not exist or     the parsing  fails, and the default configuration is generated
  I0621 13:23:23.860362   23287 join.go:100] 9. Run EdgeCore daemon
  I0621 13:23:27.498414   23287 join.go:317]
  I0621 13:23:27.498475   23287 join.go:318] KubeEdge edgecore is running, For     logs visit:  
  ```
</details>


## Check journal logs
```
journalctl -u edgecore.service -xe -f

sudo systemctl  status edgecore

```

## Check status of edge node in cluster
```
$ k get node
NAME       STATUS   ROLES           AGE     VERSION
azvm01     Ready    control-plane   6h9m    v1.24.2
pi4-home   Ready    agent,edge      3h27m   v1.22.6-kubeedge-v1.11.0


$ cat nginx-depoyment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: debian-deployment
  labels:
    app: debian
spec:
  replicas: 1
  selector:
    matchLabels:
      app: debian
  template:
    metadata:
      labels:
        app: debian
    spec:
      containers:
      - name: debian
        image: debian
```
## Apply the YAML

```
kubectl apply -f nginx.yaml

root@azvm01:~# k get po -A -o wide
NAMESPACE     NAME                               READY   STATUS    RESTARTS         AGE     IP             NODE       NOMINATED NODE   READINESS GATES
.
.
.
kubeedge      nginx-deployment-5878dd8b8-7f27h   1/1     Running   0                3h9m    172.17.0.3     pi4-home              

```

## Avoid kube-proxy being deployed on edgenode

<details>
  <summary>Solution</summary>

  Edit the `kubectl edit daemonsets.apps -n kube-system kube-proxy` file to include `affinity section` under `spec`.

  ```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  annotations:
    deprecated.daemonset.template.generation: "2"
  labels:
    k8s-app: kube-proxy
  name: kube-proxy
  namespace: kube-system
spec:
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kube-proxy
  template:
    metadata:
      creationTimestamp: null
      labels:
        k8s-app: kube-proxy
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/edge
                operator: DoesNotExist
      containers:
      - command:
        - /usr/local/bin/kube-proxy
        - --config=/var/lib/kube-proxy/config.conf
        - --hostname-override=$(NODE_NAME)
        env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: spec.nodeName
        image: k8s.gcr.io/kube-proxy:v1.24.2
        imagePullPolicy: IfNotPresent
        name: kube-proxy
        securityContext:
          privileged: true
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /var/lib/kube-proxy
          name: kube-proxy
        - mountPath: /run/xtables.lock
          name: xtables-lock
        - mountPath: /lib/modules
          name: lib-modules
          readOnly: true
      dnsPolicy: ClusterFirst
      hostNetwork: true
      nodeSelector:
        kubernetes.io/os: linux
      priorityClassName: system-node-critical
      restartPolicy: Always
      schedulerName: default-scheduler
      serviceAccount: kube-proxy
      serviceAccountName: kube-proxy
      terminationGracePeriodSeconds: 30
      tolerations:
      - operator: Exists
      volumes:
      - configMap:
          defaultMode: 420
          name: kube-proxy
        name: kube-proxy
      - hostPath:
          path: /run/xtables.lock
          type: FileOrCreate
        name: xtables-lock
      - hostPath:
          path: /lib/modules
          type: ""
        name: lib-modules
  updateStrategy:
    rollingUpdate:
      maxSurge: 0
      maxUnavailable: 1
    type: RollingUpdate
  ```
</details>
