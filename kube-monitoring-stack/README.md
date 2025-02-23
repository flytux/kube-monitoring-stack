### K8S 모니터링 차트 설치

---

```
# 기본 스토리지 클래스 설정 필요
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.31/deploy/local-path-storage.yaml
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

# k8s-monitoring 으로 release 이름 변경 없이 구성
# prometheus, mimir, minio, loki, tempo, grafana, opentelemetry collector로 구성

helm upgrade -i k8s-monitoring . -n monitoring --create-namespace
```
