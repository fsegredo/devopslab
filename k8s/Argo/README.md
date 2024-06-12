
in this scenario we're going to use argo as selfmanaged, which means we'll use argo to manage argo :)

Let's start by adding the hel repository for argo and install it
> helm repo add argo https://argoproj.github.io/argo-helm
>
> helm install argo argo/argo-cd -n argocd
