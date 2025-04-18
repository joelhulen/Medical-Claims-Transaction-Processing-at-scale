# Deployment - Standard

## Prerequisites

- Azure Subscription
- Subscription access to Azure OpenAI service. Start here to [Request Access to Azure OpenAI Service](https://aka.ms/oaiapply)

- Backend (Web API, Worker Service, Console Apps, etc.)
  - Visual Studio 2022 17.6 or later (required for passthrough Visual Studio authentication for the Docker container)
  - .NET 7 SDK
  - Docker Desktop (with WSL for Windows machines)
  - Azure CLI ([v2.51.0 or greater](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli))
  - [Helm 3.11.1 or greater](https://helm.sh/docs/intro/install/)
- Frontend (React web app)
  - Visual Studio Code
  - Ensure you have the latest version of NPM and node.js:
    - Install NVM from https://github.com/coreybutler/nvm-windows
    - Run nvm install latest
    - Run nvm list (to see the versions of NPM/node.js available)
    - Run nvm use latest (to use the latest available version)

To start the React web app:

1. Navigate to the `ui/medical-claims-ui` folder
2. Run npm install to restore the packages
3. Run npm run dev
4. Open localhost:3000 in a web browser

## Deployment steps

Follow the steps below to deploy the solution to your Azure subscription.

1. Ensure all the prerequisites are installed.  

1. Clone the repository:

    > **Important:** Do not forget the `--recurse-submodules` parameter. This loads the `AKS-Construction` submodule that contains AKS-specific Bicep templates.

    ```bash
    git clone --recurse-submodules https://github.com/Azure/Medical-Claims-Transaction-Processing-at-scale.git
    ```

1. Run the following script to provision the infrastructure and deploy the API and frontend. This will provision all of the required infrastructure, deploy the API and web app services into AKS, and provision and load artifacts into a Synapse Analytics workspace.

    ```pwsh
    cd .\Medical-Claims-Transaction-Processing-at-scale
    ./deploy/powershell/Unified-Deploy.ps1 -resourceGroup <rg_name> -location <location> -subscription <target_subscription_id>
    ```

>**NOTE**: Make sure to set the `<location>` value to a region that supports Azure OpenAI services.  See [Azure OpenAI service regions](https://azure.microsoft.com/en-us/explore/global-infrastructure/products-by-region/?products=cognitive-services&regions=all) for more information.

### Deployment samples

1. Default deployment using Azure Container Apps. 
    ```pwsh
    cd .\Medical-Claims-Transaction-Processing-at-scale
    ./deploy/powershell/Unified-Deploy.ps1 -resourceGroup <rg_name> -location <location> -subscription <target_subscription_id>
    ```
1. Deployment using Azure Kubernetes Service. 
    ```pwsh
    cd .\Medical-Claims-Transaction-Processing-at-scale
    ./deploy/powershell/Unified-Deploy.ps1 -resourceGroup <rg_name> -location <location> -subscription <target_subscription_id> -deployAks $true
    ```
1. Deployment using an existing Azure Open AI Service. 
    ```pwsh
    cd .\Medical-Claims-Transaction-Processing-at-scale
    ./deploy/powershell/Unified-Deploy.ps1 -resourceGroup <rg_name> -location <location> -subscription <target_subscription_id> -openAiRg <openai_rg_name> -openAiName <openai_service_name> -openAiCompletionsDeployment <openai_deployment>
    ```

### Enabling/Disabling Deployment Steps

The following flags can be used to enable/disable specific deployment steps in the `Unified-Deploy.ps1` script.

| Parameter Name | Description |
|----------------|-------------|
| stepDeployOpenAi | Enables or disables the deployment of an OpenAi instance in Azure prior to the Bicep template deployment. Valid values are 0 (Disabled) and 1 (Enabled). See the `deploy/powershell/Deploy-OpenAi.ps1` script.
| openAiName | Name of a pre-existing OpenAI instance to use instead of creating a new one. Default value is `$null`. See the `deploy/powershell/Unified-Deploy.ps1` script. 
| openAiRg | Name of the resource group a pre-existing OpenAI instance to use resides in. Default value is `$null`. See the `deploy/powershell/Unified-Deploy.ps1` script.
| deployAks | Enables 
| openAiCompletionsDeployment | Name of the completions deployment to use in a pre-existing OpenAI instance. Default value is `$null`. See the `deploy/powershell/Unified-Deploy.ps1` script.
| stepDeployBicep | Enables or disables the provisioning of resources in Azure via Bicep templates (located in `./infrastructure`). Valid values are 0 (Disabled) and 1 (Enabled). See the `deploy/powershell/Deploy-Bicep.ps1` script.
| stepBuildPush | Enables or disables the build and push of Docker images into the Azure Container Registry (ACR). Valid values are 0 (Disabled) and 1 (Enabled). See the `deploy/infrastructure/BuildPush.ps1` script.
| stepDeployCertManager | Enables or disables adding the official cert-manager repository to your local and updates the repo cache. Valid values are 0 (Disabled) and 1 (Enabled). See the `deploy/infrastructure/DeployCertManager.ps1` script.
| stepDeployTls | Enables or disables SSL/TLS support on the AKS cluster in the resource group. Valid values are 0 (Disabled) and 1 (Enabled). See the `deploy/infrastructure/DeployTlsSupport.ps1` script.
| stepDeployImages | Enables or disables deploying the Docker images from the `CoreClaims.WebAPI` and `CoreClaims.WorkerService` projects to AKS. Valid values are 0 (Disabled) and 1 (Enabled). See the `deploy/infrastructure/Deploy-Images-Aks.ps1` script.
| stepPublishSite | Enables or disables the build and deployment of the static HTML site to the hosting storage account in the target resource group. Valid values are 0 (Disabled) and 1 (Enabled). See the `deploy/infrastructure/Publish-Site.ps1` script.
| stepSetupSynapse | Enables or disables the deployment of a Synapse artifacts to the target synapse workspace. Valid values are 0 (Disabled) and 1 (Enabled). See the `deploy/infrastructure/Setup-Synapse.ps1` script.
| stepLoginAzure | Enables or disables interactive Azure login. If disabled, the deployment assumes that the current Azure CLI session is valid. Valid values are 0 (Disabled).

Example command:

```pwsh
cd deploy/powershell
./Unified-Deploy.ps1 -resourceGroup myRg `
                     -subscription 0000... `
                     -stepLoginAzure 0 `
                     -stepOpenAi 0 `
                     -stepDeployBicep 0 `
                     -stepDeployCertManager 0 `
                     -stepDeployTls 0 `
                     -stepBuildPush 1 `
                     -stepDeployImages 1 `
                     -stepSetupSynapse 0 `
                     -stepPublishSite 1
```
