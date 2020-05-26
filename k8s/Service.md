# Service



- Service:
  - 模型：userspace，iptables，ipvs
  - ClusterIP，NodePort，，，
    - NodePort:client -> NodeIP:NodePort->ClusterIP:ServicePort->PodIP:containerPort
    - LoadBalancer
    - ExternelName
      - FQDN
        - CNAME->FQDN
  - No ClusterIP:Headless Service
    - ServiceName->PodIP

