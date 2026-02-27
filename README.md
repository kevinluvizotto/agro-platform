agro-platform

Repositório de plataforma/infra do projeto AgroSolutions IoT (FIAP Tech Challenge – Fase 5).

Responsável por:
- Scripts e manifests para executar a solução localmente e/ou no AKS
- docker-compose para dependências (ex.: RabbitMQ)
- Manifests Kubernetes: namespaces, ingress, deployments, services, secrets
- Documentação de deploy e troubleshooting

ESTRUTURA SUGERIDA
agro-platform/
  docker-compose.yml
  k8s/
    namespaces/
    ingress/
    apps/
    secrets/
  scripts/

DEPENDÊNCIAS LOCAIS (RABBITMQ)
Subir RabbitMQ com docker-compose:
docker compose up -d

Acesso:
- AMQP: localhost:5672
- UI: http://localhost:15672

DEPLOY NO AKS (RESUMO)
1) Conectar ao cluster:
az aks get-credentials -g AgroSolutions-klztt-fiap -n aks-agro-fiap --overwrite-existing
kubectl get nodes

2) Namespaces:
kubectl create namespace agro --dry-run=client -o yaml | kubectl apply -f -
kubectl create namespace monitoring --dry-run=client -o yaml | kubectl apply -f -

3) Ingress Controller (NGINX):
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress ingress-nginx/ingress-nginx -n agro
kubectl get svc -n agro | grep ingress

4) Public IP / Host (nip.io):
EXTERNAL_IP=$(kubectl -n agro get svc ingress-ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
HOST="agro.${EXTERNAL_IP}.nip.io"
echo $EXTERNAL_IP
echo $HOST

5) Aplicar manifests das apps:
Aplique os manifests localizados em k8s/

TROUBLESHOOTING RÁPIDO
Pods:
kubectl get pods -n agro
kubectl logs -n agro deploy/agro-telemetry --tail=80

Health:
curl -i http://$EXTERNAL_IP/identity/health
curl -i http://$EXTERNAL_IP/properties/health
curl -i http://$EXTERNAL_IP/telemetry/health
curl -i http://$EXTERNAL_IP/alerts/health