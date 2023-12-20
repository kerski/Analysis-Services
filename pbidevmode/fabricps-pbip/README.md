# Requirements

The FabricPS-PBIP module has a dependency to Az.Accounts module for authentication into Fabric.

Run the following command before importing FabricPS-PBIP to ensure the "Az.Accounts" is installed in your machine:

```powershell
Install-Module Az.Accounts
```

# Export items from workspace

```powershell

Import-Module ".\FabricPS-PBIP" -Force

# Force authentication prompt
Set-FabricAuthToken -reset

Export-FabricItems -workspaceId "[Workspace Id]" -path '[Export folder file path]'

```

# Import PBIP content to workspace

```powershell

Import-Module ".\FabricPS-PBIP" -Force

Set-FabricAuthToken -reset

Import-FabricItems -workspaceId "[Workspace Id]" -path "[PBIP file path]"

```

# Import PBIP content to multiple Workspaces and file overrides

```powershell

Import-Module ".\FabricPS-PBIP" -Force

Set-FabricAuthToken -reset

$workspaceName = "RR-APIsDemo-DeployPBIP"
$workspaceDatasets = "$workspaceName-Models"
$workspaceReports = "$workspaceName-Reports"

$pbipPath = "$currentPath\SamplePBIP"

# Deploy Dataset

$workspaceId = New-FabricWorkspace  -name $workspaceDatasets -skipErrorIfExists

$deployInfo = Import-FabricItems -workspaceId $workspaceId -path $pbipPath -filter "*\sales.dataset"

# Deploy Report

$workspaceId = New-FabricWorkspace  -name $workspaceReports -skipErrorIfExists

$fileOverrides = @{
    
    # Change the connected dataset

    "*definition.pbir" = @{
        "version" = "1.0"
        "datasetReference" = @{          
            "byConnection" =  @{
            "connectionString" = $null
            "pbiServiceModelId" = $null
            "pbiModelVirtualServerName" = "sobe_wowvirtualserver"
            "pbiModelDatabaseName" = "$($deployInfo.id)"                
            "name" = "EntityDataSource"
            "connectionType" = "pbiServiceXmlaStyleLive"
            }
        }
    } | ConvertTo-Json

    # Change logo

    "*_7abfc6c7-1a23-4b5f-bd8b-8dc472366284171093267.jpg" = "$currentPath\sample-resources\logo2.jpg"

    # Change Report Name

    "*.report\item.metadata.json" = @{
        "type" = "report"
        "displayName" = "Sales-NewName"
    } | ConvertTo-Json
}

$deployInfo = Import-FabricItems -workspaceId $workspaceId -path $pbipPath -filter "*\*.report" -fileOverrides $fileOverrides

```