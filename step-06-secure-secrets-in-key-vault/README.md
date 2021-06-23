# 05 - Setup GitHub Actions

__This guide is part of the [migrate Java EE app to Azure training](../README.md)__

How to store secrets in Azure Key Vault and access those secrets from your application. 

## Overview of Key Vault References

Azure Key Vault provides a centralized secret store with role-based access controls and audit history. Using App Service's **Key Vault References** feature, you can securely store your credentials, API keys, and other secrets in Key Vault and easily access them from your application code on App Service.

[Key Vault References](https://docs.microsoft.com/azure/app-service/app-service-key-vault-references) is a feature of Azure App Service and Functions that allows your application code to retrieve secrets from Azure Key Vault without any code changes. The feature works by creating a system-assigned identity to authenticate to Key Vault, retrieve the secrets, and expose them as environment variables to your application on App Service. This means you can leverage all of Key Vault's secret management features without rewriting your application.

## Create a Managed Identity

A managed identity acts as a user in your Active Directory for automation purposes. It is inherently tied to your web app and will be deleted if the web app is deleted. For this scenario, the identity will be used to retrieve the secrets from Key Vault when the app starts. Run the following command to create a manged identity.

```bash
az webapp identity assign --name <your-jboss-webapp-name> --resource-group <rg-of-the-webapp>
```

The command wil output information about the identity. Copy the `principalId` for the next section.

## Create a Key Vault

Next, provision an Azure Key Vault, provide access to the identity you just created, and set the database username, password, and URL in Key Vault.

1. Create the Key Vault using the Azure CLI.

    ```bash
    az keyvault create --name ${WEBAPP}-key-vault              \
                        --resource-group ${RESOURCE_GROUP}     \
                        --location ${REGION}                   \
                        --enabled-for-deployment true          \
                        --enabled-for-disk-encryption true     \
                        --enabled-for-template-deployment true \
                        --sku standard
    ```

1. Now grant the managed identity `get` and `list` access to the Key Vault.

    ```bash
    az keyvault set-policy --name ${WEBAPP}-key-vault     \
                            --secret-permission get list  \
                            --object-id <the principal ID from earlier>
    ```

1. Lastly, add the Postgres username, password, and URL to the Key Vault. If you followed sections 2 and 3, you should still have the secrets saved as environment variables on your machine.

    ```bash
    az keyvault secret set --name POSTGRES_SERVER_ADMIN_FULL_NAME \
                        --value $POSTGRES_SERVER_ADMIN_FULL_NAME  \
                        --vault-name jboss-app-key-vault

    az keyvault secret set --name POSTGRES_SERVER_ADMIN_PASSWORD \
                        --value $POSTGRES_SERVER_ADMIN_PASSWORD  \
                        --vault-name jboss-app-key-vault

    az keyvault secret set --name POSTGRES_CONNECTION_URL \
                        --value $POSTGRES_CONNECTION_URL  \
                        --vault-name java-app-key-vault
    ```

## Set Key Vault References on App Service

The secrets will be exposed as environment variables to the JBoss application, so the final step is to get the URI's of the Key Vault secrets and set them as app settings on the Web App (with the appropriate syntax).

1. Get the URI’s of your three secrets. Run the commands below and copy the `id` value in the console output.

    ```bash
    az keyvault secret show --vault-name ${WEBAPP}-key-vault --name POSTGRES-URL
    az keyvault secret show --vault-name ${WEBAPP}-key-vault --name POSTGRES-USERNAME
    az keyvault secret show --vault-name ${WEBAPP}-key-vault --name POSTGRES-PASSWORD
    ```

2. Now create the app settings with the Key Vault references. For each setting, replace “YOUR_SECRET_URI” with the corresponding id’s from the previous step.

    ```bash
    az webapp config appsettings set -n ${WEBAPP} -g ${RESOURCE_GROUP} --settings \
        SPRING_DATASOURCE_URL=@Microsoft.KeyVault(SecretUri=YOUR_SECRET_URI) \
        SPRING_DATASOURCE_USERNAME=@Microsoft.KeyVault(SecretUri=YOUR_SECRET_URI)\
        SPRING_DATASOURCE_PASSWORD=@Microsoft.KeyVault(SecretUri=YOUR_SECRET_URI)
    ```

The `@Microsoft.KeyVault(SecretUri=<SecretURI>)` used above is the syntax for a Key Vault reference. When the web app starts, App Service will recognize this format and use the managed identity to retrieve the secret from Key Vault. "\<SecretURI>" is the data-plane URI of a secret in Key Vault, with an optional version. There is an alternate syntax documented here. An alternative syntax is [documented here](https://docs.microsoft.com/azure/app-service/app-service-key-vault-references#reference-syntax).

---

⬅️ Previous guide: [Step 05 - Setup GitHub Actions](../step-05-setup-github-actions/README.md)

➡️ Next guide: [Conclusion](../step-99-conclusion/README.md)
