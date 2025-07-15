# N8N LangSmith Setup Guide

This guide will walk you through setting up n8n to connect with LangSmith for LLM observability and tracing.

## Step 1: Prerequisites

### 1.1 Install N8N
If you haven't installed n8n yet, you can install it using npm:

```bash
npm install n8n -g
```

Or using Docker:

```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

### 1.2 Get LangSmith API Key
1. Go to [LangSmith](https://smith.langchain.com/)
2. Sign up or log in to your account
3. Navigate to your profile settings
4. Copy your API key

### 1.3 Create a LangSmith Project
1. In LangSmith dashboard, create a new project
2. Note down the project name (you'll need this for configuration)

## Step 2: Environment Configuration

### 2.1 Create Environment File
Create a `.env` file in your n8n installation directory:

```env
# LangSmith Configuration
LANGSMITH_API_KEY=your_langsmith_api_key_here
LANGSMITH_ENDPOINT=https://api.smith.langchain.com
LANGSMITH_PROJECT=your_project_name

# OpenAI Configuration (if using LLM tracking flow)
OPENAI_API_KEY=your_openai_api_key_here
```

### 2.2 Start N8N with Environment Variables
```bash
n8n start
```

Or with Docker:
```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  --env-file .env \
  n8nio/n8n
```

## Step 3: Import the Flows

### 3.1 Basic Trace Flow
1. Open n8n in your browser (usually `http://localhost:5678`)
2. Click "Import from file"
3. Select `flows/basic-langsmith-trace.json`
4. The flow will be imported with all nodes configured

### 3.2 LLM Tracking Flow
1. Import `flows/llm-tracking-flow.json`
2. This flow includes OpenAI integration for LLM operations

## Step 4: Configure Nodes

### 4.1 HTTP Request Node Configuration
Each HTTP Request node in the flows is configured to:
- Use Bearer token authentication
- Send JSON data
- Include proper headers

**Key Configuration Points:**
- **URL**: Uses environment variables for the LangSmith endpoint
- **Authentication**: Generic HTTP Header Auth with Bearer token
- **Headers**: Content-Type set to application/json
- **Body**: JSON parameters for trace/span creation

### 4.2 Set Node Configuration
Set nodes are used to:
- Store configuration values
- Extract data from previous nodes
- Prepare data for next nodes

## Step 5: Test the Flows

### 5.1 Test Basic Trace Flow
1. Open the "Basic LangSmith Trace" workflow
2. Click "Execute Workflow"
3. Check the execution logs
4. Verify in LangSmith dashboard that a new trace was created

### 5.2 Test LLM Tracking Flow
1. Ensure you have OpenAI API key configured
2. Execute the "LLM Tracking with LangSmith" workflow
3. Check both n8n execution logs and LangSmith dashboard

## Step 6: Customize for Your Use Case

### 6.1 Modify Input Data
In the "Set Input Data" node, change the `userInput` value to test different prompts.

### 6.2 Add More Spans
You can add additional HTTP Request nodes to create more spans for different operations:

```json
{
  "name": "custom-operation",
  "trace_id": "={{ $('Set Input Data').first().json.traceId }}",
  "start_time": "={{ new Date().toISOString() }}",
  "inputs": "{\"custom_data\": \"your_data_here\"}",
  "metadata": "{\"operation\": \"custom_operation\"}"
}
```

### 6.3 Error Handling
Add error handling by creating conditional flows:

1. Add an "IF" node after LLM calls
2. Check for error responses
3. Create error spans in LangSmith
4. Update trace status to "error"

## Step 7: Monitor and Debug

### 7.1 LangSmith Dashboard
- View traces in real-time
- Analyze performance metrics
- Debug failed operations
- Create dashboards and alerts

### 7.2 N8N Execution Logs
- Check execution history
- View node outputs
- Debug connection issues
- Monitor performance

## Troubleshooting

### Common Issues

1. **Authentication Errors**
   - Verify your LangSmith API key is correct
   - Check that the key has proper permissions
   - Ensure the key is properly set in environment variables

2. **Connection Issues**
   - Verify network connectivity to LangSmith
   - Check firewall settings
   - Ensure the endpoint URL is correct

3. **JSON Parsing Errors**
   - Validate JSON structure in node configurations
   - Check for special characters in data
   - Ensure proper escaping of quotes

4. **Missing Environment Variables**
   - Verify all required environment variables are set
   - Restart n8n after changing environment variables
   - Check variable names match exactly

### Debug Steps

1. **Test API Connection**
   ```bash
   curl -H "Authorization: Bearer YOUR_API_KEY" \
        -H "Content-Type: application/json" \
        https://api.smith.langchain.com/traces
   ```

2. **Check N8N Logs**
   - Look for detailed error messages
   - Verify node execution order
   - Check data flow between nodes

3. **Validate LangSmith Response**
   - Check response status codes
   - Verify trace/span creation in dashboard
   - Look for error messages in response body

## Next Steps

Once you have the basic flows working:

1. **Integrate with Your Applications**: Connect these flows to your existing applications
2. **Add More Metrics**: Track additional performance metrics
3. **Create Dashboards**: Build custom dashboards in LangSmith
4. **Set Up Alerts**: Configure alerts for failed operations
5. **Scale Up**: Add more complex tracing for multi-step workflows

## Support

- [LangSmith Documentation](https://docs.smith.langchain.com/)
- [N8N Documentation](https://docs.n8n.io/)
- [LangSmith API Reference](https://api.smith.langchain.com/docs) 