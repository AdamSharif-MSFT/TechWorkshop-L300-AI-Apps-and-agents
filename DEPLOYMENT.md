# Azure Container Registry Deployment Setup

This document explains how to set up automated deployment to Azure Container Registry using GitHub Actions.

## Prerequisites

1. **Azure Container Registry**: Create an ACR instance in Azure
2. **Service Principal**: Create a service principal with push permissions to ACR
3. **GitHub Secrets**: Configure the required secrets in your GitHub repository

## Required GitHub Secrets

Configure these secrets in your GitHub repository settings (`Settings > Secrets and variables > Actions`):

### 1. ACR_USERNAME
- The username for your Azure Container Registry service principal
- Usually the `appId` of the service principal

### 2. ACR_PASSWORD
- The password for your Azure Container Registry service principal
- Usually the `password` of the service principal

### 3. ENV
- The complete contents of your .env file
- This should include all environment variables your application needs
- Example format:
  ```
  gpt_endpoint=https://your-endpoint.openai.azure.com/
  gpt_deployment=your-deployment-name
  gpt_api_key=your-api-key
  gpt_api_version=2024-08-01-preview
  ```

## Setup Steps

### 1. Create Azure Container Registry
```bash
# Create resource group (if needed)
az group create --name myResourceGroup --location eastus

# Create ACR
az acr create --resource-group myResourceGroup --name your-registry-name --sku Basic
```

### 2. Create Service Principal
```bash
# Get ACR resource ID
ACR_RESOURCE_ID=$(az acr show --name your-registry-name --query id --output tsv)

# Create service principal with AcrPush role
az ad sp create-for-rbac --name "GitHub-Actions-ACR" --role AcrPush --scopes $ACR_RESOURCE_ID
```

Save the output `appId` and `password` for the GitHub secrets.

### 3. Update Workflow Configuration

Edit `.github/workflows/deploy-to-acr.yml` and update:
```yaml
env:
  REGISTRY_NAME: your-actual-registry-name # Replace with your ACR name
```

### 4. Configure GitHub Secrets

In your GitHub repository:
1. Go to `Settings > Secrets and variables > Actions`
2. Add the three required secrets listed above

## Workflow Trigger

The deployment workflow triggers on:
- Push to `main` branch
- Manual trigger via GitHub Actions UI

## Security Notes

- ✅ .env files are already in .gitignore
- ✅ .env file is created during build and cleaned up after
- ✅ Secrets are handled securely through GitHub Secrets
- ✅ No sensitive information is committed to the repository

## Docker Image Details

- **Base Image**: python:3.12-slim
- **Port**: 8000 (FastAPI/uvicorn)
- **Context**: `./src` directory only
- **Multi-platform**: linux/amd64
- **Caching**: GitHub Actions cache for faster builds

## Troubleshooting

### Common Issues

1. **ACR Login Failed**
   - Verify ACR_USERNAME and ACR_PASSWORD secrets
   - Ensure service principal has AcrPush permissions

2. **Build Context Issues**
   - Ensure all required files are in the `src/` directory
   - Check Dockerfile paths are relative to `src/`

3. **Environment Variables Not Working**
   - Verify ENV secret contains complete .env file contents
   - Check environment variable names match your application code

### Logs and Debugging

- Check GitHub Actions logs in the `Actions` tab
- Use Azure CLI to inspect ACR: `az acr repository list --name your-registry-name`
- Test locally: `docker build -t test ./src`