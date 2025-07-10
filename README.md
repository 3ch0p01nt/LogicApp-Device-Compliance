# Logic App for Adding Extended Attributes to Non-Compliant Devices

This repository contains a Logic App that identifies non-compliant devices via Microsoft Graph and adds extended attributes (e.g., marking them as "non-compliant" in extensionAttribute1). Note: this is setup for commercial, you will have to change the graph endpoint for other cloud environments

## Overview

The Logic App:
- Triggers on an HTTP request.
- Queries Microsoft Graph for devices where `isCompliant eq false`.
- Parses the response.
- Iterates through the devices and patches the extension attribute.

**Note:** The Logic App uses authentication parameters (tenantId, clientId, clientSecret) for portability across tenants. Ensure you have the necessary permissions set up.

## Prerequisites

- Azure subscription with Logic Apps enabled.
- Microsoft Graph permissions assigned to a service principal or managed identity.
- Required roles:
  - DeviceManagementManagedDevices.Read.All
  - Device.ReadWrite.All
  - Directory.Read.All

## Setup Permissions

You need to grant the required Microsoft Graph permissions to your managed identity or service principal.

### Via PowerShell

Connect to Microsoft Graph and assign the roles:

```powershell
#
# PowerShell Script to Assign Microsoft Graph API Permissions to a Managed Identity
#

# --- Step 1: Connect to Microsoft Graph ---
# Connect with the necessary permission scopes using device code authentication.
# The required scopes are for reading applications, assigning app roles, and reading directory information.
Connect-MgGraph -Scopes Application.Read.All, AppRoleAssignment.ReadWrite.All, RoleManagement.ReadWrite.Directory, Directory.Read.All -UseDeviceAuthentication

# --- Step 2: Get User Input for the Managed Identity ---
# Prompt the user to enter the Object ID (or Principal ID) of the Managed Identity.
$managedIdentityId = Read-Host "Enter the Managed Identity's Object ID"

# --- Step 3: Get the Microsoft Graph Service Principal ---
# This is the application (Microsoft Graph) to which we will assign permissions.
# We fetch its details, including the available application roles (AppRoles).
try {
    Write-Host "Fetching the Microsoft Graph service principal..."
    $msgraph = Get-MgServicePrincipal -Filter "AppId eq '00000003-0000-0000-c000-000000000000'" -Property Id, AppId, DisplayName, AppRoles -ErrorAction Stop

    if ($null -eq $msgraph) {
        Write-Error "Failed to retrieve Microsoft Graph Service Principal. Check permissions."
        return
    }
    Write-Host "Successfully fetched Microsoft Graph Service Principal: $($msgraph.DisplayName)"
} catch {
    Write-Error "An error occurred while fetching the Service Principal: $_"
    return
}

# --- Step 4: Define and Assign the Required Roles ---
# Define the list of permissions (role names) you want to assign.
$roleNames = "Device.ReadWrite.All", "Directory.Read.All"

# Loop through each role name and assign it to the Managed Identity.
foreach ($roleName in $roleNames) {
    # Find the specific role definition within the Microsoft Graph service principal.
    $role = $msgraph.AppRoles | Where-Object { $_.Value -eq $roleName -and $_.AllowedMemberTypes -contains "Application" }

    if ($null -eq $role) {
        Write-Warning "App role '$roleName' not found in Microsoft Graph service principal. Skipping."
        continue
    }

    # Create the app role assignment.
    try {
        New-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $managedIdentityId `
            -PrincipalId $managedIdentityId `
            -ResourceId $msgraph.Id `
            -AppRoleId $role.Id `
            -ErrorAction Stop
        Write-Host "✅ Successfully assigned role: $roleName"
    } catch {
        # Check if the role is already assigned, which is a common, non-fatal error.
        if ($_.Exception.Message -like "*Permission being assigned already exists*") {
            Write-Host "⚠️ Role '$roleName' was already assigned."
        } else {
            Write-Error "Failed to assign role '$roleName': $_"
        }
    }
}

Write-Host "`nAll specified roles have been processed."
```

### Via EXE Tool

Use the Managed Identity Permission Manager tool:

[Download and instructions here](https://github.com/michaelmsonne/ManagedIdentityPermissionManager?tab=readme-ov-file)

## Deployment

1. Copy the Logic App JSON from the provided file (or paste it into Azure Logic Apps designer).
2. Deploy the Logic App in your Azure portal.
3. Configure parameters: tenantId, clientId, clientSecret.
4. Test by sending an HTTP request to the trigger URL.

## Usage

- Trigger the Logic App via HTTP POST to the generated URL.
- It will automatically fetch non-compliant devices and update their extension attributes.

## Logic App JSON

For reference, the Logic App definition is included in `logicapp.json` (or paste the provided JSON here if including in README).


## Contributing

Feel free to fork and submit pull requests for improvements.

## License

MIT License - see LICENSE file for details.



<img width="1185" height="1278" alt="image" src="https://github.com/user-attachments/assets/2ca15835-519b-4ee0-a6cf-61247ba74522" />

<img width="1917" height="1410" alt="image" src="https://github.com/user-attachments/assets/fe069bf4-f1fd-4b1b-b0f5-b3e4fec4abf3" />




