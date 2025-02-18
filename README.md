# kube-monitoring-stack

kubernetes cluster 모니터링
---
#### 1) 프로메테우스 메트릭 - Mimir 저장 환경
- ~~kube-state-metrics install (helm chart)~~
- ~~node-exporter install (helm chart)~~
- 프로메테우스 helm charts의 kube-state-metric, node-exporter와 기본 수집 메트릭 설정 적용
- prometheus install (node-exporter, state-metrics, mimir remote push 설정)
- mimir 설치, 프로메테우스 push 설정
   ```
   # prometheus.yaml
   remoteWrite:
     - url: http://mimir-nginx.monitoring.svc:80/api/v1/push
   ```
- grafana install (메트릭 확인용)
   ```
   # grafana datasource
   datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      url: http://prometheus-server.monitoring:80      
      isDefault: true
      editable: true
    - name: Mimir
      type: prometheus
      url: http://mimir-nginx.monitoring.svc:80/prometheus
      editable: true
    ```
--- 
#### 2) 클러스터 모니터링 대시보드 
- 클러스터 정보 : 버전, 노드 수 , CPU, 메모리, 디스크, 평균 사용량
- 네임스페이스 수, 목록, 디플로이먼트, 스테이트풀셋, 데몬셋 정보
- 서비스, Ingress 정보
  ```
  # https://github.com/dotdc/grafana-dashboards-kubernetes
  dashboards:
    default:
      k8s_global:
        gnetId: 15757
        revision: 1
        datasource: Mimir
      k8s_nodes:
        gnetId: 15759
        revision: 1
        datasource: Mimir  
      k8s_namespaces:
        gnetId: 15758
        revision: 1
        datasource: Mimir
      k8s_pods:
        gnetId: 15760
        revision: 1
        datasource: Mimir
  ```
=======
- https://github.com/dotdc/grafana-dashboards-kubernetes
---
#### 3) Opentelemetry Collector - Mimir 메트릭 수집 구성 (Working)
- Opentelemetry를 이용한 메트릭 수집 설정
  - Opentelemtry Connector helm chart 추가 (chats/opentelemetry-collector)
  - Mimir 엔드포인트 설정
  ```
  config:
    exporters:
      debug:
        verbosity: detailed
      otlphttp:
        endpoint: http://k8s-monitoring-mimir-nginx.monitoring.svc:80/otlp
  ```
  - OTLP receiver 설정

- Loki, Tempo 저장소 구성
  - loki 추가 - loki-simple-scalable
  - mimir-distributed/minio bucket create 추가
  ```
    loki:
    auth_enabled: false
    storage:
      bucketNames:
        chunks: loki-chunks
        ruler: loki-ruler
        admin: loki-admin
      type: s3
      s3:      
        endpoint: http://k8s-monitoring-minio:9000
        secretAccessKey: console123
        accessKeyId: console
        s3ForcePathStyle: true
        insecure: true
  ```

  - temp chart 추가 : temp chart
  - metric generation > mimir 로 저장
  - s3 bucket tempo-store에 데이터 저장
  ```
  tempo:
    nameOverride: "k8s-monitoring-tempo"
    tempo:
      metricsGenerator:
        enabled: true
        remoteWriteUrl: "http://k8s-monitoring-mimir-nginx.monitoring.svc:80/api/v1/push"
      storage:
        trace:
          backend: s3
          s3:
            bucket: tempo-store   
            endpoint: "k8s-monitoring-minio:9000"  # http:// 제외해야 정상 동작
            access_key: console 
            secret_key: console123
            insecure: true
            forcepathstyle: true       
  ```

  - 그라파나 Datassource 설정
    - loki: http://k8s-monitoring-loki-distributed-gateway
    - tempo: http://k8s-monitoring-k8s-monitoring-tempo:3100
    - mimir: http://k8s-monitoring-mimir-nginx.monitoring.svc:80/prometheus
    - prometheus: http://k8s-monitoring-prometheus-server.monitoring:80
  - Trace ID - Exemplar 연동 구성
  - Opentelemetry Connector와 연동 구성

  