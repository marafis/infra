## One-time bootstrap
`kubectl apply -f bootstrap/root-app.yaml` 
1. [x] That's the last kubectl apply you'll ever run for this cluster.
 
## What Happens Next (Automatically)
ArgoCD picks up root-app.yaml
- └─► reads argocd/ folder
- └─► Wave 0: deploys MetalLB
- └─► Wave 1: deploys IP Pool + cert-manager (parallel)
- └─► Wave 2: deploys ingress-nginx → gets IP 94.130.107.251
- └─► Wave 3: deploys ClusterIssuer + triggers wildcard cert DNS01 challenge
- └─► cert-manager writes TXT record to Cloud DNS
- └─► Let's Encrypt validates → issues *.marafis.com + marafis.com
- └─► Secret wildcard-tls created in ingress-nginx namespace
- └─► Wave 4: deploys website → live at marafis.com with HTTPS ✅

## Verify cert issuance
``
kubectl get certificate -n ingress-nginx
``
``
kubectl describe certificate wildcard-tls -n ingress-nginx
``
 Look for: Status: True, Reason: Ready

## Adding a New App in the Future

1. Create your app folder
`mkdir myapp/`
// deployment.yaml + service.yaml + ingress.yaml (rules only, no tls:)

 2. Create one ArgoCD app file
`cp argocd/website-app.yaml argocd/myapp-app.yaml`
 Edit: name, path, namespace

 3. Push
`git push`

 Done. HTTPS at myapp.marafis.com in ~60 seconds.`


# newapp/ingress.yaml — this is ALL you need for HTTPS
spec:
ingressClassName: nginx
rules:
- host: app1.marafis.com   # HTTPS automatic, no tls: block needed