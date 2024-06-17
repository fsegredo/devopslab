Vault setup

this is not a production vault setup

> helm install vault hashicorp/vault \
       --set='server.dev.enabled=true' \
       --set='ui.enabled=true' \
       --namespace vault


Create policy names read-policy wuth reading capabilities

> cat <<EOF > /home/vault/read-policy.hcl
path "secret*" {
  capabilities = ["read"]
}
EOF

apply policy
> vault policy write read-policy /home/vault/read-policy.hcl

Enable k8s authentication
> vault auth enable kubernetes

Configure K8s Authentication
> vault write auth/kubernetes/config \
   token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
   kubernetes_host=https://${KUBERNETES_PORT_443_TCP_ADDR}:443 \
   kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt



    Breakdown:
        This configuration allows vault to authenticate with the K8s API Server using the service account token of the pod. This is necessary for Vault to verify the identify of k8s pods that are requesting secrets.

        - token_reviewer_jwt => Jwt token for authenticating the k8s API
        - kubernetes_host => URL of the kubernetes API Server
        - Kubernetes_ca_cert=> CA to verify the Kubernetes API's server's certificate


Create role vault-auth that binds the Vault policy to a kubernetes service account in a specific namespace
>  vault write auth/kubernetes/role/vault-role \
   bound_service_account_names=vault-serviceaccount \
   bound_service_account_namespaces=vault \
   policies=read-policy \
   ttl=1h

   TODO
   * creation of secrets
   * apploy deployment resource with vault secret injection