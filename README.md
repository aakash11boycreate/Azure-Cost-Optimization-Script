# Azure-Cost-Optimization-Script
PowerShell script for automating Azure billing reports and cost optimization
# Automate Azure Billing Reports and Cost Optimization with PowerShell

Welcome to the **Azure Cost Optimization and Automated Billing Reports** repository! This repository provides a powerful PowerShell script to help you automate Azure billing reports, analyze cost patterns, and implement cost-saving recommendations. 

## Table of Contents

- [Introduction](#introduction)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Script Breakdown](#script-breakdown)
- [Advanced Optimization Techniques](#advanced-optimization-techniques)
- [Best Practices](#best-practices)
- [Contributing](#contributing)
- [License](#license)

## Introduction

Managing cloud costs efficiently is crucial for organizations using Microsoft Azure. This repository offers a comprehensive solution for automating Azure billing and cost management. By utilizing Azure Cost Management APIs, Azure Resource Graph, and PowerShell, the script helps you identify high-cost resources, analyze usage patterns, and suggest actionable optimizations to reduce Azure costs.

## Features

- **Automated Azure Billing Reports**: Fetch and compile Azure cost and usage data automatically.
- **Cost Analysis**: Identify high-cost resources and analyze cost data by service.
- **Cost Optimization Recommendations**: Get actionable suggestions, like resizing underutilized VMs or deleting unattached disks.
- **Email Notifications**: Automatically send cost reports and optimization recommendations to stakeholders.
- **Customizable and Extensible**: Easily modify the script to meet your organization’s specific needs.

## Prerequisites

To use this script, you need:

- Azure Cost Management + Billing Contributor role access.
- Azure PowerShell or Azure CLI installed and authenticated.
- Access to Azure Resource Graph for querying resources.

## Getting Started

Follow these steps to start automating your Azure cost reports:

1. **Clone the Repository**:
    ```bash
    git clone https://github.com/yourusername/Azure-Cost-Optimization-Script.git
    cd Azure-Cost-Optimization-Script
    ```

2. **Install Azure PowerShell Module**:
    ```powershell
    Install-Module -Name Az -AllowClobber -Scope CurrentUser
    ```

3. **Run the Script**:
    Update the variables in the script with your Azure Subscription ID, Resource Group Name, and other details. Then run the script in PowerShell.

## Script Breakdown

Below is the PowerShell script that automates Azure billing reports and provides cost optimization recommendations:

```powershell
# Step 1: Connect to Azure
Connect-AzAccount

# Step 2: Set Variables
$subscriptionId = "<Your Subscription ID>"
$resourceGroupName = "<Your Resource Group Name>"
$reportFilePath = "C:\AzureCostReport.csv"
$emailRecipient = "recipient@example.com"
$emailSubject = "Azure Cost Report and Optimization Recommendations"
$smtpServer = "smtp.example.com"

# Step 3: Fetch Cost and Usage Data
$startDate = (Get-Date).AddMonths(-1).ToString("yyyy-MM-01")
$endDate = (Get-Date).ToString("yyyy-MM-dd")

$costData = Get-AzConsumptionUsageDetail -SubscriptionId $subscriptionId -StartDate $startDate -EndDate $endDate

# Step 4: Analyze Cost Data
$highCostResources = $costData | Where-Object { $_.Cost > 100 }
$costByService = $costData | Group-Object -Property ServiceName | Select-Object Name, @{Name="TotalCost";Expression={($_.Group | Measure-Object -Property Cost -Sum).Sum}}

# Step 5: Generate Recommendations
$recommendations = @()

# Check for underutilized VMs
$vms = Get-AzVM -ResourceGroupName $resourceGroupName
foreach ($vm in $vms) {
    $metrics = Get-AzMetric -ResourceId $vm.Id -MetricName "Percentage CPU"
    if ($metrics.Data.Average -lt 20) {
        $recommendations += "VM '$($vm.Name)' has low CPU usage. Consider resizing or shutting down to save costs."
    }
}

# Check for unattached disks
$disks = Get-AzDisk -ResourceGroupName $resourceGroupName | Where-Object { $_.ManagedBy -eq $null }
foreach ($disk in $disks) {
    $recommendations += "Disk '$($disk.Name)' is unattached. Consider deleting to save costs."
}

# Step 6: Export Data to CSV
$reportData = $costByService | Select-Object Name, TotalCost
$reportData | Export-Csv -Path $reportFilePath -NoTypeInformation

# Step 7: Email the Report
$smtp = New-Object System.Net.Mail.SmtpClient($smtpServer)
$emailMessage = New-Object System.Net.Mail.MailMessage
$emailMessage.From = "sender@example.com"
$emailMessage.To.Add($emailRecipient)
$emailMessage.Subject = $emailSubject
$emailMessage.Body = "Attached is your Azure cost report and optimization recommendations." + "`n`n" + ($recommendations -join "`n")
$emailMessage.Attachments.Add($reportFilePath)

$smtp.Send($emailMessage)

Write-Output "Azure Cost Report has been generated and emailed successfully."

# Cleanup
Remove-Item $reportFilePath
