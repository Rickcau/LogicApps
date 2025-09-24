# Azure Logic Apps Standard host.json Configuration Guide

Azure Logic Apps Standard's host.json file serves as the **central runtime configuration hub** that controls workflow execution, performance, and operational behavior across all workflows within a Logic Apps resource. Unlike app settings that only apply locally, host.json settings affect both development and deployed environments, making proper configuration critical for optimal performance.

## Exact file location and structure

### Local development environment
In Visual Studio Code projects, host.json is located at the **project root level** alongside other core configuration files. The typical project structure follows this hierarchy:

```
MyWorkspaceName/
├── MyLogicAppProjectName/
│   ├── .vscode/                    (VS Code settings)
│   ├── Artifacts/                  (Integration account artifacts)
│   ├── WorkflowName1/              (Individual workflow folders)
│   │   └── workflow.json
│   ├── workflow-designtime/        (Development environment settings)
│   │   ├── host.json
│   │   └── local.settings.json
│   ├── connections.json            (Connection definitions)
│   ├── host.json                   (Primary runtime configuration)
│   └── local.settings.json         (Local environment settings)
```

The **primary host.json file** at the project root contains runtime settings that apply to all workflows, while a secondary host.json may exist in the workflow-designtime folder for local development-specific settings.

### Deployed environment in Azure
In deployed Logic Apps Standard instances, host.json is located at `/home/site/wwwroot/host.json`. You can access this through the Azure portal by navigating to **Development Tools → Advanced Tools → Go (Kudu) → Debug Console → CMD → site/wwwroot directory**. Azure automatically maintains backup files named `host.snapshot.*.json` with a limit of 10 snapshots.

## Runtime.ScaleUnitsCount configuration with multiple storage accounts

The `Runtime.ScaleUnitsCount` setting enables **horizontal scaling** by distributing workflow data across multiple storage accounts. This configuration is essential for high-throughput scenarios exceeding ~100,000 action executions per minute per storage account.

### Basic configuration example
```json
{
  "version": "2.0",
  "extensionBundle": {
    "id": "Microsoft.Azure.Functions.ExtensionBundle.Workflows",
    "version": "[1.*, 2.0.0)"
  },
  "extensions": {
    "workflow": {
      "settings": {
        "Runtime.ScaleUnitsCount": "3"
      }
    }
  }
}
```

### Required application settings
When configuring multiple storage accounts, you must define connection strings following this naming pattern:

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "DefaultEndpointsProtocol=https;AccountName=storage1;AccountKey=key1;EndpointSuffix=core.windows.net",
    "CloudStorageAccount.Workflows.ScaleUnitsDataStorage.CU00.ConnectionString": "DefaultEndpointsProtocol=https;AccountName=storage1;AccountKey=key1;EndpointSuffix=core.windows.net",
    "CloudStorageAccount.Workflows.ScaleUnitsDataStorage.CU01.ConnectionString": "DefaultEndpointsProtocol=https;AccountName=storage2;AccountKey=key2;EndpointSuffix=core.windows.net",
    "CloudStorageAccount.Workflows.ScaleUnitsDataStorage.CU02.ConnectionString": "DefaultEndpointsProtocol=https;AccountName=storage3;AccountKey=key3;EndpointSuffix=core.windows.net"
  }
}
```

**Critical requirements**: Configure Runtime.ScaleUnitsCount **before creating workflows** to avoid data loss. The maximum supported limit is 32 storage accounts, numbered from 00 to 31.

## Complete host.json configuration examples

### High-performance production configuration
```json
{
  "version": "2.0",
  "extensionBundle": {
    "id": "Microsoft.Azure.Functions.ExtensionBundle.Workflows",
    "version": "[1.*, 2.0.0)"
  },
  "extensions": {
    "workflow": {
      "settings": {
        "Runtime.ScaleUnitsCount": "5",
        "Runtime.Backend.HttpOperation.RequestTimeout": "00:03:45",
        "Runtime.Backend.VariableOperation.MaximumVariableSize": "104857600",
        "Runtime.Trigger.MaximumRunConcurrency": "100",
        "Runtime.Trigger.MaximumWaitingRuns": "200",
        "Runtime.FlowRunRetryableActionJobCallback.ActionJobExecutionTimeout": "00:10:00",
        "Runtime.Backend.ForeachDefaultDegreeOfParallelism": "20",
        "Jobs.BackgroundJobs.NumWorkersPerProcessorCount": "192"
      }
    }
  },
  "functionTimeout": "00:30:00",
  "logging": {
    "applicationInsights": {
      "samplingSettings": {
        "isEnabled": true,
        "excludedTypes": "Request"
      }
    },
    "logLevel": {
      "default": "Information",
      "Host.Results": "Error"
    }
  }
}
```

### Development environment configuration
```json
{
  "version": "2.0",
  "extensionBundle": {
    "id": "Microsoft.Azure.Functions.ExtensionBundle.Workflows",
    "version": "[1.*, 2.0.0)"
  },
  "extensions": {
    "workflow": {
      "settings": {
        "Runtime.Trigger.MaximumRunConcurrency": "10",
        "Runtime.Backend.HttpOperation.RequestTimeout": "00:02:00"
      }
    }
  },
  "logging": {
    "logLevel": {
      "default": "Debug"
    }
  }
}
```

## Editing and deploying host.json changes to existing instances

### Visual Studio Code method (recommended for development)
For local development or downloaded Logic Apps from the portal:
1. Open the Logic App project in VS Code
2. Navigate to the host.json file at the project root
3. **Important**: Save the file first (Ctrl+S) before editing if downloaded from portal
4. Make your configuration changes
5. Deploy using your preferred method (zip deployment, CI/CD pipeline)

### Azure portal direct editing method
For immediate changes to deployed Logic Apps:
1. Navigate to your Logic App in Azure Portal
2. Go to **Development Tools → Advanced Tools**
3. Click **Go** to open Kudu console
4. Select **Debug Console → CMD**
5. Navigate to `/home/site/wwwroot`
6. Click the **edit icon** next to host.json
7. **Critical**: Stop your Logic App before making changes
8. Save changes and restart the Logic App

### DevOps pipeline deployment
For production environments, use automated deployment pipelines:
- Package host.json modifications in deployment artifacts
- Include in zip deployment structure at root level
- Use Azure Functions deployment mechanisms
- Implement environment-specific parameter replacement

## Direct access methods from Azure Portal

When building Logic Apps primarily in the Azure Portal, you can access and modify host.json through several methods, though with important limitations compared to local development.

### Method 1: Kudu Console (Advanced Tools) - Recommended
This is the most reliable method for portal-based host.json editing:

1. Navigate to your Logic Apps Standard resource in Azure Portal
2. Go to **Development Tools → Advanced Tools**
3. Click **Go** to open the Kudu console (this opens in a new tab)
4. Select **Debug Console → CMD** from the top menu
5. Navigate to `/home/site/wwwroot` directory
6. Locate `host.json` in the file listing
7. Click the **edit icon** (pencil) next to `host.json`
8. **Critical**: Stop your Logic App before making changes
9. Edit the file directly in the web-based editor
10. Save changes using Ctrl+S or the Save button
11. Restart the Logic App for changes to take effect

### Method 2: App Service Editor (if available)
Some Logic Apps Standard instances may have App Service Editor enabled:

1. In your Logic Apps resource, look for **Development Tools → App Service Editor**
2. Click **Go** to open the web-based editor (opens in new tab)
3. Navigate to the root directory to find `host.json`
4. Edit directly in the Monaco editor interface
5. Save changes and restart the Logic App

### Method 3: Export-Edit-Deploy approach
For more complex modifications or when you need IntelliSense support:

1. **Export your Logic App**: Go to **Export → Download app content** in the portal
2. **Extract and edit locally**: Open the downloaded ZIP file, edit host.json with any text editor
3. **Deploy back**: Use Azure CLI, PowerShell, or zip deployment to upload changes
4. **Restart** the Logic App after deployment

### Portal access limitations and considerations

**Key Limitations:**
- The main Logic Apps designer interface provides **NO access** to host.json
- No IntelliSense or syntax validation when editing through portal methods
- Changes require Logic App restart to take effect
- No version control or change tracking through portal methods
- Risk of syntax errors that could prevent Logic App startup

**Critical Requirements:**
- **Always stop your Logic App** before modifying host.json to prevent conflicts
- **Restart the Logic App** after making changes for settings to take effect
- **Backup the original file** before making changes (Kudu maintains automatic snapshots)
- **Test changes thoroughly** as syntax errors can prevent Logic App from starting

**When Portal Access is Essential:**
- Immediate performance tuning for saturated App Service Plans
- Emergency configuration changes in production environments  
- Initial setup of `Runtime.ScaleUnitsCount` for multiple storage accounts
- Quick timeout adjustments for specific scenarios

**Best Practice Recommendation:**
For organizations building primarily in the portal but requiring frequent host.json modifications, consider implementing a **hybrid approach**:
- Build and design workflows in the Azure Portal for ease of use
- Use the export-edit-deploy method for host.json changes to get proper editing tools
- Establish a simple CI/CD pipeline for configuration management as your solution matures

This approach maximizes the benefits of both portal-based development and proper configuration management practices.

## Development versus deployed environment differences

### Local development characteristics
- **File access**: Direct editing in VS Code with IntelliSense support
- **Change detection**: Modifications take effect immediately on next debug run
- **Additional files**: May include workflow-designtime/host.json for development-specific settings
- **Testing**: Can test configuration changes locally before deployment

### Deployed environment characteristics  
- **File access**: Requires Kudu console or deployment pipeline for modifications
- **Change activation**: Requires Logic App restart for settings to take effect
- **Backup management**: Azure automatically maintains snapshot history
- **Restrictions**: Read-only access through normal portal interface

The **fundamental difference** is that local development allows immediate iterative testing of configuration changes, while deployed environments require more careful change management with restart procedures.

## Best practices for host.json configuration

### Performance optimization guidelines
**Concurrency settings**: Start with default values (100 concurrent runs, 200 waiting runs) and adjust based on downstream system capacity and performance monitoring.

**Timeout configurations**: Set `Runtime.FlowRunRetryableActionJobCallback.ActionJobExecutionTimeout` to 10 minutes for most scenarios, increasing only for genuinely long-running operations.

**Variable size limits**: The default 100MB limit for `Runtime.Backend.VariableOperation.MaximumVariableSize` is appropriate for most use cases, but can be increased up to 200MB if needed.

### Storage and scaling best practices
**Multiple storage accounts**: Implement when consistently exceeding 100,000 action executions per minute per storage account. Configure **before** creating workflows to prevent data loss.

**Target-based scaling**: Use `Runtime.TargetScaler.TargetScalingFactor` (0.05-1.0 range) to control scaling aggressiveness, with higher values providing more responsive scaling.

### Monitoring and cost optimization
**Application Insights configuration**: Enable sampling with `"excludedTypes": "Request"` to capture all workflow runs while managing ingestion costs. Set log levels to `"Information"` for default logging with `"Error"` for specific noisy components.

**Retention management**: Align `Runtime.Backend.FlowRunTimeout` with your `Workflows.RuntimeConfiguration.RetentionInDays` app setting to ensure consistent data cleanup.

## Visual Studio Code and deployment considerations

### Azure Logic Apps Standard extension integration
The VS Code extension provides **automated deployment script generation** and seamless integration with Azure resources. Key capabilities include:
- Direct deployment to Azure with authentication management
- Local debugging with breakpoints and real-time monitoring
- Automatic connection metadata generation for different environments
- Git integration for source control of configuration changes

### Project type considerations
**Extension Bundle-based projects** (default Node.js type) offer simpler project structure and are suitable for most scenarios. **NuGet Package-based projects** (.NET type) are required only for developing custom built-in operations and include additional .bin folder complexity.

### Deployment artifact management
Ensure your deployment package includes host.json at the root level alongside connections.json and workflow definitions. **Critical deployment sequence**: Always include configuration changes in the same deployment as workflow modifications to maintain consistency.

### Environment-specific configuration strategies
Implement **parameterized configurations** using application settings to override host.json values per environment:
```
AzureFunctionsJobHost__extensions__workflow__settings__Runtime.Trigger.MaximumRunConcurrency = 50
```

This approach enables environment-specific tuning while maintaining a single host.json template across all environments.

## Conclusion

Proper host.json configuration is fundamental to Azure Logic Apps Standard success, directly impacting performance, scalability, and operational reliability. The key to effective configuration lies in understanding the file's dual role in both development and production environments, implementing proper change management procedures, and continuously monitoring performance metrics to optimize settings based on actual usage patterns. Always test configuration changes in development environments first, and remember that modifications to scaling-related settings like Runtime.ScaleUnitsCount must be configured before creating workflows to avoid data loss.
