# Local Auto Device Registration and Enrollment

## Overview
- **Author**: Chase Bradley
- **Email**: cbradley@vmware.com
- **Date Created**: 8/1/2017
- **Tested on Windows 10**: 1703 Enterprise

## Purpose 
This set of scripts will run locally on a device and use AirWatch's APIs to register itself using it's serial number then perform an auto enrollment using a staging account, but auto map to the correct user in AirWatch. You can use your existing deployment tool, bake into the image, or manually run on a device. 

## Change Log
- 8/1/2017: Initial upload of Local Auto Device Registration and Enrollment


## Requirements
The MVC4 Package is required. A lightweight version of the MVC4 installer is included, however the full version (which can be baked into the image) can be downloaded from: https://www.microsoft.com/en-us/download/details.aspx?id=30683

In order for machines to register you need to ensure they have a proper serial number.  While this is never an issue on physical machines virtual machines often need updates to get working

> **For FUSION Machines**:
> 
> 1. Before you start the VM navigate to the root folder for the VM. You'll see a config file with a .vmx extension.
> 1. If you insert the following two lines in the .vmx file, it will boot with a shorter 12 Char serial number. Without this you cannot use WS1 or any feature that relies on serial number.
> 1. SMBIOS.useShortSerialNumber = "TRUE"
> 1. SMBIOS.use12CharSerialNumber = "TRUE" 


> **For VSphere Machines**:
> 
> 1. In the vSphere Web Client, navigate to the vCenter Server instance.
> 1. Select the Manage tab.
> 1. Select Advanced Settings.
> 1. Click Edit.
> 1. Add the following two lines:
> 	1. SMBIOS.useShortSerialNumber = "TRUE"
> 	1. SMBIOS.use12CharSerialNumber = "TRUE" 



**Step 1**: Copy the contents of the AirWatch folder (optionally just copy the AirWatch folder) to a location of your choice.  My preferred location for the files/folders is C:\Installs\AirWatch.

**Step 2**: Create a staging user in the AirWatch console at the top Organization Group.  Set the staging mode to: Single User, Advanced: Enroll on behalf of user.  Record the username and password of this user.

**Step 3**: (Optional): Download the latest agent (you can use the download_latest_agent1.ps1 in \setupfiles\) then copy that agent to the same folder as the localdevice.exe OR Registration.cs file.  Rename the file to AirWatchAgent.msi (you may need to replace an existing file).

**Step 4**: Create an AirWatch Administrator account API Service Account in AirWatch with Console Administrator role.  Using a [Base 64 encoder ](https://www.base64encode.org/) get the encoded string using the format:
		
			`username:password`
		
Copy the encoded string to be used later in the INI file.
		
**Step 5**: Ensure that you have a Rest API key generated in the AirWatch Console.  **Settings -> General -> Advanced -> API -> REST API**
		
**Step 6**: Modify the localdevice.ini file to reflect the correct settings.  ; represent comments in ini files.

	#************************************#
	# 		 INI SAMPLE FILE			 #
	#************************************#
	[Config]
	Authorization=Basic %BASE_64_ENCODED_API_CREDENTIALS%
	API_Key=%API_KEY%
	API_Server=https://%API_SERVER_URL%/api
	Enrollment_Server=%ENROLLMENT_SERVER_URL%
	;LocationGroupID is Optional - can search by group id
	LocationGroupID=%LOCATIONGROUP_ID% 
	GroupID=%GROUP_ID%
	AdminEmailAddress=%ADMIN_EMAIL_ADDRESS%
	StagingUser=%STAGING_USERNAME%
	StagingPassword=%STAGING_PASSWORD%

	[SMTP]
	UseSMTP=0
	SMTPServer=%SMTPServer%
	Sender=%SMTPSender%

	[Staging]
	AllowedStagingUsers=%UserAccount% 
	;Deliniate multiple accounts using commas.  Use a period to represent local machines
	;Azure Users 

	[Debug]
	EnableDebug=0
	DebugUser=%DebugUserName%
	;This section is for testing only.  Delete entire section when deploying.

**Step 7**: In the imaging software you will like to use, you will need to copy the software to the install path, and either have the scheduled task built OR have an instruction to install the scheduled task.  The recommended approach is the \setupfiles\install_task_psonly.ps1

**Step 8**: On user login the device will:
1. Register with AirWatch
1. Enroll
		
