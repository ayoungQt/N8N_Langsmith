# Guard Prompt Retraining Workflow - Quick Reference

## Workflow Overview
**Name**: Automated Daily Guard Prompt Retraining  
**Schedule**: Daily at 5:00 AM UTC (12:00 AM EST)  
**Purpose**: Automatically retrain guard prompts based on LangSmith false positive feedback

## Workflow Flow
```
Schedule Trigger → Set Config → Get Feedback → Check Records → 
Get Original Prompt → Loop Feedback → Get Run Details → Get Run Feedback → 
First LLM Analysis → Extract Analysis → Second LLM Refinement → 
Create Google Doc → Log Success
```

## Key Nodes Configuration

### 1. Daily Schedule Trigger
- **Cron**: `0 5 * * *` (5 AM UTC = 12 AM EST)
- **Type**: Schedule Trigger

### 2. Set Date Range & Config
**Critical Variables**:
```json
{
  "min_created_at": "={{ $now.minus({ days: 7 }).startOf('day').toUTC().toISO() }}",
  "max_created_at": "={{ $now.minus({ days: 7 }).endOf('day').toUTC().toISO() }}",
  "guard_name": "unusual_prompt_agentic",
  "prompt_file_name": "unusual_prompt_agentic.txt",
  "google_drive_folder_id": "YOUR_FOLDER_ID",
  "google_sheets_spreadsheet_id": "YOUR_SPREADSHEET_ID",
  "google_sheets_sheet_name": "Guard_Retraining_Log"
}
```

### 3. Get False Positive Feedback
- **URL**: `https://api.smith.langchain.com/feedback`
- **Auth**: `x-api-key` header with `{{ $env.LANGSMITH_API_KEY }}`
- **Query Params**: 
  - `min_created_at`, `max_created_at`
  - `key: "False Positive"`
  - `limit: 100`
- **Retry**: 3 attempts

### 4. Check for Records
- **Condition**: `{{ $json.length > 0 }}`
- **TRUE**: Proceed with processing
- **FALSE**: Log "No Records Found"

### 5. Get Original Guard Prompt
- **Operation**: Download file from Google Drive
- **File**: `unusual_prompt_agentic.txt`
- **Folder**: Configured Google Drive folder

### 6. Loop Through Feedback
- **Type**: Split In Batches
- **Batch Size**: 1 (process each feedback item individually)

### 7. Get Run Details
- **URL**: `https://api.smith.langchain.com/runs/{{ $json.run_id }}`
- **Auth**: `x-api-key` header
- **Purpose**: Get original inputs that caused failure

### 8. Get All Feedback for Run
- **URL**: `https://api.smith.langchain.com/feedback`
- **Query**: `run: {{ $json.run_id }}`
- **Purpose**: Get all feedback comments for context

### 9. First LLM Call - Analysis
- **Service**: Google Vertex AI / Gemini
- **Model**: `gemini-1.5-flash`
- **Temperature**: 0.3
- **Max Tokens**: 2048
- **Purpose**: Analyze failure and generate corrected example

### 10. Extract Analysis Result
- **Operation**: Parse JSON from LLM response
- **Output**: Structured analysis with reasoning and new example

### 11. Second LLM Call - Prompt Refinement
- **Service**: Google Vertex AI / Gemini
- **Model**: `gemini-1.5-flash`
- **Temperature**: 0.2
- **Max Tokens**: 4096
- **Purpose**: Refine original prompt with new examples

### 12. Create Google Doc
- **Title**: `{{ guard_name }} - Prompt Update - {{ $now.toFormat('yyyy-MM-dd') }}`
- **Content**: LLM refinement output
- **Folder**: Configured Google Drive folder

### 13. Log Success to Sheet
- **Sheet**: "Guard_Retraining_Log"
- **Columns**: Date, Status, Guard Name, Result, Doc Link

### 14. Log No Records
- **Sheet**: "Guard_Retraining_Log"
- **Result**: "No Records Found"

### 15. Send Error Email
- **Service**: Gmail
- **To**: ayoung@questrade.com
- **Subject**: "Guard Prompt Retraining Workflow Failed"
- **Content**: Workflow name, failed node, error details
- **Auth**: OAuth2 (uses authenticated Gmail account)

## Environment Variables Required
```env
LANGSMITH_API_KEY=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
LANGSMITH_PROJECT=Prod_CSS_Service_AI
GOOGLE_CLOUD_PROJECT_ID=your-project-id
# Gmail uses OAuth2 authentication - no additional env vars needed
```

## Credentials Required
1. **LangSmith API**: Generic API credential with `x-api-key` header
2. **Google Drive**: OAuth2 credential
3. **Google Sheets**: OAuth2 credential
4. **Google Vertex AI**: OAuth2 credential (or service account)
5. **Gmail**: OAuth2 credential

## Monitoring Points
- **Daily Execution**: Check n8n execution history
- **Success Logs**: Google Sheets "Guard_Retraining_Log"
- **Error Notifications**: Email to ayoung@questrade.com
- **Generated Docs**: Google Drive folder

## Common Issues & Solutions

### No Feedback Found
- Check date range calculation
- Verify LangSmith project has data
- Confirm API key permissions

### Google Drive Access Issues
- Verify OAuth2 credentials
- Check file/folder permissions
- Ensure file exists in specified location

### LLM Errors
- Verify Google Cloud project ID
- Check Vertex AI API enablement
- Confirm service account permissions

### Gmail Notifications
- Verify Gmail OAuth2 credentials
- Check Gmail API permissions
- Test email delivery

## Performance Notes
- **Processing Time**: ~5-10 minutes for 100 feedback items
- **API Limits**: 100 feedback items per run
- **LLM Costs**: ~$0.01-0.05 per analysis
- **Storage**: Google Docs created daily

## Maintenance Tasks
- **Weekly**: Review execution logs
- **Monthly**: Update guard prompts based on generated improvements
- **Quarterly**: Review and optimize workflow performance
- **Annually**: Rotate API keys and credentials

## Customization Points
- **Guard Name**: Change in "Set Date Range & Config" node
- **Schedule**: Modify cron expression in trigger
- **Batch Size**: Adjust feedback processing limit
- **LLM Model**: Change Vertex AI model selection
- **Email Recipients**: Update error notification settings

## File Structure
```
N8N_Langsmith/
├── flows/
│   └── guard-prompt-retraining.json
├── samples/
│   └── unusual_prompt_agentic.txt
├── GUARD_RETRAINING_SETUP.md
├── WORKFLOW_REFERENCE.md
└── env.example
```

## Support Resources
- [LangSmith API Docs](https://api.smith.langchain.com/docs)
- [N8N Documentation](https://docs.n8n.io/)
- [Google Cloud Vertex AI](https://cloud.google.com/vertex-ai)
- [Google Drive API](https://developers.google.com/drive/api)
- [Google Sheets API](https://developers.google.com/sheets/api) 