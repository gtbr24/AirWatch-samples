# AirWatch GPO Migration Tool

## Overview
- **Author**: Justin Sheets
- **Email**: jsheets@vmware.com
- **Date Updated**: 02/16/2018
- **Tested on AirWatch 9.3.0.0**: Completed

## SYNOPSIS
This Powershell script allows you to capture and upload both new or existing GPO backups to AirWatch to easily deploy and apply policies to your managed devices.

**Requirements**

- Must Download and include the [Microsoft Security Compliance Toolkit](https://www.microsoft.com/en-us/download/details.aspx?id=55319 "Microsoft Security Compliance Toolkit") in the root folder of the project
- Must have access to an AirWatch Admin Account that can authenticate to the APIs with Basic (Certificates currently not supported)
- Full API automation support only available on AirWatch 9.2.4.0 and newer 
        
## DESCRIPTION
When run, this script will prompt you to view, capture, or upload GPO backups to AirWatch.

**Viewing & Capturing GPOs:**

GPO backups are captured and stored within the GPO Backups folder in the project files.  In addition to capturing GPO backups, you can
also copy or move existing GPO backups to this GPO Backups folder to easily upload these to AirWatch.

**Uploading GPOs:**

When deploying packages to AirWatch, you will need AirWatch Admin credentials to authenticate against the AirWatch APIs.  This AirWatch Admin needs access to the Organization Group you are deploying the package to.  Once the packages are uploaded to AirWatch, you will need to assign them to the desired devices within the AirWatch Console.
When selecting GPOs backups to upload, you can select multiple GPOs by holding Shift or Ctrl when clicking.  GPOs will be applied on machines in the order in which they were selected.
	
## Modifications Required
You will need to supply your AirWatch Admin credentials, including the AirWatch API Key, when prompted. 

## Known Issues
The task to upload the GPO package to the AirWatch Console will fail versions of AirWatch prior to 9.2.3.X.  See the EXAMPLE section for a full walkthrough of both processes.
	
## Video Walk-through
Click the below link to see a guided walk-through on how to setup and use this tool.

[![[Deep Dive] AirWatch GPO Migration Tool for Windows 10](https://img.youtube.com/vi/_PSVNd8_af0/0.jpg)](https://www.youtube.com/watch?v=_PSVNd8_af0)

## EXAMPLE

    .\Migrate-GPO-AirWatch.ps1 `
        -awServer "https://mondecorp.ssdevrd.com" `
        -awUsername "tkent" `
        -awPassword "SecurePassword" `
        -awTenantAPIKey "iVvHQnSXpX5elicaZPaIlQ8hCe5C/kw21K3glhZ+g/g=" `
        -awGroupID "652" `
        -Verbose

**AirWatch 9.3.0.0 and Newer:**

1. Run the Migrate-GPO-AirWach.ps1 script
2. When prompted, select option 2 to capture a local GPO backup
3. When prompted, select option 3 to upload the local GPO backup to the AirWatch Console.
4. Navigate to the AirWatch Console and find the uploaded GPO package under Apps & Books > Applications > Native.  Click Assign by the Application and add your target devices and/or users.

**AirWatch 9.2.3.X and Prior:**

1. Run the Migrate-GPO-AirWach.ps1 script
2. When prompted, select option 2 to capture a local GPO backup
3. When prompted, select option 4 to Build the GPO Package which will be manually uploaded to the AirWatch.  The process will complete and display the build folder, which will contain a .zip package and a installpath.txt. We will upload the .zip package as an Application while using the contents of the installpath.txt to configure the application.
4. Navigate to the AirWatch Console > Apps & Books > Applications > Native.  Click Add Application.
	1. For the Application File, provide the .zip from your Project Root Folder > GPO Uploads.
	2. When editing the Application, provide the following details:
		1. **Files:**
			1. Uninstall Command: LGPO.exe
		1. **Deployment Options:**
			1. Install Context: Device
			2. Install Command: powershell -executionpolicy bypass -File DeployPackage.ps1			
			3. Admin Privileges: Yes
			4. Installer Reboot Exit Code: 0
			5. Installer Success Exit Code: 0
			6. Identify Application By: Defining Criteria
			7. Click Add for Defining Criteria.
			8. Criteria Type: File Exists
			9. Open the installpath.txt file generated from the powershell script earlier and paste the contents into the Path field.  This will be "%programdata%\AirWatch\GPOs\{GUID}\success.txt"
			10. Click Add to confirm the Criteria.
        
## Parameters

**awServer**

Server URL for the AirWatch API Server

**awUsername**

The username of an AirWatch account being used in the target AirWatch server.  This user must have a role that allows API access.
  
**awPassword**

The password of the AirWatch account specified by the awUsername parameter.

**awTenantAPIKey**

This is the REST API key that is generated in the AirWatch Console.  You locate this key at All Settings -> Advanced -> API -> REST, and you will find the key in the API Key field.  If it is not there you may need override the settings and Enable API Access, which is available at Customer type Organization Groups

**awGroupID**

The groupID is the ID of the Organization Group where the apps will be migrated. The API key and admin credentials need to be authenticated at this Organization Group. 

The shorcut to getting this value is to navigate to **https://[HOST]/AirWatch/#/AirWatch/OrganizationGroup/Details**.
The ID you are redirected to appears in the URL (7 in the following example). **https://[HOST]/AirWatch/#/AirWatch/OrganizationGroup/Details/Index/7**

## Additional Information

**Logging**

When devices process the GPO package application from AirWatch, logs will be generated at **%programdata%/AirWatch/GPOs/**.