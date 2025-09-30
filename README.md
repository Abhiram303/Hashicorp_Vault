# Hashicorp_Vault
## Vault Integration

Here are the detailed steps for each of these steps:

## Create an AWS EC2 instance with Ubuntu

To create an AWS EC2 instance with Ubuntu, you can use the AWS Management Console or the AWS CLI. Here are the steps involved in creating an EC2 instance using the AWS Management Console:

- Go to the AWS Management Console and navigate to the EC2 service.
- Click on the Launch Instance button.
- Select the Ubuntu Server xx.xx LTS AMI.
- Select the instance type that you want to use.
- Configure the instance settings.
- Click on the Launch button.

## Install Vault on the EC2 instance

To install Vault on the EC2 instance, you can use the following steps:

**Install gpg**

```
sudo apt update && sudo apt install gpg
```

**Download the signing key to a new keyring**

```
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
```

**Verify the key's fingerprint**

```
gpg --no-default-keyring --keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg --fingerprint
```

The fingerprint must match 798A EC65 4E5C 1542 8C8E 42EE AA16 FCBC A621 E701, which can also be verified at https://www.hashicorp.com/security under "Linux Package Checksum Verification". Please note that there was a previous signing key used prior to January 23, 2023, which had the fingerprint E8A0 32E0 94D8 EB4E A189 D270 DA41 8C88 A321 9F7B. Details about this change are available on the status page: https://status.hashicorp.com/incidents/fgkyvr1kwpdh, https://status.hashicorp.com/incidents/k8jphcczkdkn.

**Add the HashiCorp repo**

```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
apt update!
```

```
@@ -82,22 +79,47 @@

2. **Create an AppRole**:

After enabling AppRole, you'll need to create an AppRole with appropriate policies and configure its authentication settings. Here are the steps to create an AppRole:
We need to create policy first,

```
vault policy write terraform - <<EOF
path "*" {
  capabilities = ["list", "read"]
}

path "secrets/data/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

path "kv/data/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}


path "secret/data/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

path "auth/token/create" {
capabilities = ["create", "read", "update", "list"]
}
EOF
```

Now you'll need to create an AppRole with appropriate policies and configure its authentication settings. Here are the steps to create an AppRole:

**a. Create the AppRole**:

```bash
vault write auth/approle/role/my-approle \
 policies="my-policy" \
 secret_id_ttl=10m \
 token_num_uses=10
vault write auth/approle/role/terraform \
    secret_id_ttl=10m \
    token_num_uses=10 \
    token_ttl=20m \
    token_max_ttl=30m \
    secret_id_num_uses=40 \
    token_policies=terraform
```

- `my-approle` is the name of your AppRole.
- `policies` specifies the policies that should be associated with the AppRole. Replace `my-policy` with the name of your desired policy.
- `secret_id_ttl` sets the TTL (time to live) for the Secret ID, which is the dynamic credential that Terraform will use for authentication. Adjust it to your requirements.
- `token_num_uses` sets the number of times the token created from this AppRole can be used. Adjust it as needed.

3. **Generate Role ID and Secret ID**:

After creating the AppRole, you need to generate a Role ID and Secret ID pair. The Role ID is a static identifier, while the Secret ID is a dynamic credential.
@@ -120,55 +142,4 @@
vault write -f auth/approle/role/my-approle/secret-id
   ```

This command generates a Secret ID and provides it in the response. Save the Secret ID securely, as it will be used for Terraform authentication.

4. **Terraform Configuration**:

In your Terraform configuration, you can use the Role ID and Secret ID to authenticate with Vault:

```hcl
provider "vault" {
 address = "http://<IP-Address>:8200" # Vault server address
 auth = {
   method = "approle"
   role_id = "YOUR_ROLE_ID"
   secret_id = "YOUR_SECRET_ID"
 }
}
```

Replace `YOUR_ROLE_ID` and `YOUR_SECRET_ID` with the actual Role ID and Secret ID generated in step 3.

With these steps, you've enabled and configured the AppRole authentication method in Vault, generated the necessary Role ID and Secret ID, and set up your Terraform configuration to use these credentials for authentication when interacting with Vault. This approach provides improved security and automation for your Terraform deployments.

To configure Terraform to read the secret from Vault, you need to add the following code to your Terraform configuration file:

```hcl
terraform {
required_providers {
vault = "~> 3.0"
}
}

provider "vault" {
address = "http://localhost:8200"
}

resource "vault_generic_secret" "my_secret" {
path = "my_secret"
}

variable "my_secret_value" {
value = vault_generic_secret.my_secret.value
}
```

## Apply the Terraform configuration.

Once you have configured Terraform, you can apply the configuration to create the resources in your infrastructure.

```
terraform apply
```

This will create a new resource called vault_generic_secret.my_secret. The value of this resource will be the value of the secret that we created in Vault.
This command generates a Secret ID and provides it in the response. Save the Secret ID securely, as it will be used for Terraform authentication.
