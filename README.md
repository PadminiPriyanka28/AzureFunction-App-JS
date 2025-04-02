# AzureFunction-App-JS
# Azure DevOps Pipeline for Function App Deployment

This repository contains an Azure DevOps pipeline configuration to package and deploy an Azure Function App using CI/CD.

## Prerequisites
Before running this pipeline, ensure you have:
- An Azure DevOps organization and project set up
- An active Azure subscription
- A Function App created in Azure
- Service connection to Azure configured in Azure DevOps

## Pipeline Overview
This pipeline consists of the following stages:

### 1. **Build Stage**
- Packages the function app into a `.zip` file
- Publishes the package as an artifact named `drop`

### 2. **Deploy Stage**
- Downloads the published artifact
- Deploys the `.zip` package to the Azure Function App

## YAML Pipeline Configuration
```yaml
trigger:
  - main

pool:
  name: AgentPoolName

stages:
  - stage: Build
    jobs:
      - job: BuildFunctionApp
        steps:
          - task: CmdLine@2
            displayName: 'Package the function app'
            inputs:
              script: |
                powershell Compress-Archive -Path * -DestinationPath functionapp.zip

          - task: PublishBuildArtifacts@1
            displayName: 'Publish Artifact'
            inputs:
              pathToPublish: 'functionapp.zip'
              artifactName: 'drop'

  - stage: Deploy
    dependsOn: Build
    jobs:
      - job: DeployFunctionApp
        steps:
          - task: DownloadBuildArtifacts@0
            displayName: 'Download Artifact'
            inputs:
              artifactName: 'drop'
              downloadPath: '$(Pipeline.Workspace)' #work directory path

          - script: ls $(Pipeline.Workspace)/drop
            displayName: 'Verify Downloaded Artifacts'

          - task: AzureFunctionApp@1
            displayName: 'Deploy to Azure Function App'
            inputs:
              azureSubscription: 'YourAzureServiceConnection'
              appType: 'functionApp'
              appName: 'YourFunctionAppName'
              package: '$(Pipeline.Workspace)/drop/functionapp.zip'
```

## Explanation of Pipeline Steps
### **Build Stage**
1. **Package the Function App**
   - Uses `Compress-Archive` in PowerShell to zip all project files into `functionapp.zip`
2. **Publish Artifact**
   - Saves `functionapp.zip` as a build artifact under the name `drop`

### **Deploy Stage**
1. **Download Artifact**
   - Retrieves the `drop` artifact and places it in `$(Pipeline.Workspace)`
2. **Verify Download**
   - Lists the contents of `drop` for debugging
3. **Deploy to Azure**
   - Uses the `AzureFunctionApp` task to deploy the zip package

## Troubleshooting
- **Missing `functionapp.zip` in Deployment:**
  - Ensure `PublishBuildArtifacts` is running successfully in the Build stage
  - Check if `DownloadBuildArtifacts` is correctly configured
  
- **Function App Not Found:**
  - Verify the `appName` in the deployment task matches the Azure Function App name
  - Ensure your Azure service connection has permissions to deploy

## Conclusion
This pipeline automates the packaging and deployment of an Azure Function App using Azure DevOps. Modify the pipeline as needed to fit your project structure and requirements.

For more information, refer to the [Azure DevOps Documentation](https://learn.microsoft.com/en-us/azure/devops/pipelines/).

