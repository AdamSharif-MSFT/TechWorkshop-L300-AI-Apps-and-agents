# Customer Loyalty Agent GitHub Actions Setup

This document explains how to configure GitHub Actions to automatically run the Customer Loyalty Agent when related files change.

## Workflow Triggers

The workflow (`customer-loyalty-agent.yml`) automatically runs when changes are made to:
- `src/app/agents/customerLoyaltyAgent_initializer.py`
- `src/prompts/CustomerLoyaltyAgentPrompt.txt` 
- `src/app/tools/discountLogic.py`

It triggers on:
- Push to `main` or `develop` branches
- Pull requests to `main` branch
- Manual workflow dispatch

## Required GitHub Secrets

Configure these secrets in your GitHub repository (`Settings > Secrets and variables > Actions`):

### 1. ENV
Complete .env file contents including:
```
AZURE_AI_AGENT_ENDPOINT=https://your-project.cognitiveservices.azure.com/
AZURE_AI_AGENT_MODEL_DEPLOYMENT_NAME=your-model-deployment
AZURE_OPENAI_ENDPOINT=https://your-openai.openai.azure.com/
AZURE_OPENAI_KEY=your-openai-key
AZURE_OPENAI_API_VERSION=2024-08-01-preview
gpt_deployment=your-gpt-deployment
APPLICATIONINSIGHTS_CONNECTION_STRING=your-app-insights-connection
```

### 2. AZURE_CREDENTIALS
Service principal credentials for Azure CLI login:
```json
{
  "clientId": "your-client-id",
  "clientSecret": "your-client-secret", 
  "subscriptionId": "your-subscription-id",
  "tenantId": "your-tenant-id"
}
```

### 3. Individual Azure Secrets (Alternative to AZURE_CREDENTIALS)
If not using AZURE_CREDENTIALS, configure these individually:
- `AZURE_CLIENT_ID`: Service principal client ID
- `AZURE_CLIENT_SECRET`: Service principal client secret  
- `AZURE_TENANT_ID`: Azure tenant ID

## Azure Setup Requirements

### 1. Create Service Principal
```bash
# Create service principal with Cognitive Services User role
az ad sp create-for-rbac --name "GitHub-Actions-CustomerLoyalty" \
  --role "Cognitive Services User" \
  --scopes /subscriptions/{subscription-id}/resourceGroups/{resource-group}
```

### 2. Grant Additional Permissions
The service principal needs:
- **Cognitive Services User**: For Azure AI Projects access
- **Storage Blob Data Contributor**: If using blob storage
- **Application Insights Component Contributor**: For telemetry (optional)

### 3. Azure AI Project Setup
Ensure your Azure AI Project has:
- Deployed AI model (GPT-4, etc.)
- Proper endpoint configuration
- Network access from GitHub Actions runners

## Workflow Steps Overview

1. **Checkout & Setup**: Gets code and sets up Python 3.12
2. **Dependencies**: Installs requirements from `src/requirements.txt`
3. **Environment**: Creates .env file from GitHub secret
4. **Azure Auth**: Authenticates with Azure using service principal
5. **Validation**: Checks required environment variables
6. **Agent Init**: Runs the customer loyalty agent initializer
7. **Validation**: Confirms agent and discount logic work correctly
8. **Cleanup**: Removes sensitive files

## Environment Variables

The workflow validates these required variables:
- `AZURE_AI_AGENT_ENDPOINT`: Azure AI Project endpoint
- `AZURE_AI_AGENT_MODEL_DEPLOYMENT_NAME`: Model deployment name

Additional variables used by the Python code:
- `AZURE_OPENAI_*`: OpenAI service configuration
- `APPLICATIONINSIGHTS_CONNECTION_STRING`: Telemetry (optional)

## Troubleshooting

### Common Issues

1. **Authentication Failed**
   - Verify AZURE_CREDENTIALS JSON format
   - Check service principal permissions
   - Ensure subscription and tenant IDs are correct

2. **Module Import Errors**
   - Verify all dependencies in requirements.txt
   - Check PYTHONPATH is set correctly in workflow
   - Ensure relative imports work from src/ directory

3. **Azure AI Project Connection**
   - Verify endpoint URL format
   - Check model deployment name
   - Ensure service principal has Cognitive Services access

4. **Environment Variables Missing**
   - Check ENV secret contains all required variables
   - Verify variable names match code expectations
   - Test .env file format locally

### Debugging Steps

1. **Check Workflow Logs**: View detailed logs in GitHub Actions
2. **Test Locally**: Run the script locally with same environment
3. **Validate Secrets**: Ensure all required secrets are configured
4. **Azure CLI Test**: Test service principal authentication manually

### Local Testing
```bash
# Set up local environment
cd src
pip install -r requirements.txt

# Copy your .env file (don't commit it!)
cp /path/to/your/.env .env

# Test the agent initializer
python -m app.agents.customerLoyaltyAgent_initializer

# Test discount logic
python -c "from app.tools.discountLogic import *; print('Success')"
```

## Security Notes

- ✅ .env files are in .gitignore
- ✅ Secrets are cleaned up after workflow runs  
- ✅ Service principal uses minimal required permissions
- ✅ Artifacts are retained for only 7 days
- ✅ Sensitive data never appears in logs

## Monitoring

- Workflow logs show execution status
- Failed runs upload debug artifacts
- Application Insights can track agent performance (if configured)
- Azure AI Project logs show agent interactions