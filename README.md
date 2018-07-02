# vault-ldap-auth

Example of configuring HashiCorp Vault to use LDAP for authentication

![Final Environment](https://user-images.githubusercontent.com/3911650/42184445-c899755e-7e02-11e8-809a-5052c6dadfb4.png)

## Getting Started

Deploy the CloudFormation `infrastructure/cloudformation.json` template. The template creates a user with the following credentials and minimal required permisisons to complete the Lab:

- Username: _student_
- Password: _password_

## Instructions

1. In the Cloud9 environment terminal, install Vault:

    ```sh
    wget https://releases.hashicorp.com/vault/0.10.3/vault_0.10.3_linux_amd64.zip -O /tmp/vault.zip
    sudo unzip /tmp/vault.zip -d /usr/local/bin/
    ```

1. Start the Vault server in development mode in a new terminal tab:

    ```sh
    vault server -dev
    ```

1. In the original terminal tab, configure the Vault server address:

    ```sh
    export VAULT_ADDR='http://127.0.0.1:8200'
    ```

1. Create a file named Engineering.hcl with the following Vault policy as its contents:

    ```txt
    path "secret/data/Engineering" {
        capabilities = ["create", "read", "update", "delete", "list"]
    }
    ```

1. Write the policy into Vault:

    ```sh
    vault policy write engineering Engineering.hcl
    ```

1. Use the CA to sign the certificate in the CSR created earlier:

    ```sh
    sudo openssl ca -days 100 -notext -md sha256 -batch -in req.csr -keyfile ca/ca.key.pem -cert ca/ca.crt -out cert.pem
    sudo chmod 444 cert.pem
    ```

1. Enable Vault LDAP auth:

    ```sh
    vault auth enable ldap
    ```

1. Write the following LDAP auth config:

    ```sh
    vault write auth/ldap/config \
        url="ldap://ldap.ca-lab.private" \
        userattr="cn" \
        userdn="ou=Users,dc=ca-lab,dc=private" \
        groupdn="ou=Users,dc=ca-lab,dc=private" \
        groupfilter="(&(objectClass=groupOfNames)(member={{.UserDN}}))" \
        groupattr="cn"
    ```

1. Map the engineering Vault policy to the engineering LDAP group:

    ```sh
    vault write auth/ldap/groups/Engineering policies=Engineering
    ```

1. Login to Vault using LDAP with the following command, and enter _sheep_ as the password when prompted:

    ```sh
    vault login -method=ldap username='Jeremy Cook'
    ```


1. Confirm that you have the capabilities given in the engineering Vault policy:

    ```sh
    vault token capabilities secret/data/Engineering
    ```

## Cleaning Up

Delete the CloudFormation stack to remove all the resources used in the Lab.