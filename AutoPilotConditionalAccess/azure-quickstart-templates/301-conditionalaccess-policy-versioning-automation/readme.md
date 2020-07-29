# Tutorial:	Easy Versioning of Conditional Acess Policies

Intent: As an IT admin, I want to be able to version control all conditional access policies changes so I can roll-back to a previous version when needed.

You can use the conditional access APIs to manage policy versioning that you can then use to roll-back from non-compliant policy changes. For example, you can:

As a IT admin, add, update or delete a conditional access policy using conditional APIs. The cretion of conditional access policy will create a versioning-enabled policy file within onedrive. Any updates to the conditional access policy will update the relevant file with new policy version. 

This automation can be very useful for: 
- Organizations that manages large numbers of conditional access policies. OR
- Identity partners that manages policy backups for customers. 

This tutorial shows how to build a [logic app](https://docs.microsoft.com/en-us/azure/logic-apps/) that automates policy versioning to Onedrive. Specifically, this logic app monitors the audit logs for policy changes made using conditional access APIs and either creates a new versioning-enabled file or updates exisiting file version.

In this tutorial, you learn how to:

:heavy_check_mark: Deploy this logic app to your organization.  <br /> 
:heavy_check_mark: Authenticate your logic app to Azure AD with the right permissions.  <br /> 
:heavy_check_mark: Add parameters specific to your organization within logic app.  <br /> 
:heavy_check_mark: Add the Recurrence trigger to run a schedule workflow task.<br /> 
:heavy_check_mark: Get client secret from key vault using managed identity.<br /> 
:heavy_check_mark: Get audit logs for CRUD operation on conditional access policies.<br /> 
:heavy_check_mark: Add a condition that checks if any CRUD operations were returned.<br /> 
:heavy_check_mark: Add a switch statement that checks whether the conditional access policy was added, updated or deleted.<br /> 
:heavy_check_mark: Add a check to find if the newly created conditional access policy has already been version enabled.<br /> 
:heavy_check_mark: Initialize versioning of newly created conditional access policy in Onedrive. <br /> 
:heavy_check_mark: Update the policy file to new version. <br /> 

When you're done, your logic app looks like this workflow at a high level:

![High-level finished logic app overview](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-backup-automation/images/backup0.PNG)

# Pre-requisites

If you don't have an Azure subscription, create a [free Azure account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you start.

### The contents of this repository are unsupported and may or not be current. Replies to questions about unsupported material have the lowest priority.

# Why unsupported?
Unsupported samples and documentation are provided for our fans and partners for training, and feedback only.

# Step 1: Deploy this logic app to your organization

Follow the option that you want to use for deploying the quickstart template:

| Option | Description |
|--------|-------------|
| [Azure portal](../logic-apps/quickstart-create-deploy-azure-resource-manager-template.md?tabs=azure-portal#deploy-template) | If your Azure environment meets the prerequisites, and you're familiar with using ARM templates, these steps help you sign in directly to Azure and open the quickstart template in the Azure portal. For more information, see [Deploy resources with ARM templates and Azure portal](../azure-resource-manager/templates/deploy-portal.md). |
| [Azure CLI](../logic-apps/quickstart-create-deploy-azure-resource-manager-template.md?tabs=azure-cli#deploy-template) | The Azure command-line interface (Azure CLI) is a set of commands for creating and managing Azure resources. To run these commands, you need Azure CLI version 2.6 or later. To check your CLI version, type `az --version`. For more information, see these topics: <p><p>- [What is Azure CLI](https://docs.microsoft.com/cli/azure/what-is-azure-cli?view=azure-cli-latest) <br>- [Get started with Azure CLI](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli?view=azure-cli-latest) |
| [Azure PowerShell](../logic-apps/quickstart-create-deploy-azure-resource-manager-template.md?tabs=azure-powershell#deploy-template) | Azure PowerShell provides a set of cmdlets that use the Azure Resource Manager model for managing your Azure resources. For more information, see these topics: <p><p>- [Azure PowerShell Overview](https://docs.microsoft.com/powershell/azure/azurerm/overview) <br>- [Introducing the Azure PowerShell Az module](https://docs.microsoft.com/powershell/azure/new-azureps-module-az) <br>- [Get started with Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps) |
|||

<a name="deploy-azure-portal"></a>

#### [Azure Portal](#tab/azure-portal)

1. Select the following image to sign in with your Azure account and open the logic app in the Azure portal:

   [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fvideor%2FAutoPilotConditionalAccess%2Fmaster%2FAutoPilotConditionalAccess%2Fazure-quickstart-templates%2F301-conditionalaccess-policy-backup-automation%2Fazuredeploy.json)

1. In the portal, on the **Custom deployment** page, enter or select these values:

   | Property | Value | Description |
   |----------|-------|-------------|
   | **Subscription** | <*Azure-subscription-name*> | The name for the Azure subscription to use |
   | **Resource group** | <*Azure-resource-group-name*> | The name for a new or existing Azure resource group. This example uses `AutoPilotConditionalAccess`. |
   | **Location** |  <*Azure-region-for-all-resources*> | The Azure region to use for all resources, if different from the default value. This example uses the default value, `[resourceGroup().location]`, which is the resource group location. |
   | **Logic App Name** | <*logic-app-name*> | The name to use for your logic app. This example uses `301-conditionalaccess-policy-backup-automation`. |
   ||||

   Here is how the page looks with the values used in this example:

   ![Provide information for quickstart template](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-blueprint-automation/images/Deploy-LA-edit.png)

1. When you're done, select **Purchase**.

#### [CLI](#tab/azure-cli)

```azurecli-interactive
read -p "Enter a project name name to use for generating resource names:" projectName &&
read -p "Enter the location, such as 'westus':" location &&
templateUri="https://raw.githubusercontent.com/videor/AutoPilotConditionalAccess/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-backup-automation/azuredeploy.json" &&
resourceGroupName="${projectName}rg" &&
az group create --name $resourceGroupName --location "$location" &&
az deployment group create --resource-group $resourceGroupName --template-uri  $templateUri &&
echo "Press [ENTER] to continue ..." &&
read
```

For more information, see these topics:

* [Azure CLI: az deployment group](https://docs.microsoft.com/cli/azure/deployment/group)
* [Deploy resources with ARM templates and Azure CLI](../azure-resource-manager/templates/deploy-cli.md)

#### [PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter a project name to use for generating resource names"
$location = Read-Host -Prompt "Enter the location, such as 'westus'"
$templateUri = "https://raw.githubusercontent.com/videor/AutoPilotConditionalAccess/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-backup-automation/azuredeploy.json"

$resourceGroupName = "${projectName}rg"

New-AzResourceGroup -Name $resourceGroupName -Location "$location"
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri

Read-Host -Prompt "Press [ENTER] to continue ..."
```

For more information, see these topics:

* [Azure PowerShell: New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup)
* [Azure PowerShell: New-AzResourceGroupDeployment](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroupdeployment)
* [Deploy resources with ARM templates and Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md)


# Step 2: Authenticate your logic app to Azure AD with the right permissions

This logic app uses managed identity for getting secrets from key vault in order to call conditional access APIs. Please look at [Authenticate your logic app to Azure AD with the right permissions](https://github.com/videor/AutoPilotConditionalAccess/tree/master/AutoPilotConditionalAccess/azure-quickstart-templates/docs) on how to create key vault and connect to managed identity. To learn more on how to use managed identities within Logic App please look at [**Logic Apps and Managed Identities**](https://docs.microsoft.com/en-us/azure/logic-apps/create-managed-service-identity) .

1. In the left-hand navigation pane, select Identity > User Assigned > Select Add.

2. Select the User-assigned managed identity from the context pane that appears on the right, select Add.

![](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-blueprint-automation/images/blueprintMI-edit.png).

# Step 3: Update parameters

1. In the left-hand navigation pane, select Logic App designer > Parameters > Replace the default value with Key Vault URI, Client ID and Tenant ID.

![](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-blueprint-automation/images/blueprint-parameters-edit.png)


# Step 4: Add the Recurrence trigger

1. On the Logic App Designer, click `recurrence` box. This example uses a recurrence trigger.

   ![Change the Recurrence trigger's interval and frequency](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-backup-automation/images/backup1-edit.png)

   | Property | Required | Value | Description |
   |----------|----------|-------|-------------|
   | **Interval** | Yes | 1 | The number of intervals to wait between checks |
   | **Frequency** | Yes | Minute | The unit of time to use for the recurrence |
   |||||
      
 This trigger fires every 1 minute, starting at time provided in `start time`. The **recurrence** box shows the recurrence schedule. For more information, see [Schedule tasks and workflows](https://docs.microsoft.com/en-us/azure/connectors/connectors-native-recurrence) and [Workflow actions and triggers](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-workflow-actions-triggers#recurrence-trigger).
 
Sometimes, you might want to run operations on data in your workflow, and then use the results in later actions. To save these results so that you can easily reuse or reference them, you can create variables to store those results after processing them. You can create variables only at the top level in your logic app.

By default, the previous **recurrence** action returns the time in seconds when the workflow has started. By converting and storing this value as a range within the audit logs endpoint filter, you make the value easier to reuse later without converting again. Similarly, storing old and new values of conditional access policy JSON, we can reuse these values in different parts of workflow. 

2. On the Logic App Designer, click `Audit logs endpoint for conditional access policies` box. Provide the details for your variable as described here:

   | Property | Required | Value | Description |
   |----------|----------|-------|-------------|
   | **Name** | Yes | URL | The name for your variable. This example uses "URL". |
   | **Type** | Yes | String | The data type for your variable |
   | **Value** | No| MS graph audit logs endpoint. | The initial value for your variable |
   ||||
   
3. On the Logic App Designer, click `Old policy JSON before change is made to conditional access policy` box. Provide the details for your variable as described here:

   | Property | Required | Value | Description |
   |----------|----------|-------|-------------|
   | **Name** | Yes | OldJSON | The name for your variable. This example uses "OldJSON". |
   | **Type** | Yes | String | The data type for your variable |
   | **Value** | No| empty | The initial value for your variable |
   ||||
   
3. On the Logic App Designer, click `New policy JSON after change is made to conditional access policy` box. Provide the details for your variable as described here: 

   | Property | Required | Value | Description |
   |----------|----------|-------|-------------|
   | **Name** | Yes | NewJSON | The name for your variable. This example uses "NewJSON". |
   | **Type** | Yes | String | The data type for your variable |
   | **Value** | No| empty | The initial value for your variable |
   ||||

# Step 5: Get client secret from key vault using managed identity.

1. On the Logic App Designer, in the HTTP connection box, click `GET client secret from key vault using managed identity`. This example uses HTTP connector.

1. Specify the Method, URI, Queries, Authentication type, Managed Identity and Audience.

   ![Select "GET client secret from key vault using managed identity" HTTP connector](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-backup-automation/images/backup2-edit.png)

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Method** | `GET` | Method to call |
      | **URI** | `AutoPilotConditionalAccessKeyVaultClientCredentials` | The key vault parameter URI configured in step 3  |
      | **Queries** | `2016-10-01` | api-version |
      | **Authentication type** | `Managed Identity` | Authentication type is managed identity |
      | **Managed Identity** | `AutoPilotCAUAI1` | User assigned managed identity connected in step 2 |
      | **Audience** | `https://vault.azure.net` | Key vault |
      ||||
    
1. Response from Key vault is parsed.

# Step 6: Get audit logs for CRUD operation on conditional access policies.

1. On the Logic App Designer, in the HTTP connection box, click `Get audit logs for CRUD operation on conditional access policies`. This example uses HTTP connector.

1. Specify the Method, URI, Headers, Body, Authentication type, Tenant, Audience, Client ID, Credential Type and Secret.

   ![Select "GET audit logs for CRUD operation on conditional access policies" HTTP connector](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-backup-automation/images/backup3-edit.png)

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Method** | `GET` | Method to call |
      | **URI** | `URL variable` | Audit Logs API v1.0 endpoint |
      | **Headers** | `application/json` | Content-Type |
      | **Authentication type** | `Active Directory OAuth` | Authentication type for App-only flow |
      | **Tenant** | `TenantID` | Tennat ID configured in step 3 |
      | **Audience** | `https://graph.microsoft.com` | MS Graph |
      | **Client ID** | `Client ID` | Client ID configured in step 3 |
      | **Credential Type** | `Secret` | Client Secret  |
      | **Secret** | `value` | Secret value retrieved from key vault |
      ||||
      
# Step 7: Add a condition that checks if any CRUD operations were returned.

1. On the Logic App Designer, in the Condition box, verify response. This example uses response from earlier HTTP connector:

1. Specify the HTTP response to evaluate, expression and verify the greater than or equal to `1` response in the condition.
    
      ![Select "Condition to check the response from get audit logs HTTP connector"](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-backup-automation/images/backup4-edit.png)
    
      | Property | Value | Description |
      |----------|-------|-------------|
      | **Length** | `length of array` | The length of array response to evaluate |
      | **Expression** | `is greater than or equal to` | The expression to evaluate |
      | **Condition** | `1` | response to verify in the condition |
      ||||

# Step 8: Add a switch statement that checks whether the conditional access policy was added, updated or deleted.

1. On the Logic App Designer, in the HTTP connection box, click `Switch`. This example uses logic app switch statement.

1. Specify the Switch On and Case to evaluate. Specify `Add conditional access policy` for case 1, `Update conditional access policy` for case 2 and `Delete conditional access policy` for case 3.

   ![Select "Check if deployment is successful" condition](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-backup-automation/images/backup5-edit.png)

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Switch On** | `Activity display name` | Check the activity display name retrieved from audit logs |
      | **Case 1 equals** | `Add conditional access policy` | Case 1 to check Add operations on conditional access policies |
      | **Case 2 equals** | `Update conditional access policy` | Case 2 to check Update operations on conditional access policies|
      | **Case 3 equals** | `Delete conditional access policy` | Case 3 to check Delete operations on conditional access policies|
      ||||
      
# Step 9: Add a check to find if the newly created conditional access policy has already been backed up.

1. On the Logic App Designer, in the Onedrive connection box, click `find if the newly created conditional access policy has already been backed up`. This example uses OneDrive trigger:

   ![Select "find if the newly created conditional access policy has already been backed up" trigger for Onedrive](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-backup-automation/images/backup6-edit.png)

1. If prompted, sign in to your email account with your credentials so that Logic Apps can create a connection to your Onedrive account.

1. In the trigger, provide the criteria for checking files.

1. Specify the search query, folder, file search mode, and number of files to retrieve.

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Search query** | `id` | The Onedrive folder to monitor |
      | **Folder** | `/ConditionalAccess/Backup` | The Onedrive folder to search |
      | **File search mode** | `OneDriveSearch` | Search mode |
      | **Number of files** | `1` | The number of files to retrieve from search |
      ||||
      
# Step 10: Backup the newly created conditional access policy in Onedrive.

1. On the Logic App Designer, in the Condition box, verify response. This example uses response from earlier HTTP connector:

1. Specify the HTTP response to evaluate, expression and verify the greater than or equal to `1` response in the condition.
    
      ![Select "Condition to check the response from get audit logs HTTP connector"](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-backup-automation/images/backup7-edit.png)
    
      | Property | Value | Description |
      |----------|-------|-------------|
      | **Length** | `length of array` | The length of array response to evaluate |
      | **Expression** | `is greater than or equal to` | The expression to evaluate |
      | **Condition** | `1` | response to verify in the condition |
      ||||      
 
 1. On the Logic App Designer, in the Onedrive connection box, click `create backup of newly created conditional access policy`. This example uses OneDrive trigger:

1. If prompted, sign in to your email account with your credentials so that Logic Apps can create a connection to your Onedrive account.

1. In the trigger, provide the criteria for creating new file in onedrive.

1. Specify the folder path, file name, and file content.

      | Property | Value | Description |
      |----------|-------|-------------|
      | **Folder path** | `/ConditionalAccess/Backup` | The Onedrive folder to backup the policies |
      | **File name** | `[Id] display name.json` | File name |
      | **File content** | `newValue` | The new JSON value retrieved from audit log |
      ||||
      
# Step 11: Update the backup file in Onedrive.

1. On the Logic App Designer, in the Switch box, select case for `Update of conditional access policies`. 

1. Select the Onedrive connection box, click `Get file content`. This example uses OneDrive connector:
    
      ![Select "Condition to check the response from get audit logs HTTP connector"](https://github.com/videor/AutoPilotConditionalAccess/blob/master/AutoPilotConditionalAccess/azure-quickstart-templates/301-conditionalaccess-policy-backup-automation/images/backup8-edit.png)    
 
1. If prompted, sign in to your email account with your credentials so that Logic Apps can create a connection to your Onedrive account.

1. In the trigger, provide the criteria for getting file content from onedrive.

1. Specify the file id.

      | Property | Value | Description |
      |----------|-------|-------------|
      | **File id** | `id` | The Onedrive file id to get content |
      ||||
      
   
1. Select the Onedrive connection box, click `Update file`. This example uses OneDrive connector:    
 
1. If prompted, sign in to your email account with your credentials so that Logic Apps can create a connection to your Onedrive account.

1. In the trigger, provide the criteria for updating file content within onedrive.

1. Specify the file id and file content.

      | Property | Value | Description |
      |----------|-------|-------------|
      | **File id** | `Id` | File id |
      | **File content** | `Outputs` | The new JSON value output from audit log |
      ||||
