# Nested ARM template with Azure Key Vault using a dynamic id

When doing deployments to your Azure environment using an ARM template, it's quite common to use this template for adding secrets, like connection strings, to your resources also.

In order to add secrets to your resources it's advised to use a service like Azure Key Vault.
Azure Key Vault enables you to add secrets to your resources, without writing them in your templates, which defeats the purpose of them being secrets.

It's rather easy to enable Azure Key Vault in your template, take a look at the following sample:
```
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "adminLogin": {
            "value": "exampleadmin"
        },
        "adminPassword": {
            "reference": {
              "keyVault": {
                "id": "/subscriptions/<subscription-id>/resourceGroups/examplegroup/providers/Microsoft.KeyVault/vaults/<vault-name>"
              },
              "secretName": "examplesecret"
            }
        },
        "sqlServerName": {
            "value": "<your-server-name>"
        }
    }
}
```

Source: https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-keyvault-parameter#reference-a-secret-with-static-id

However, in this sample you need to specify a subscription id. While this isn't wrong, it's not something I like very much when doing OSS development. 
In a closed development environment (for example a customer), you can get away with this as you will probably deploy everything to the same subscription(s).

When doing OSS development there's no way to tell which Azure subscription people are using. Therefore you need to use a dynamic id.


I was struggling with this for quite a bit. It's explained quite well in the documentation (https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-keyvault-parameter#reference-a-secret-with-dynamic-id),
but it wasn't working for me.
That is, until I discovered the `templateLink` is only available when actually specifying an URI to a remote location. Local files aren't supported.

The contents of this repository contain a sample which is works properly. You have to specify the `templateLink` base URI, along with some other parameters you want to use.
If everything is filled out correctly, it'll create an Azure Key Vault and an App Service which retrieves a secret value from the Key Vault.

## Add support for local filesystem in the `templateLink`.

There's a feedback suggestion where you can vote if local files should also be supported for the use of `templateLink`

Suggestion: https://feedback.azure.com/forums/281804-azure-resource-manager/suggestions/17275646-enable-using-local-filesystem-for-linked-templates

I'd like this to be implemented of course, so I can use the templates from the artifacts VSTS is producing for me, without having to copy the templates (per build) to some kind of (public) storage container.