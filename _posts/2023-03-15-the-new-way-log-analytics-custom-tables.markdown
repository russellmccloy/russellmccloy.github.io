---
layout: post
title:  "Azure Monitor - Custom Tables, Data Collection Endpoints and Rules in Azure Monitor and Terraform"
date:   2023-03-15 11:26:18 +1000
categories: azure

---

## Summary

After some recent work I though I would post about how to send data to a log analytics workspace custom table using Azure Monitor data collection end points and rules.
Originally I was using the PowerShell shown in [this article](https://learn.microsoft.com/en-au/azure/azure-monitor/logs/data-collector-api?tabs=powershell#sample-requests) to create the custom table automatically and then send data to it. But there is a new way to do this using:

* [Data collection endpoints in Azure Monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/data-collection-endpoint-overview?tabs=portal)
* [Data collection rules in Azure Monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/data-collection-rule-overview)

## Azure Monitor

Azure Monitor is a comprehensive monitoring solution for collecting, analyzing, and responding to telemetry from your cloud and on-premises environments. You can use Azure Monitor to maximize the availability and performance of your applications and services.

Azure Monitor collects and aggregates the data from every layer and component of your system into a common data platform. It correlates data across multiple Azure subscriptions and tenants, in addition to hosting data for other services. Because this data is stored together, it can be correlated and analyzed using a common set of tools. The data can then be used for analysis and visualizations to help you understand how your applications are performing and respond automatically to system events.

![Azure Monitor high level architecture](/assets/azure_monitor_high_level_architecture.png)

## Data sources

Azure Monitor can collect data from multiple sources, including from your application, operating systems, the services they rely on, and from the platform itself.

You can integrate monitoring data from sources outside Azure, including on-premises and other non-Microsoft clouds, using the application, infrastructure, and custom data sources.

Azure Monitor collects these types of data:

| Data Type      | Description |
|----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Application    | Data about the performance and functionality of your application code on any platform.|
| Infrastructure | **Container**. Data about containers, such as Azure Kubernetes Service, Prometheus, and about the applications running inside containers. <br />**Operating system**. Data about the guest operating system on which your application is running.   |
| Azure Platform | **Azure resource**. The operation of an Azure resource. <br />**Azure subscription**. The operation and management of an Azure subscription, and data about the health and operation of Azure itself. <br />**Azure tenant**. Data about the operation of tenant-level Azure services, such as Azure Active Directory. <br />**Azure resource changes**. Data about changes within your Azure resources and how to address and triage incidents and issues. |
| **Custom Sources** | Use the Azure Monitor REST API to send customer metric or log data to Azure Monitor and incorporate monitoring of resources that don’t expose monitoring data through other methods.  |

## Deployed Azure Resources

For this post we will be using a custom source and using the Azure Monitor REST API.

The final result in Azure will look as follows:

The data collection end point and rule in a resource group:
![Azure Monitor data collection rule and endpoint](/assets/azure_monitor_data_collection_rule_endpoint.png)

The custom table we created:
![Log Analytics Custom Table](/assets/azure_monitor_custom_table.png)

The data we sent to our custom table (`MyData_CL`) will be able to be queried in Azure Monitor:
![Azure Monitor query](/assets/azure_monitor_query.png)

## How I did it

### Steps

- Write the Terraform code to deploy the resources
- Write to PowerShell code to send data to the custom table

### Terraform

For the Terraform code you will need the following resources:

- [azapi_resource](https://registry.terraform.io/providers/Azure/azapi/latest/docs/resources/azapi_resource) - for the custom table
- [azurerm_monitor_data_collection_endpoint](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/monitor_data_collection_endpoint)
- [azapi_resource](https://registry.terraform.io/providers/Azure/azapi/latest/docs/resources/azapi_resource) - for the data collection rule
- [azurerm_role_assignment](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/role_assignment)

**Note:** There is a terraform mdule for the Data Collection Rule [here](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/monitor_data_collection_rule) but it is not flexible enough to create the data collection rule

#### Custom Table

>truncated for the purpose of this post and i appologise as it is not the full code solution, it's just to give you an outline of what to do

```hcl
resource "azapi_resource" "my_data_table" {

  type      = "Microsoft.OperationalInsights/workspaces/tables@2022-10-01"
  name      = "MyData_CL"
  parent_id = "The id of the log analytics workspace"

  body = jsonencode(
    {
      "properties" : {
        "schema" : {
          "name" : "MyData_CL",
          "columns" : [
            {
              "name" : "TimeGenerated",
              "type" : "datetime"
            },
            {
              "name" : "RawData",
              "type" : "string"
            },
            {
              "name" : "customerId",
              "type" : "string"
            },
            {
              "name" : "correlationId",
              "type" : "string"
            }
          ]
        },
        "retentionInDays" : 30,
        "totalRetentionInDays" : 30
      }
    }
  )
  response_export_values = ["id"]
}
```

#### Data Collection Endpoint

```hcl
resource "azurerm_monitor_data_collection_endpoint" "my_data_collection_endpoint" {

  name                          = "my-data-collection-endpoint"
  resource_group_name           = "name of the parent reource group"
  location                      = "the Azure region"
  kind                          = "Windows"
  public_network_access_enabled = true
  description                   = "useful desription here"
}
```

```hcl
resource "azapi_resource" "my_data_collection_rule" {

  type = "Microsoft.Insights/dataCollectionRules@2021-09-01-preview"
  name = "my-data-collection-rule"

  parent_id = "id of the parent reource group"
  body = jsonencode(
    {
      "location" = "the Azure region",
      "properties" : {
        "dataCollectionEndpointId" : "https://my-data-collection-endpoint-n0d1.australiasoutheast-1.ingest.monitor.azure.com",
        "streamDeclarations" : {
          "Custom-MyData_CL" : {
            "columns" : [
            {
              "name" : "TimeGenerated",
              "type" : "datetime"
            },
            {
              "name" : "RawData",
              "type" : "string"
            },
            {
              "name" : "customerId",
              "type" : "string"
            },
            {
              "name" : "correlationId",
              "type" : "string"
            }            ]
          }
        },
        "dataSources" : {},
        "destinations" : {
          "logAnalytics" : [
            {
              "workspaceResourceId" : "The id of the log analytics workspace",
              "name" : "la--55718050" # not sure what this is?
            }
          ]
        },
        "dataFlows" : [
          {
            "streams" : [
              "Custom-MyData_CL"
            ],
            "destinations" : [
              "la--55718050" # not sure what this is?
            ],
            "transformKql" : "source",
            "outputStream" : "Custom-MyData_CL"
          }
        ]
      }
    }
  )
  response_export_values = ["id", "properties.immutableId"]

  depends_on = [
    azurerm_monitor_data_collection_endpoint.my_data_collection_endpoint,
    azapi_resource.my_data_table
  ]
}
```

#### Role Assignment

The service pricipal you are using needs the **Monitoring Metrics Publisher** role assignment on the Data Collection Rule:

```hcl
resource "azurerm_role_assignment" "my_role_assignment" {

  scope                = azapi_resource.my_data_collection_rule.id
  role_definition_name = "Monitoring Metrics Publisher"
  principal_id         = data.azuread_service_principal.this.object_id
}
```

### PowerShell

Now that we have the Azure resources deployed we need to send data to the Custom Table and this can be done through the Azure Monitor REST API. You can use any language you want but we will use PowerShell here.

For the PowerShell you will need the following methods:

- A method to get a Bearer token given the Azure Service Principal you are using
- A method to send the payload to the Azure Monitor REST API
- A Method to construct your custom payload

#### Get Bearer Token

```powershell
# Creates the authorization signature for posting data to a log analytics custom table
Function Build-Signature ($spnAppId, $spnSecret, $tenantId) {

    ## Obtain a bearer token used to authenticate against the data collection endpoint
    $scope = [System.Web.HttpUtility]::UrlEncode("https://monitor.azure.com//.default")   
    $body = "client_id=$spnAppId&scope=$scope&client_secret=$spnSecret&grant_type=client_credentials";
    $headers = @{"Content-Type" = "application/x-www-form-urlencoded" };
    $uri = "https://login.microsoftonline.com/$tenantId/oauth2/v2.0/token"
    $bearerToken = (Invoke-RestMethod -Uri $Uri -Method "Post" -Body $body -Headers $headers).access_token

    return $bearerToken
}
```

#### Send Payload to the Azure Monitor REST API

```powershell
# Creates the request to log data to a log analytics custom table
Function Set-LogAnalyticsData($bearerToken, $dceURI, $dcrImmutableId, $body, $customTableName) {
            
    try {
        $Body = $body | ConvertTo-Json -AsArray;
        
        $headers = @{"Authorization" = "Bearer $bearerToken"; "Content-Type" = "application/json" }
        $uri = "$dceURI/dataCollectionRules/$dcrImmutableId/streams/Custom-$($customTableName)?api-version=2021-11-01-preview"

        $ingestLogResponse = Invoke-RestMethod -Uri $uri -Method "Post" -Body $body -Headers $headers
    }
    Catch {
        $errorMessage = ($_ -split "\n")[0]
        Write-Error $errorMessage
        throw
    }
    return $ingestLogResponse.StatusCode
}
```

#### Construct your custom payload

```powershell
# Sends data to the log analytics custom table
Function Invoke-LogAnalytics () {
    param (
        [string]$spnAppId,
        [string]$spnSecret,
        [string]$tenantId,
        [string]$dceURI, 
        [string]$dcrImmutableId,

        [string]$customerId,
        [string]$correlationId,
        [string]$customLawTableName
    )

    $body = @{ 
        "customerId"           = "$($customerId)"
        "correlationId"        = "$($correlationId.ToString())"
        "TimeGenerated"        = Get-Date ([datetime]::UtcNow) -Format O
    }
            
    try {
        Write-Verbose "[$((get-date).TimeOfDay.ToString()) INFORMATION] Writing alert messsage for customer '$($customerId)' with correlationId '$($correlationId)' to the log analytics workspace custom table: : '$($customLawTableName)'"
        $bearerToken = Build-Signature -spnAppId $spnAppId -spnSecret $spnSecret -TenantId $tenantId
        Set-LogAnalyticsData -BearerToken $bearerToken -dceURI $dceURI -dcrImmutableId $dcrImmutableId -body $body -CustomTableName $customLawTableName
   
                   
        Write-Verbose "[$((get-date).TimeOfDay.ToString()) INFORMATION] Wrote alert messsage for customer '$($customerId)' with correlationId '$($correlationId)' to the log analytics workspace custom table: '$($customLawTableName)'"
    }
    Catch {
        $errorMessage = ($_ -split "\n")[0]
        Write-Error "[$((get-date).TimeOfDay.ToString()) ERROR] $errorMessage"
        throw
    }
}
```

**Note:** You may have noticed references to the data collection rule `immutableId` and `[string]$dcrImmutableId`. You can find the immutableId of the data collection rule here:

![azure_monitor_immutable_id.png](/assets/azure_monitor_immutable_id.png)

Ok, that is all and I hope this helps someone.

> PS. Shout out to [Jin Tang](https://www.linkedin.com/in/jin-tang-45b76b91/) for being an absolute legend.