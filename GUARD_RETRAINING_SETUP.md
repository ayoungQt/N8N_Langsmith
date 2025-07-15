# Guard Prompt Retraining Workflow Setup Guide

This guide will walk you through setting up the automated daily guard prompt retraining workflow that connects n8n with LangSmith and Google services.

## Prerequisites

### 1. Required Accounts & API Keys

1. **LangSmith Account**
   - Sign up at [LangSmith](https://smith.langchain.com/)
   - Get your API key from profile settings
   - Create a project for your guard prompts

2. **Google Workspace Account**
   - Access to Google Drive and Google Sheets
   - Enterprise OAuth2 credentials for Gemini 2.5
   - No Google Cloud project required

3. **N8N Installation**
   - Ensure n8n is installed and running with access to:
   - HTTP Request nodes
   - Google Drive nodes
   - Google Sheets nodes
   - Google Gemini nodes
   - Gmail nodes

### 2. N8N Installation
Ensure n8n is installed and running with access to:
- HTTP Request nodes
- Google Drive nodes
- Google Sheets nodes
- Google Gemini nodes
- Gmail nodes

## Step 1: Environment Configuration

### 1.1 Update Environment Variables
Add these to your `.env` file:

```env
# LangSmith Configuration
LANGSMITH_API_KEY=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
LANGSMITH_ENDPOINT=https://api.smith.langchain.com
LANGSMITH_PROJECT=Prod_CSS_Service_AI
LANGSMITH_SESSION=6d53c216-306e-4bd7-9c2d-e80ba5d66ce9

# Google Gemini Configuration
# Uses enterprise OAuth2 - no additional env vars needed

# Email Configuration (for error notifications)
# Note: Using Gmail node with OAuth2 authentication
# No additional environment variables needed for Gmail
```

### 1.2 Google Drive Setup
1. Create a folder in Google Drive for guard prompts
2. Upload your guard prompt file (e.g., `unusual_prompt_agentic.txt`)
3. Note the folder ID from the URL

### 1.3 Google Sheets Setup
1. Create a new Google Sheet for logging
2. Create a sheet named "Guard_Retraining_Log"
3. Add these columns:
   - Date of Run
   - Status
   - Guard Name Updated
   - Result
   - Link to Google Doc
4. Note the spreadsheet ID from the URL

## Step 2: Configure N8N Credentials

### 2.1 LangSmith Credential
1. Go to n8n Credentials
2. Create new credential: "Generic API"
3. Name: "LangSmith API"
4. HTTP Header Auth:
   - Name: `x-api-key`
   - Value: `{{ $env.LANGSMITH_API_KEY }}`

### 2.2 Google Services Credentials
1. **Google Drive**: Configure OAuth2 credentials
2. **Google Sheets**: Configure OAuth2 credentials  
3. **Basic LLM Chain**: Configure with your Gemini enterprise OAuth2 credentials
4. **Gmail**: Configure OAuth2 credentials (same as other Google services)

### 2.3 Gmail Credential
1. Configure Gmail OAuth2 credential for error notifications
2. This will use the same Google OAuth2 setup as other Google services
3. The Gmail node will automatically use the authenticated user's email as the sender

## Step 3: Import and Configure the Workflow

### 3.1 Import the Workflow
1. Open n8n
2. Click "Import from file"
3. Select `flows/guard-prompt-retraining-v1.95.3.json`

**Note**: If the "Set Date Range & Config" node appears empty after import, manually add the fields using the configuration below.

### 3.2 Update Configuration Node
The workflow is pre-configured with your specific values, but verify the "Set Date Range & Config" node contains:

```json
{
  "min_created_at": "={{ $now.minus({ days: 7 }).startOf('day').toUTC().toISO() }}",
  "max_created_at": "={{ $now.minus({ days: 7 }).endOf('day').toUTC().toISO() }}",
  "guard_name": "unusual_prompt_agentic",
  "prompt_file_name": "unusual_prompt_agentic.txt",
  "google_drive_folder_id": "1jplKoFeo3xLShJFO4_MpPUVHfDr-cOwx",
  "google_sheets_spreadsheet_id": "1maCntXucpkj8MPV_w8Hi0NGvNka7r7aOvFTYilaqtis",
  "google_sheets_sheet_name": "Guard_Retraining_Log"
}
```

**If the node is empty, manually add each field with the values above.**

### 3.2.1 Manual Field Configuration (if needed)
If the "Set Date Range & Config" node appears empty after import:

1. **Click "Add Field"** in the "Fields to Set" section
2. **Add each field as a String type** with these exact names and values:

| Field Name | Value |
|------------|-------|
| `min_created_at` | `={{ $now.minus({ days: 7 }).startOf('day').toUTC().toISO() }}` |
| `max_created_at` | `={{ $now.minus({ days: 7 }).endOf('day').toUTC().toISO() }}` |
| `guard_name` | `unusual_prompt_agentic` |
| `prompt_file_name` | `unusual_prompt_agentic.txt` |
| `google_drive_folder_id` | `1jplKoFeo3xLShJFO4_MpPUVHfDr-cOwx` |
| `google_sheets_spreadsheet_id` | `1maCntXucpkj8MPV_w8Hi0NGvNka7r7aOvFTYilaqtis` |
| `google_sheets_sheet_name` | `Guard_Retraining_Log` |

### 3.3 Update Basic LLM Chain Configuration
In both Basic LLM Chain nodes, verify:
- The prompt is properly configured
- Connect to your Gemini enterprise OAuth2 credentials
- Temperature and max tokens are set appropriately

### 3.4 Update Gmail Configuration
In the "Send Error Email" node:
- `toEmail`: ayoung@questrade.com (as specified)
- The Gmail node will automatically use the authenticated user's email as the sender

## Step 4: Test the Workflow

### 4.1 Manual Test
1. Disable the schedule trigger temporarily
2. Add a manual trigger for testing
3. Execute the workflow manually
4. Check each step's output

### 4.2 Verify LangSmith Connection
1. Check that feedback is retrieved correctly
2. Verify run details are fetched
3. Confirm API responses are as expected

### 4.3 Test Google Services
1. Verify prompt file is downloaded from Drive
2. Check that Google Doc is created
3. Confirm logging to Google Sheets works

## Step 5: Schedule Configuration

### 5.1 Cron Expression
The workflow uses: `0 5 * * *`
- Runs daily at 5:00 AM UTC
- Adjust for your timezone (EST = UTC-5, so 5 AM UTC = 12 AM EST)

### 5.2 Timezone Considerations
- n8n runs in UTC by default
- For EST (UTC-5), use: `0 5 * * *`
- For other timezones, adjust accordingly

## Step 6: Error Handling Setup

### 6.1 Retry Configuration
All API calls are configured with:
- Retry on error: Enabled
- Max tries: 3
- Backoff strategy: Exponential

### 6.2 Error Notification
The workflow includes:
- Error workflow configuration
- Email notification to ayoung@questrade.com
- Detailed error information in email

## Step 7: Monitoring and Maintenance

### 7.1 Logging
The workflow logs to Google Sheets with:
- Date of execution
- Success/failure status
- Guard name being processed
- Link to generated Google Doc

### 7.2 Monitoring Points
1. **Daily execution**: Check n8n execution history
2. **LangSmith feedback**: Monitor for new false positives
3. **Google Doc creation**: Verify documents are being created
4. **Error notifications**: Monitor email for failures

## Troubleshooting

### Common Issues

1. **LangSmith API Errors**
   ```
   Error: 401 Unauthorized
   Solution: Verify API key is correct and has proper permissions
   ```

2. **Google Drive Access Issues**
   ```
   Error: File not found
   Solution: Check file ID and ensure OAuth2 credentials are valid
   ```

3. **Basic LLM Chain Errors**
   ```
   Error: Authentication failed
   Solution: Verify enterprise OAuth2 credentials are properly configured for Gemini
   ```

4. **Date Range Issues**
   ```
   Error: No feedback found
   Solution: Check date calculations and LangSmith data availability
   ```

### Debug Steps

1. **Test LangSmith API Directly**
   ```bash
   curl -H "x-api-key: YOUR_API_KEY" \
        "https://api.smith.langchain.com/feedback?key=False%20Positive&limit=1"
   ```

2. **Verify Google Credentials**
   - Test Google Drive access manually
   - Check OAuth2 token validity
   - Verify enterprise OAuth2 permissions for Gemini

3. **Check Date Calculations**
   - Verify the date range logic
   - Ensure timezone handling is correct
   - Test with known data periods

### Performance Optimization

1. **Batch Processing**
   - Current limit: 100 feedback items
   - Adjust based on your data volume
   - Consider pagination for large datasets

2. **LLM Optimization**
   - Temperature: 0.2-0.3 for consistent results
   - Max tokens: 2048-4096 based on prompt complexity
   - Consider model selection based on cost/performance

## Customization Options

### 1. Multiple Guards
To process multiple guards:
1. Create separate workflows for each guard
2. Use different schedule times to avoid conflicts
3. Maintain separate Google Drive folders and sheets

### 2. Enhanced Logging
Add additional logging:
1. Token usage tracking
2. Processing time metrics
3. Success rate calculations

### 3. Advanced Error Handling
Implement:
1. Slack notifications
2. Retry with exponential backoff
3. Circuit breaker pattern

## Security Considerations

1. **API Key Management**
   - Store keys in environment variables
   - Rotate keys regularly
   - Use least privilege access

2. **Data Privacy**
   - Ensure LangSmith data handling complies with policies
   - Review Google Drive sharing settings
   - Implement data retention policies

3. **Access Control**
   - Limit n8n access to authorized users
   - Monitor execution logs
   - Implement audit trails

## Support and Maintenance

### Regular Tasks
1. **Weekly**: Review execution logs and error rates
2. **Monthly**: Update guard prompts based on generated improvements
3. **Quarterly**: Review and optimize workflow performance

### Monitoring Alerts
Set up alerts for:
- Workflow failures
- High error rates
- Missing executions
- Performance degradation

## Next Steps

Once the workflow is running successfully:

1. **Scale Up**: Add more guards to the automation
2. **Optimize**: Fine-tune LLM prompts and parameters
3. **Enhance**: Add more sophisticated analysis and reporting
4. **Integrate**: Connect with other systems and workflows

For additional support:
- [LangSmith Documentation](https://docs.smith.langchain.com/)
- [N8N Documentation](https://docs.n8n.io/)
- [Google Cloud Documentation](https://cloud.google.com/docs) 