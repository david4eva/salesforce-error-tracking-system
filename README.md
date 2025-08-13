# Salesforce Error Tracking System

🚫 **Stop hunting through exception emails for Salesforce errors**

A comprehensive, production-ready error tracking system for Salesforce that captures errors from Apex, Flow, LWC, and Integrations.

[![Salesforce](https://img.shields.io/badge/Salesforce-Ready-00A1E0.svg)](https://salesforce.com)

## Why This Exists

After analyzing the Salesforce error tracking landscape, I found existing solutions were either:
- ❌ Too complex to set up (weeks of configuration)
- ❌ Too expensive ($50+ per user per org)
- ❌ Scattered across multiple locations
- ❌ Lacking comprehensive error context

This library solves those problems with a **5-minute setup** that works immediately.

## ✨ Features

### 🎯 Universal Error Tracking
- ✅ **Apex Classes & Triggers** - Automatic exception capture with stack traces
- ✅ **Salesforce Flow** - Failed flow interviews and element errors
- ✅ **Lightning Web Components** - JavaScript errors and server exceptions
- ✅ **Integration APIs** - External system failures and timeout errors

### 📊 Production-Ready Features
- ✅ **Centralized error logging**
- ✅ **Error categorization** by type, severity, and source
- ✅ **User impact tracking** - know who's affected
- ✅ **Resolution workflow** - from detection to fix
- ✅ **Knowledge base** - track solutions for recurring issues

## 🚀 Quick Start

### 1. Deploy the Package
```bash
sf package install --package 04t... --target-org your-org
```

### 2. Use Immediately
```apex
// Log any error with one line
ErrorLogger.logError('Apex Exception', 'MyClass.myMethod', ex, 'Processing user data');
```

### 3. View Results
Navigate to the **Error Logs** tab to see captured errors.

## 📋 Installation Options

### Option 1: Managed Package (Recommended)
```bash
sf package install --package 04t5Y0000015pGtQAI
```

### Option 2: Source Code Deployment
1. Clone this repository
2. Deploy to your org:
```bash
sf project deploy start --source-dir force-app --target-org your-org
```

### Option 3: Scratch Org for Testing
```bash
sf org create scratch --definition-file config/project-scratch-def.json
sf project deploy start --source-dir force-app
```

## 🏗️ Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Error Source  │───▶│   Error Logger   │───▶│   Error_Log__c  │
│  (Apex/Flow/    │    │      Class       │    │     Object      │
│   LWC/API)      │    │                  │    │                 │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

## 📋 Error Log Fields

The `Error_Log__c` object captures comprehensive error information:

| Field | Type | Description |
|-------|------|-------------|
| **Error Name** | Auto Number | Unique identifier for each error |
| **Error Type** | Picklist | Category of error (Apex, Flow, LWC, Integration) |
| **Error Source** | Picklist | Specific source (Class, Trigger, Flow, etc.) |
| **Error Message** | Long Text | The actual error message |
| **Error Details** | Long Text | Stack trace and technical details |
| **Context** | Long Text | Additional context about what was happening |
| **Error Hash** | Text(255) | Unique hash for grouping similar errors |
| **Error Count** | Number | How many times this error has occurred |
| **Affected User** | Lookup(User) | User who experienced the error |
| **Assigned To** | Lookup(User) | User responsible for fixing the error |
| **Business Impact** | Picklist | Severity level (Critical, High, Medium, Low) |
| **Environment** | Picklist | Environment where error occurred |
| **First Occurrence** | Date/Time | When this error was first seen |
| **Last Occurrence** | Date/Time | Most recent occurrence |
| **API Endpoint** | URL(255) | For integration errors |
| **External System** | Text(255) | Name of external system involved |
| **Object Type** | Text(255) | Salesforce object being processed |
| **Record ID** | Text(18) | Specific record that caused the error |

## 📖 Usage Examples

### Apex Error Handling
```apex
public class AccountService {
    public static void updateAccounts(List<Account> accounts) {
        try {
            // Your business logic here
            update accounts;
            
        } catch (Exception ex) {
            // Automatic error logging with context
            ErrorLogger.logError('Apex Exception', 'AccountService.updateAccounts', 
                ex, 'Failed updating ' + accounts.size() + ' accounts');
        }
    }
}
```

### Flow Error Handling
Add the **Log Error** Flow element to any Flow to capture failures:
```
Element: Log Error (Invocable Action)
Error Type: Flow Error
Source: [Flow API Name]
Message: {!$Flow.FaultMessage}
Context: Record ID {!recordId}
```

### Batch Job Error Tracking
```apex
public class MyBatchJob implements Database.Batchable<sObject> {
    public void execute(Database.BatchableContext bc, List<sObject> scope) {
        try {
            // Process records...
            
        } catch (Exception ex) {
            ErrorLogger.logError('Batch Error', 'MyBatchJob.execute', 
                ex, 'Batch processing failed for ' + scope.size() + ' records');
        }
    }
}
```

### Integration Error Tracking
```apex
public class APIService {
    public static void callExternalAPI() {
        try {
            HttpResponse response = http.send(request);
            if (response.getStatusCode() != 200) {
                ErrorLogger.logError('Integration Error', 'APIService.callExternalAPI',
                    'HTTP ' + response.getStatusCode() + ': ' + response.getBody(),
                    'Failed calling customer API');
            }
        } catch (Exception ex) {
            ErrorLogger.logError('Integration Error', 'APIService', ex, 'API timeout or connection failure');
        }
    }
}
```

## 📊 Error Management

View and manage errors through the **Error Logs** tab:
- Error trends over time
- Error types and sources
- User impact analysis
- Resolution status tracking

## ⚙️ Configuration

### Custom Settings
Configure behavior via **Error Tracking Settings**:

```apex
Error_Tracking_Settings__c settings = Error_Tracking_Settings__c.getOrgDefaults();
settings.Auto_Assign_Errors__c = true;    // Auto-assign to record creator
settings.Email_Notifications__c = true;   // Send email alerts for critical errors
settings.Max_Stack_Trace_Length__c = 5000; // Truncate long stack traces
upsert settings;
```

### Permission Sets
- **Error Log Admin** - Full access to create, read, update, delete
- **Error Log User** - Read access to errors, create new errors
- **Error Log Viewer** - Read-only access for reporting

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guidelines](CONTRIBUTING.md) for details.

## 📈 Roadmap

### v1.1 - Enhanced Analytics
- [ ] Advanced error clustering and categorization
- [ ] Automated error resolution suggestions

### v1.2 - External Integrations
- [ ] Slack/Teams notifications
- [ ] Jira ticket creation
- [ ] Email digest reports

### v1.3 - Advanced Features
- [ ] Custom error handling workflows
- [ ] API for external monitoring tools

## 🏷️ Keywords

`salesforce` `apex` `error-tracking` `flow` `lwc` `integration` `monitoring` `debugging` `production-ready` `devops` `reliability`

---

🐛 **Found a bug?** [Open an issue](https://github.com/yourusername/salesforce-error-tracker/issues)

💡 **Have an idea?** [Start a discussion](https://github.com/yourusername/salesforce-error-tracker/discussions)

---

*Built with ❤️ for the Salesforce community*
