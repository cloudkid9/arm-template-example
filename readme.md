# Introduction

- This guide is part of an Azure ARM template blog - URL.

# Setup 

```powershell
$ACC_NAME="testvnetlabssa"
$RG_NAME="templates-rg"
$LOC_NAME="Australia East"
$FILE_CONTAINER="templates"

az group create -l $LOC_NAME -n $RG_NAME --tags project=blog

az storage account create --name $ACC_NAME  `
    --resource-group $RG_NAME `
    --sku Standard_LRS --location $LOC_NAME 

$env:AZURE_STORAGE_ACCOUNT=$ACC_NAME
$env:AZURE_STORAGE_KEY=(az storage account keys list `
    --account-name $ACC_NAME `
    --resource-group $RG_NAME  | ConvertFrom-Json)[0].valueaz storage container create --name $FILE_CONTAINER

az storage container create --name $FILE_CONTAINER
```


# Deployment

```powershell
$ACC_NAME="testvnetlabssa"
$RG_NAME="templates-rg"
$LOC_NAME="Australia East"
$FILE_CONTAINER="templates"

$env:AZURE_STORAGE_ACCOUNT=$ACC_NAME
$env:AZURE_STORAGE_KEY=(az storage account keys list `
    --account-name $ACC_NAME `
    --resource-group $RG_NAME  | ConvertFrom-Json)[0].value
    
Get-ChildItem -Path ./templates/* -Include *.json |  foreach-Object `
{ `
    az storage blob upload -c $FILE_CONTAINER -f $_ -n $_.Name `
} 

$end=(Get-Date).AddDays(1).ToString("yyyy-MM-dd")

$sas = az storage container generate-sas -n $FILE_CONTAINER `
    --https-only --permissions lr --expiry $end 

az deployment create -n testdeployment --template-file  deploy.json `
    -l "Australia East" --parameters deploy.parameters.json `
    --parameters sasToken=$sas
    
# Deleting the deployment
az deployment delete -n tdeploym 
```