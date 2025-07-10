Via PowerShell
Connect-MgGraph -Scopes Application.Read.All, AppRoleAssignment.ReadWrite.All, RoleManagement.ReadWrite.Directory, Directory.Read.All -UseDeviceAuthentication

$managedIdentityId = "27dfa32f-c0ad-403b-b157-6b90784a0413"

$msgraph = Get-MgServicePrincipal -Filter "AppId eq '00000003-0000-0000-c000-000000000000'"
##do it for each role name
$roleName = "DeviceManagementManagedDevices.Read.All", "Device.ReadWrite.All", "Directory.Read.All"
$role = $msgraph.AppRoles | Where-Object { $_.Value -eq $roleName -and $_.AllowedMemberTypes -contains "Application" }

if ($null -eq $role) {
    Write-Error "App role '$roleName' not found in Microsoft Graph service principal."
    return
}

New-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $managedIdentityId `
                                        -PrincipalId $managedIdentityId `
                                        -ResourceId $msgraph.Id `
                                        -AppRoleId $role.Id

Via EXE
https://github.com/michaelmsonne/ManagedIdentityPermissionManager?tab=readme-ov-file

Roles Needed:
DeviceManagementManagedDevices.Read.All
Device.ReadWrite.All
Directory.Read.All
