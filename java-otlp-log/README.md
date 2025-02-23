# Java Otel Agent to OTLP trace / log

1. docker build -t dice:v1 .
2. kubectl apply -f k8s/dice.yaml
3. generate traffic
4. http://grafana.local admin / admin
5. Go to "Explore"
6. Select "Tempo" trace to Log
