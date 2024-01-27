# IntuneMigration
With the assumptions in place, let’s break down how the process works.  Like most cool things with Intune, we are accomplishing the migration with a series of scripts.  Here is the high-level overview of how we’re performing the migration: 

1. Migration app package (.intunewin) is made available to users via the Intune Company Portal.
2. User installs the migration app
3. Application unpacks series of scripts to the local PC and begins running the primary PowerShell script which performs the following:
    * Authenticates to Tenant A using Microsoft Graph API with app registration
    * Collects device information from Tenant A
    * Copy %APPDATA% and %LOCALAPPDATA% to a temporary location
    * Remove previous MDM enrollments from registry
    * Stage post migration tasks
    * PC leaves Azure AD of Tenant A
    * Intune object deleted from Tenant A
    * Autopilot object deleted from Tenant A
4. User is prompted that they will be signed out in 1 minute.
5. Device reboots, and then subsequently reboots again in 30 seconds
6. User signs in with Tenant B credentials
7. PC is Azure AD joined and Intune enrolled to Tenant B
8. Task runs to restore %APPDATA% and %LOCALAPPDATA% items new profile
9. Authenticate to Tenant B using Microsoft Graph API with app registration
10. Task runs to set the primary user of the device within Intune
11. Task runs to migrate the BitLocker Key to Tenant B Azure environment
12. Task runs to register device in Tenant B Autopilot with Group Tag attribute from Tenant A

## What is needed?
So what do you need to do this?  Well here are individual pieces we’re going to be covering in the next few posts:

1. **App Registrations**: App registrations are created in both Tenant A and Tenant B.  This will be the primary means of authenticating objects throughout the various scripts used in the process.
2. **Scripts**: There are several scripts that are used during the process which will need to be modified for organization specific values such as the app registration info, tenant IDs, Autopilot group tags, etc.
3. **Align XML Tasks to PowerShell Scripts**: The PowerShell scripts run at various context levels, but most are as SYSTEM, or NT\AUTHORITY.  To achieve this, each PowerShell script will have an accompanying XML file to set a scheduled task in Windows that will run the script with the appropriate context.  There is also quite a bit of sequencing, so the tasks will allow us to set the correct timing intervals we want each piece to run at, both pre and post migration.
4. **Windows Provisioning Package**: This is the crux of the entire process.  We use the Windows Configuration Designer  tool to create a .PPKG file, which is used to deliver an Azure AD Bulk Primary Refresh Token (BPRT).  The package is what binds the device to Tenant B after being removed from Tenant A
5. **Cleanup**: As I said, there are pre and post elements to the migration, and they need to be sequenced carefully.  Our scripts ensure that at the appropriate times, Intune and Autopilot objects in Tenant A are being deleted to allow for the enrollment and registration to Tenant B.
6. **Validate virtually**: I’ve said this before, but using virtual machines (VM) to test this process is critical.  Ensure you have several  VMs to test with and 1 or 2 accounts in Tenant A that have a counterpart identity in Tenant B.  This testing is also a great time to document the end user experience, which is critical to the whole operation.
