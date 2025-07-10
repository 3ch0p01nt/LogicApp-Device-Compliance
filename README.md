# Logic App for Adding Extended Attributes to Non-Compliant Devices

This repository contains a Logic App that identifies non-compliant devices via Microsoft Graph and adds extended attributes (e.g., marking them as "non-compliant" in extensionAttribute1).

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
Connect-MgGraph -Scopes Application.Read.All, AppRoleAssignment.ReadWrite.All, RoleManagement.ReadWrite.Directory, Directory.Read.All -UseDeviceAuthentication

$managedIdentityId = "27dfa32f-c0ad-403b-b157-6b90784a0413"

$msgraph = Get-MgServicePrincipal -Filter "AppId eq '00000003-0000-0000-c000-000000000000'"

# Assign each role
$roleNames = "DeviceManagementManagedDevices.Read.All", "Device.ReadWrite.All", "Directory.Read.All"

foreach ($roleName in $roleNames) {
    $role = $msgraph.AppRoles | Where-Object { $_.Value -eq $roleName -and $_.AllowedMemberTypes -contains "Application" }

    if ($null -eq $role) {
        Write-Error "App role '$roleName' not found in Microsoft Graph service principal."
        continue
    }

    New-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $managedIdentityId `
                                            -PrincipalId $managedIdentityId `
                                            -ResourceId $msgraph.Id `
                                            -AppRoleId $role.Id
}
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


<img width="1185" height="1278" alt="image" src="https://github.com/user-attachments/assets/f69fe298-98b6-4b38-8686-6328e5a4ad4f" />


## Contributing

Feel free to fork and submit pull requests for improvements.

## License

MIT License - see LICENSE file for details.
