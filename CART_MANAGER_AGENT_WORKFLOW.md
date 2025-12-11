# Cart Manager Agent GitHub Actions Setup

This document explains how to configure GitHub Actions to automatically run the Cart Manager Agent when related files change.

## Workflow Triggers

The workflow (`cart-manager-agent.yml`) automatically runs when changes are made to:
- `src/app/agents/cartManagerAgent_initializer.py`
- `src/prompts/CartManagerPrompt.txt`
- `src/app/agents/agent_processor.py` (affects cart manager function tools)

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

## Cart Manager Agent Overview

The Cart Manager Agent is designed to:
- Handle shopping cart operations and management
- Use conversation context for cart interactions
- Minimal tool dependencies (uses empty function set in agent_processor)
- Focus on cart state management and user experience

### Key Components

1. **Agent Initializer** (`cartManagerAgent_initializer.py`):
   - Sets up Azure AI Projects client
   - Loads cart manager prompt
   - Creates minimal toolset
   - Initializes agent with "cart_manager" environment variable

2. **Prompt File** (`CartManagerPrompt.txt`):
   - Contains instructions for cart management behavior
   - Defines how agent should handle cart operations
   - Specifies conversation patterns and responses

3. **Agent Processor Integration** (`agent_processor.py`):
   - Handles "cart_manager" agent type
   - Provides empty function set (cart manager uses conversation context)
   - Minimal tool dependencies by design

## Workflow Steps Overview

1. **Checkout & Setup**: Gets code and sets up Python 3.12
2. **Dependencies**: Installs requirements from `src/requirements.txt`
3. **Environment**: Creates .env file from GitHub secret
4. **Azure Auth**: Authenticates with Azure using service principal
5. **Validation**: Checks required environment variables and prompt file
6. **Agent Init**: Runs the cart manager agent initializer
7. **Function Tools Test**: Validates agent processor integration
8. **Agent Validation**: Confirms cart manager agent environment variable is set
9. **Cleanup**: Removes sensitive files

## Environment Variables

The workflow validates these required variables:
- `AZURE_AI_AGENT_ENDPOINT`: Azure AI Project endpoint
- `AZURE_AI_AGENT_MODEL_DEPLOYMENT_NAME`: Model deployment name

The initialized agent sets:
- `cart_manager`: Environment variable containing agent configuration

## Troubleshooting

### Common Issues

1. **Authentication Failed**
   - Verify AZURE_CREDENTIALS JSON format
   - Check service principal permissions
   - Ensure subscription and tenant IDs are correct

2. **Prompt File Not Found**
   - Verify `CartManagerPrompt.txt` exists in `src/prompts/`
   - Check file encoding (should be UTF-8)
   - Ensure file has proper read permissions

3. **Agent Processor Integration**
   - Verify `agent_processor.py` has cart_manager case
   - Check that empty function set is returned for cart_manager
   - Ensure FunctionTool can handle empty function sets

4. **Environment Variables Missing**
   - Check ENV secret contains all required variables
   - Verify variable names match code expectations
   - Test .env file format locally

### Debugging Steps

1. **Check Workflow Logs**: View detailed logs in GitHub Actions
2. **Validate Prompt**: Ensure CartManagerPrompt.txt is accessible and properly formatted
3. **Test Agent Processor**: Verify agent_processor.py cart_manager integration locally
4. **Azure Connection**: Test Azure AI Projects client connection

### Local Testing
```bash
# Set up local environment
cd src
pip install -r requirements.txt

# Copy your .env file (don't commit it!)
cp /path/to/your/.env .env

# Test the agent initializer
python -m app.agents.cartManagerAgent_initializer

# Test agent processor integration
python -c "from app.agents.agent_processor import create_function_tool_for_agent; print(create_function_tool_for_agent('cart_manager'))"

# Verify prompt file
python -c "
import os
with open('prompts/CartManagerPrompt.txt', 'r', encoding='utf-8') as f:
    print(f'Prompt loaded: {len(f.read())} characters')
"
```

## Agent Architecture Notes

The Cart Manager Agent follows a **conversation-centric** design:
- **No External Tools**: Uses empty function set, relies on conversation context
- **Minimal Dependencies**: Reduces complexity and potential failure points
- **Context-Aware**: Manages cart state through conversation memory
- **Focused Scope**: Specialized for cart management operations only

This design makes the Cart Manager Agent:
- ✅ Fast to initialize and execute
- ✅ Less prone to tool-related failures
- ✅ Easier to test and validate
- ✅ Focused on core cart management functionality

## Security Notes

- ✅ .env files are in .gitignore
- ✅ Secrets are cleaned up after workflow runs  
- ✅ Service principal uses minimal required permissions
- ✅ Artifacts are retained for only 7 days
- ✅ Sensitive data never appears in logs
- ✅ Empty function set reduces attack surface

## Monitoring

- Workflow logs show execution status and agent initialization
- Failed runs upload debug artifacts for troubleshooting
- Application Insights can track agent performance (if configured)
- Azure AI Project logs show agent interactions and conversations