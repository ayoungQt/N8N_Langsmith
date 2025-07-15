# N8N LangSmith Integration

This project demonstrates how to connect n8n workflows with LangSmith for LLM observability and tracing.

## Prerequisites

1. **N8N Installation**: Make sure you have n8n installed and running
2. **LangSmith Account**: Sign up at [LangSmith](https://smith.langchain.com/)
3. **API Keys**: Get your LangSmith API key from the dashboard

## Setup

### 1. Environment Variables

Create a `.env` file in your n8n installation directory:

```env
LANGSMITH_API_KEY=your_langsmith_api_key_here
LANGSMITH_ENDPOINT=https://api.smith.langchain.com
LANGSMITH_PROJECT=your_project_name
```

### 2. N8N Nodes Required

You'll need to install the following nodes:
- HTTP Request node (built-in)
- Set node (built-in)
- Code node (built-in)

## Flow Examples

### Basic LangSmith Trace Creation

This flow demonstrates how to:
1. Create a new trace in LangSmith
2. Add spans to track LLM operations
3. Update trace status

### LangSmith Metrics Collection

This flow shows how to:
1. Collect metrics from your LLM operations
2. Send them to LangSmith
3. Create dashboards and alerts

## Usage

1. Import the provided flow JSON files into n8n
2. Configure your LangSmith API credentials
3. Customize the flows according to your needs
4. Activate the workflows

## API Endpoints Used

- `POST /traces` - Create new traces
- `POST /spans` - Add spans to traces
- `PUT /traces/{trace_id}` - Update trace status
- `POST /metrics` - Send metrics data

## Troubleshooting

- Ensure your API key has the correct permissions
- Check that your LangSmith project exists
- Verify network connectivity to LangSmith endpoints
- Review n8n execution logs for detailed error messages 