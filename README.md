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
# Error Logging Sub-Flow

## Overview
This sub-flow creates error log records when placed in fault paths of parent flows. It accepts parameters to populate a custom Error Log object with comprehensive error tracking information.

## Sub-Flow Configuration

### Input Variables
Create the following input variables in your sub-flow:

| Variable Name | Data Type | Required | Description |
|---------------|-----------|----------|-------------|
| `varErrorType` | Text | Yes | Category of error (Apex, Flow, LWC, Integration) |
| `varErrorSource` | Text | Yes | Specific source (Class, Trigger, Flow, etc.) |
| `varErrorMessage` | Text | Yes | The actual error message |
| `varErrorDetails` | Text | No | Stack trace and technical details |
| `varContext` | Text | No | Additional context about what was happening |
| `varAffectedUserId` | Text | No | User ID who experienced the error |
| `varBusinessImpact` | Text | No | Severity level (Critical, High, Medium, Low) |
| `varEnvironment` | Text | No | Environment where error occurred |
| `varAPIEndpoint` | Text | No | For integration errors |
| `varExternalSystem` | Text | No | Name of external system involved |
| `varObjectType` | Text | No | Salesforce object being processed |
| `varRecordId` | Text | No | Specific record that caused the error |

### Sub-Flow Elements

#### 1. Create Error Hash Formula
```
Element Type: Assignment
Variable: varErrorHash
Value: LEFT(BLANKVALUE({!varErrorMessage}, '') + BLANKVALUE({!varErrorSource}, ''), 255)
```

#### 2. Query Existing Error
```
Element Type: Get Records
Object: Error_Log__c
Conditions: Error_Hash__c = {!varErrorHash}
How Many: First record only
Store in: varExistingError
```

#### 3. Decision: Error Exists?
```
Element Type: Decision
Outcome 1: Error Exists
  Condition: {!varExistingError} Is Null = False
Outcome 2: New Error (Default)
```

#### 4A. Update Existing Error (Error Exists Path)
```
Element Type: Update Records
Record: {!varExistingError}
Fields to Update:
  - Error_Count__c = {!varExistingError.Error_Count__c} + 1
  - Last_Occurrence__c = {!$System.OriginDateTime}
  - Error_Message__c = {!varErrorMessage}
  - Error_Details__c = {!varErrorDetails}
  - Context__c = {!varContext}
```

#### 4B. Create New Error Record
```
Element Type: Create Records
Object: Error_Log__c
Fields:
  - Error_Type__c = {!varErrorType}
  - Error_Source__c = {!varErrorSource}
  - Error_Message__c = {!varErrorMessage}
  - Error_Details__c = {!varErrorDetails}
  - Context__c = {!varContext}
  - Error_Hash__c = {!varErrorHash}
  - Error_Count__c = 1
  - Affected_User__c = {!varAffectedUserId}
  - Business_Impact__c = {!varBusinessImpact}
  - Environment__c = {!varEnvironment}
  - First_Occurrence__c = {!$System.OriginDateTime}
  - Last_Occurrence__c = {!$System.OriginDateTime}
  - API_Endpoint__c = {!varAPIEndpoint}
  - External_System__c = {!varExternalSystem}
  - Object_Type__c = {!varObjectType}
  - Record_ID__c = {!varRecordId}
Store in: varNewErrorRecord
```

## Usage in Parent Flow

### In Fault Path
Add this sub-flow element to your fault path:

```
Element Type: Sub-flow
Sub-flow: Log_Error_Sub_Flow
Input Variables:
  - varErrorType = "Flow"
  - varErrorSource = "Account_Update_Flow"
  - varErrorMessage = {!$Flow.FaultMessage}
  - varErrorDetails = "Flow API Name: " + {!$Flow.APIName} + "\nInterview GUID: " + {!$Flow.InterviewGuid}
  - varContext = "Processing Account ID: " + {!recordId}
  - varAffectedUserId = {!$User.Id}
  - varBusinessImpact = "Medium"
  - varEnvironment = "Production"
  - varObjectType = "Account"
  - varRecordId = {!recordId}
```

### Example Implementation
```
Parent Flow: Account_Update_Flow
├── Get Account Record
├── Update Account
│   ├── Success Path → End
│   └── Fault Path → Log Error Sub-Flow
│       ├── Input: varErrorType = "Flow"
│       ├── Input: varErrorSource = "Account_Update_Flow"
│       ├── Input: varErrorMessage = {!$Flow.FaultMessage}
│       └── Input: varContext = "Account ID: " + {!recordId}
└── End (Fault)
```

## Benefits
- **Centralized Error Logging**: One sub-flow handles all error logging
- **Error Deduplication**: Groups similar errors using hash
- **Occurrence Tracking**: Counts repeated errors
- **Rich Context**: Captures comprehensive error information
- **Reusable**: Can be used across multiple flows
- **Maintainable**: Single point of maintenance for error logging logic

## Best Practices
1. Always include meaningful context in the `varContext` parameter
2. Set appropriate `varBusinessImpact` based on the operation
3. Include `varRecordId` when processing specific records
4. Use consistent `varErrorSource` naming conventions
5. Consider adding custom validation or notification logic in the sub-flow

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
