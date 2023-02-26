This chart will deploy a MySQL instance with 3 replica with database credentials dynamically injected during deployment from Hashicorp vault.
MySQL is deployed from Bitnami Helm chart https://github.com/bitnami/charts/tree/main/bitnami/mysql

    Add the Hashicorp Helm repository:

#helm repo add hashicorp https://helm.releases.hashicorp.com

    Install the Vault Agent Injector Helm chart:

#helm install vault hashicorp/vault \
  --set "server.dev.enabled=true" \
  --set "injector.externalVaultAddr=http://vault.default.svc:8200" \
  --set "injector.authMethod=kubernetes" \
  --set "injector.listenAddr=:8080" \
  --set "injector.configFile=/vault/config/agent-inject.hcl"

    Create a Kubernetes Secret that contains the Vault AppRole credentials:

#kubectl create secret generic vault-approle \
  --from-literal=vault_role_id=<role_id> \
  --from-literal=vault_secret_id=<secret_id>


    Create a Vault policy that allows read access to the MySQL credentials:

vault policy write mysql-creds - <<EOF
path "database/creds/my-role" {
  capabilities = ["read"]
}
EOF

   Create a Kubernetes Secret that will hold the MySQL credentials:

#kubectl create secret generic mysql-creds

   Create values.yaml file with deployment parameters

   Install the MySQL Helm chart:

#helm install my-mysql bitnami/mysql -f values.yaml

  This will install MySQL with the specified values and enable the Vault Agent Injector with the specified configuration.

  Once the chart is installed, you can verify that the MySQL credentials have been populated in the mysql-creds Kubernetes.


