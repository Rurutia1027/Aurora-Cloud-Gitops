# How to Deploy Argo CD Application manifests 

- Make sure Argo CD is installed 
```bash 
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```


- Apply Our Apps Manifests 
```bash 
kubectl apply -f argo-applications/dev-app.yaml
kubectl apply -f argo-applications/prod-app.yaml
```
After that, Argo CD UI will show both applications, and we can sync or let automated sync handle updates 

**What's Next?**
- Confirm that the `aurora-prod` namespace exists and Istio injection is enabled 
- Make sure our overlay `kustomization.yaml` is correct with the `namespace:` field
- After pushing changes to our GitHub Repo, Argo CD will pick up updates automatically  
