# Azure Pipelines Cheat Sheet

## Introduction
Azure Pipelines is a service that helps you automate building, testing, and deploying code. It supports multiple languages and integrates with GitHub, Azure Repos, and other repositories.

This cheat sheet provides step-by-step guidance, starting from basics to advanced topics.

---

## Basic Structure of `azure-pipelines.yml`

```yaml
trigger:
  - main  # Branch to trigger the pipeline on

pool:
  vmImage: 'ubuntu-latest'  # Specify the agent

steps:
  - script: echo "Hello, World!"  # Run a script command
```

---

## Key Sections in Azure Pipelines

### 1. **Trigger**
Defines the branches or paths that start the pipeline.

```yaml
trigger:
  branches:
    include:
      - main
      - develop
  paths:
    include:
      - src/
```

### 2. **Pipeline Variables**
Define reusable variables.

```yaml
variables:
  buildConfiguration: Release

steps:
  - script: echo "$(buildConfiguration)"  # Access variable
```

### 3. **Pool**
Specifies the agent to run the pipeline.

```yaml
pool:
  vmImage: 'windows-latest'
```

### 4. **Steps**
Defines tasks or scripts to run in the pipeline.

```yaml
steps:
  - script: echo "Running a step"
```

### 5. **Jobs**
Group related steps into a single job.

```yaml
jobs:
- job: Build
  pool:
    vmImage: 'ubuntu-latest'
  steps:
    - script: echo "Building the app"
```

---

## Tasks

### 1. **Checkout Source Code**
Use `checkout` to fetch code from the repository.

```yaml
steps:
  - checkout: self  # Checkout the current repository
```

### 2. **Build with .NET**

```yaml
steps:
  - task: UseDotNet@2
    inputs:
      packageType: 'sdk'
      version: '6.x'

  - script: dotnet build MyApp.sln
```

### 3. **Publish Artifacts**
Store build outputs for later stages or downloading.

```yaml
steps:
  - publish: $(System.DefaultWorkingDirectory)/output
    artifact: drop
```

---

## Pipelines by Use Case

### 1. **CI Pipeline**
A simple Continuous Integration pipeline that builds and runs tests.

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

steps:
  - script: npm install
  - script: npm run build
  - script: npm test
```

### 2. **CD Pipeline**
Deploy to Azure App Service after building the code.

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

steps:
  - task: UseDotNet@2
    inputs:
      packageType: 'sdk'
      version: '6.x'

  - script: dotnet publish -o $(Build.ArtifactStagingDirectory)

  - task: PublishBuildArtifacts@1

  - task: AzureRmWebAppDeployment@4
    inputs:
      azureSubscription: 'your-service-connection'
      appType: 'webApp'
      appName: 'my-app-name'
      package: $(Build.ArtifactStagingDirectory)/*.zip
```

---

## Advanced Topics

### 1. **Stages**
Divide the pipeline into multiple stages.

```yaml
stages:
- stage: Build
  jobs:
    - job: BuildJob
      steps:
        - script: echo "Building..."

- stage: Deploy
  dependsOn: Build
  jobs:
    - job: DeployJob
      steps:
        - script: echo "Deploying..."
```

### 2. **Conditions**
Run steps or jobs only if certain conditions are met.

```yaml
steps:
  - script: echo "Run this step only on main"
    condition: eq(variables['Build.SourceBranch'], 'refs/heads/main')
```

### 3. **Templates**
Reuse pipeline code using templates.

**Template File:** `_steps-template.yml`
```yaml
steps:
  - script: echo "Hello from template!"
```

**Main Pipeline:**
```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

extends:
  template: _steps-template.yml
```

### 4. **Secrets**
Use Azure Key Vault to manage secrets.

```yaml
variables:
- group: MyVariableGroup

steps:
  - script: echo "Using secret: $(MySecret)"
```

---

## Best Practices

1. **Keep Pipelines Simple**: Avoid overcomplicating the pipeline file.
2. **Use Templates**: Reuse steps across multiple pipelines.
3. **Secure Secrets**: Always use Azure Key Vault or secure variable groups.
4. **Test Locally**: Validate pipeline tasks using tools like `tfx-cli`.
5. **Monitor Pipeline Runs**: Use Azure DevOps dashboards for insights.

---

## Debugging Pipelines

- Enable verbose logging:
  ```yaml
  variables:
    System.Debug: true
  ```

- Rerun failed jobs with debug logs from the Azure Pipelines UI.

---

This cheat sheet provides a comprehensive overview of Azure Pipelines for both beginners and advanced users. Let me know if you need further clarification or examples!
