---
# required metadata

title: Deploy the Azure Information Protection scanner
description: Instructions to install, configure, and run the Azure Information Protection scanner to discover, classify, and protect files on data stores.
author: cabailey
ms.author: cabailey
manager: mbaldwin
ms.date: 04/09/2018
ms.topic: article
ms.prod:
ms.service: information-protection
ms.technology: techgroup-identity
ms.assetid: 20d29079-2fc2-4376-b5dc-380597f65e8a

# optional metadata

#ROBOTS:
#audience:
#ms.devlang:
ms.reviewer: demizets
ms.suite: ems
#ms.tgt_pltfrm:
#ms.custom:

---

# Deploying the Azure Information Protection scanner to automatically classify and protect files

>*Applies to: [Azure Information Protection](https://azure.microsoft.com/pricing/details/information-protection), Windows Server 2016, Windows Server 2012 R2*

Use this information to learn about the Azure Information Protection scanner, and then how to successfully install, configure, and run it. 

This scanner runs as a service on Windows Server and lets you discover, classify, and protect files on the following data stores:

- Local folders on the Windows Server computer that runs the scanner.

- UNC paths for network shares that use the Server Message Block (SMB) protocol.

- Sites and libraries for SharePoint Server 2016 and SharePoint Server 2013.

## Overview of the Azure Information Protection scanner

When you have configured your [Azure Information Protection policy](configure-policy.md) for labels that apply automatic classification, files that this scanner discovers can then be labeled. Labels apply classification, and optionally, apply protection or remove protection:

![Azure Information Protection scanner architecture overview](../media/infoprotect-scanner.png)

The scanner can inspect any files that Windows can index, by using iFilters that are installed on the computer. Then, to determine if the files need labeling, the scanner uses the Office 365 built-in data loss prevention (DLP) sensitivity information types and pattern detection, or Office 365 regex patterns. Because the scanner uses the Azure Information Protection client, it can classify and protect the same [file types](../rms-client/client-admin-guide-file-types.md).

You can run the scanner in discovery mode only, where you use the reports to check what would happen if the files were labeled. Or, you can run the scanner to automatically apply the labels.

Note that the scanner does not discover and label in real time. It systematically crawls through files on data stores that you specify, and you can configure this cycle to run once, or repeatedly.

## Prerequisites for the Azure Information Protection scanner
Before you install the Azure Information Protection scanner, make sure that the following requirements are in place.

|Requirement|More information|
|---------------|--------------------|
|Windows Server computer to run the scanner service:<br /><br />- 4 processors<br /><br />- 4 GB of RAM|Windows Server 2016 or Windows Server 2012 R2. <br /><br />Note: For testing or evaluation purposes in a non-production environment, you can use a Windows client operating system that is [supported by the Azure Information Protection client](../get-started/requirements.md#client-devices).<br /><br />This computer can be a physical or virtual computer that has a fast and reliable network connection to the data stores to be scanned. <br /><br />Make sure that this computer has the [Internet connectivity](../get-started/requirements.md#firewalls-and-network-infrastructure) that it needs for Azure Information Protection. Or, you must configure it as a [disconnected computer](../rms-client/client-admin-guide-customizations.md#support-for-disconnected-computers). |
|SQL Server to store the scanner configuration:<br /><br />- Local or remote instance<br /><br />- Sysadmin role to install the scanner|SQL Server 2012 is the minimum version for the following editions:<br /><br />- SQL Server Enterprise<br /><br />- SQL Server Standard<br /><br />- SQL Server Express<br /><br />The account that installs the scanner requires permissions to write to the master database (must be a member of the db_datawriter role). The installation process grants the db-owner role to the service account that runs the scanner. Alternately, you can manually create the AzInfoProtectionScanner database before you install the scanner, and assign the db-owner role to the scanner service account.|
|Service account to run the scanner service|This account must be an Active Directory account that is synchronized to Azure AD, with the following additional requirements:<br /><br />- **Log on locally** right. This right is required for the installation and configuration of the scanner, but not for operation. You must grant this right to the service account but you can remove this right after you have confirmed that the scanner can discover, classify, and protect files. <br /><br />Note: If internal policies do not allow service accounts to have this right, but service accounts can be granted the **Log on as a batch job** right, you can meet this requirement with additional configuration. For instructions, see [Specify and use the Token parameter for Set-AIPAuthentication](../rms-client/client-admin-guide-powershell.md#specify-and-use-the-token-parameter-for-set-aipauthentication) from the admin guide.<br /><br />- **Log on as a service** right. This right is automatically granted to the service account during the scanner installation and this right is required for the installation, configuration, and operation of the scanner. <br /><br />- Permissions to the data repositories: You must grant **Read** and **Write** permissions for scanning the files and then applying classification and protection to the files that meet the conditions in the Azure Information Protection policy. To run the scanner in discovery mode only, **Read** permission is sufficient.<br /><br />- For labels that reprotect or remove protection: To ensure that the scanner always has access to protected files, make this account a [super user](configure-super-users.md) for the Azure Rights Management service, and ensure that the super user feature is enabled. For more information about the account requirements for applying protection, see [Preparing users and groups for Azure Information Protection](../plan-design/prepare.md).|
|The Azure Information Protection scanner is installed on the Windows Server computer|Currently, the Azure Information Protection scanner is a separate download named **AzInfoProtectionScanner.exe** on the [Microsoft Download Center](https://www.microsoft.com/en-us/download/details.aspx?id=53018). Subsequent releases of the scanner will be included in the Azure Information Protection client.|
|Configured labels that apply automatic classification, and optionally, protection|For more information about how to configure the conditions, see [How to configure conditions for automatic and recommended classification for Azure Information Protection](configure-policy-classification.md).<br /><br />For more information about how to configure labels to apply protection to files, see [How to configure a label for Rights Management protection](configure-policy-protection.md).<br /><br />These labels can be in the global policy, or one or more [scoped policies](configure-policy-scope.md).|
|If all files in one or more data repositories must have a label:<br /><br />- A default label configured as a policy setting|For more information about how to configure the default label setting, see [How to configure the policy settings for Azure Information Protection](configure-policy-settings.md).<br /><br />This default label setting must be in the global policy or a scoped policy for the scanner. However, this default label setting can be overridden by a different default label that you configure at the data repository level.|

## Install the Azure Information Protection scanner

1. Sign in to the Windows Server computer that will run the scanner. Use an account that has local administrator rights and that has permissions to write to the SQL Server master database.

2. Open a Windows PowerShell session with the **Run as an administrator** option.

3. Run the [Install-AIPScanner](/powershell/module/azureinformationprotection/Install-AIPScanner) cmdlet, specifying your SQL Server instance on which to create a database for the Azure Information Protection scanner: 
    
    ```
    Install-AIPScanner -SqlServerInstance <database name>
    ```
    
    Examples:
    
    For a default instance: `Install-AIPScanner -SqlServerInstance SQLSERVER1`
    
    For a named instance: `Install-AIPScanner -SqlServerInstance SQLSERVER1\AIPSCANNER`
    
    For SQL Server Express: `Install-AIPScanner -SqlServerInstance SQLSERVER1\SQLEXPRESS`
    
    Use the online help for this cmdlet if you need more [detailed examples](/powershell/module/azureinformationprotection/install-aipscanner#examples).
    
    When you are prompted, provide the credentials for the scanner service account (\<domain\user name>) and password.

4. Verify that the service is now installed by using **Administrative Tools** > **Services**. 
    
    The installed service is named **Azure Information Protection Scanner** and is configured to run by using the scanner service account that you created.

Now that you have installed the scanner, you need to get an Azure AD token for the scanner service account to authenticate so that it can run unattended. 

## Get an Azure AD token for the scanner service account to authenticate to the Azure Information Protection service

1. From the same Windows Server computer, or from your desktop, sign in to the Azure portal to create two Azure AD applications that are needed to specify an access token for authentication. After an initial interactive sign-in, this token lets the scanner run non-interactively.
    
    To create these applications, follow the instructions in [How to label files non-interactively for Azure Information Protection](../rms-client/client-admin-guide-powershell.md#how-to-label-files-non-interactively-for-azure-information-protection) from the admin guide.

2. From the Windows Server computer, if your scanner service account has been granted the **Log on locally** right for the installation: Sign in with this account and start a PowerShell session. Run [Set-AIPAuthentication](/powershell/module/azureinformationprotection/set-aipauthentication), specifying the values that you copied from the previous step:
    
    ```
    Set-AIPAuthentication -webAppId <ID of the "Web app / API" application>  -webAppKey <key value generated in the "Web app / API" application> -nativeAppId <ID of the "Native" application >
    ```
    
    When prompted, specify the password for your service account credentials for Azure AD, and then click **Accept**.
    
    If your scanner service account cannot be granted the **Log on locally** right for the installation: Follow the instructions in the [Specify and use the Token parameter for Set-AIPAuthentication](../rms-client/client-admin-guide-powershell.md#specify-and-use-the-token-parameter-for-set-aipauthentication) section from the admin guide. 

The scanner now has a token to authenticate to Azure AD, which is valid for one year, two years, or never expires, according to your configuration of the **Web app /API** in Azure AD. When the token expires, you must repeat steps 1 and 2.

You're now ready to specify the data stores to scan. 

## Specify data stores for the Azure Information Protection scanner

Use the [Add-AIPScannerRepository](/powershell/module/azureinformationprotection/Add-AIPScannerRepository) cmdlet to specify the data stores to be scanned by the Azure Information Protection scanner. You can specify local folders, UNC paths, and SharePoint Server URLs for SharePoint sites and libraries. 

Supported versions for SharePoint: SharePoint Server 2016 and SharePoint Server 2013.

1. From the same Windows Server computer, in your PowerShell session, add your first data store by running the following command:
    
    	Add-AIPScannerRepository -Path <path>
    
    For example: `Add-AIPScannerRepository -Path \\NAS\Documents`
    
    For other examples, see the [online help](/powershell/module/azureinformationprotection/Add-AIPScannerRepository#examples) for this cmdlet.

2. Repeat this command for all the data stores that you want to scan. If you need to remove a data store that you added, use the [Remove-AIPScannerRepository](/powershell/module/azureinformationprotection/Remove-AIPScannerRepository) cmdlet.

3. Confirm that you have specified all the data stores correctly, by running the [Get-AIPScannerRepository](/powershell/module/azureinformationprotection/Get-AIPScannerRepository) cmdlet:
    
    	Get-AIPScannerRepository

With the scanner's default configuration, you're now ready to run your first scan in discovery mode.

## Run a discovery cycle and view reports for the Azure Information Protection scanner

1. Using **Administrative Tools** > **Services**, start the **Azure Information Protection Scanner** service.

2. Wait for the scanner to complete its cycle. When the scanner has crawled through all the files in the data stores that you specified, the service stops. You can use the local Windows **Applications and Services** event log, **Azure Information Protection**, to confirm when the service is stopped. Look for the informational event ID **911**.

3. Review the reports that are stored in %*localappdata*%\Microsoft\MSIP\Scanner\Reports and that have a .csv file format. With the default configuration of the scanner, only files that meet the conditions for automatic classification are included in these reports.
    
    If the results are not as you expect, you might need to fine-tune the conditions that you specified in your Azure Information Protection policy. If that's the case, repeat steps 1 through 3 until you are ready to change the configuration to apply the classification and optionally, protection. Each time you repeat these steps, first run the following PowerShell command on the Windows Server computer:
    
    	Set-AIPScannerConfiguration -Schedule OneTime

When you're ready to automatically label the files that the scanner discovers, continue to the next procedure. 

## Configure the Azure Information Protection scanner to apply classification and protection to discovered files

In its default setting, the scanner runs one time and in the reporting-only mode. To change these settings, run the [Set-AIPScannerConfiguration](/powershell/module/azureinformationprotection/Set-AIPScannerConfiguration) cmdlet.

1. On the Windows Server computer, in the PowerShell session, run the following command:
    
    	Set-AIPScannerConfiguration -ScanMode Enforce -Schedule Continuous
    
    There are other configuration settings that you might want to change. For example, whether file attributes are changed and what is logged in the reports. In addition, if your Azure Information Protection policy includes the setting that requires a justification message to lower the classification level or remove protection, specify that message by using this cmdlet. Use the [online help](/powershell/module/azureinformationprotection/Set-AIPScannerConfiguration#parameters) for more information about each configuration setting. 

2. Using **Administrative Tools** > **Services**, restart the **Azure Information Protection Scanner** service.

3. As before, monitor the event log and the reports to see which files were labeled, what classification was applied, and whether protection was applied.

Because we configured the schedule to run continuously, when the scanner has worked its way through all the files, it starts a new cycle so that new and changed files are discovered.


## How files are scanned by the Azure Information Protection scanner

The scanner automatically skips files that are [excluded from classification and protection](../rms-client/client-admin-guide-file-types.md#file-types-that-are-excluded-from-classification-and-protection-by-the-azure-information-protection-client), such as executables and system files.

Then the scanner uses Windows iFilter to scan the following file types. For these file types, the document will be labeled by using the conditions that you specified for your labels.

|Application type|File type|
|--------------------------------|-------------------------------------|
|Word|.docx; .docm; .dotm; .dotx|
|Excel|.xls; .xlt; .xlsx; .xltx; .xltm; .xlsm; .xlsb|
|PowerPoint|.ppt; .pps; .pot; .pptx; .ppsx; .pptm; .ppsm; .potx; .potm|
|Project|.mpp; .mpt|
|PDF|.pdf|
|Text|.txt; .xml; .csv|


Finally, for the remaining file types, the scanner applies the default label in the Azure Information Protection policy.

|Application type|File type|
|--------------------------------|-------------------------------------|
|Project|.mpp; .mpt|
|Publisher|.pub|
|Visio|.vsd; .vdw; .vst; .vss; .vsdx; .vsdm; .vssx; .vssm; .vstx; .vstm|
|XPS|.xps; .oxps; .dwfx|
|Solidworks|.sldprt; .slddrw; .sldasm|
|Jpeg |.jpg; .jpeg; .jpe; .jif; .jfif; .jfi|
|Png |.png|
|Gif|.gif|
|Bitmap|.bmp; .giff|
|Tiff|.tif; .tiff|
|Photoshop|.psdv|
|DigitalNegative|.dng|
|Pfile|.pfile|

Note that when a label applies generic protection to documents, the file name extension changes to .pfile. In addition, the file becomes read-only until it is opened by an authorized user and saved in its native format. Text and images files can also change their file name extension and become read-only. If you do not want this behavior, you can prevent files that have a specific file type from being protected. For example, prevent a PDF file from becoming a protected PDF (.ppdf) file, and prevent a .txt file from becoming a protected text (.ptxt) file.

For more information about the different levels of protection for different file types, and how to control the protection behavior, see the [File types supported for protection](../rms-client/client-admin-guide-file-types.md#file-types-supported-for-protection) section in the admin guide.


## When files are rescanned by the Azure Information Protection scanner

For the first scan cycle, the scanner inspects all files in the configured data stores and then for subsequent scans, only new or modified files are inspected. 

You can force the scanner to inspect all files again by running [Set-AIPScannerConfiguration](/powershell/module/azureinformationprotection/Set-AIPScannerConfiguration) with the `-Type` parameter set to **Full**. This configuration is useful when you want the reports to include all files and it is typically used when the scanner runs in discovery mode. When a full scan is complete, the scan type automatically changes to incremental so that for subsequent scans, only new or modified files are scanned.

In addition, all files are inspected when the scanner downloads an Azure Information Protection policy that has new or changed conditions. The scanner refreshes the policy every hour, and when the service starts and the policy is older than one hour.  

> [!TIP]
> If you need to refresh the policy sooner than this one hour interval, for example, during a testing period: Manually delete the policy file, **Policy.msip** from both **%LocalAppData%\Microsoft\MSIP\Policy.msip** and **%LocalAppData%\Microsoft\MSIP\Scanner**. Then restart the Azure Information Scanner service.
> 
> If you changed protection settings in the policy, also wait 15 minutes from when you saved the protection settings before you restart the service.

If the scanner downloaded a policy that had no automatic conditions configured, the copy of the policy file in the scanner folder does not update. In this scenario, you must delete the policy file, **Policy.msip** from both **%LocalAppData%\Microsoft\MSIP\Policy.msip** and **%LocalAppData%\Microsoft\MSIP\Scanner** before the scanner can use a newly downloaded policy file that has labels correctly figured for automatic conditions.

## Optimizing the performance of the Azure Information Protection scanner

To maximize the scanner performance:

- **Have a high speed and reliable network connection between the scanner computer and the scanned data store**
    
    For example, place the scanner computer in the same LAN, or (preferred) in the same network segment as the scanned data store.
    
    The quality of the network connection affects the scanner performance because to inspect the files, the scanner transfers the contents of the files to the computer running the scanner service. When you reduce (or eliminate) the number of network hops this data has to travel, you also reduce the load on your network. 

- **Make sure the scanner computer has available processor resources**
    
    Inspecting the file contents for a match against your configured conditions, and encrypting and decrypting files are processor-intensive actions. Monitor typical scanning cycles for your specified data stores to identify whether a lack of processor resources is negatively affecting the scanner performance.
    
- **Do not scan local folders on the computer running the scanner service**
    
    If you have folders to scan on a Windows server, install the scanner on a different computer and configure those folders as network shares to scan. Separating the two functions of hosting files and scanning files means that the computing resources for these services are not competing with one another.

Other factors that affect the scanner performance:

- The current load and response times of the data stores that contain the files to scan

- Whether the scanner runs in discovery mode or enforce mode
    
    Discovery mode typically has a higher scanning rate than enforce mode because discovery requires a single file read action, whereas enforce mode requires read and write actions.

- You change the conditions in the Azure Information Protection
    
    Your first scan cycle when the scanner must inspect every file will obviously take longer than subsequent scan cycles that by default, inspect only new and changed files. However, if you change the conditions in the Azure Information Protection policy, all files are scanned again, as described in the [preceding section](#when-files-are-rescanned-by-the-azure-information-protection-scanner).

- Your chosen logging level
    
    You can choose between **Debug**, **Info**, **Error** and **Off** for the scanner reports. **Off** results in the best performance; **Debug** considerably slows down the scanner and should be used only for troubleshooting. For more information, see the *ReportLevel* parameter for the [Set-AIPScannerConfiguration](/powershell/module/azureinformationprotection/Set-AIPScannerConfiguration) cmdlet.

- The files themselves:
    
    - Office files are more quickly scanned than PDF files.
    
    - Unprotected files are quicker to scan than protected files.
    
    - Large files obviously take longer to scan than small files.

## List of cmdlets for the Azure Information Protection scanner 

Other cmdlets for the scanner let you change the service account and database for the scanner, get the current settings for the scanner, and uninstall the scanner service. The scanner uses the following cmdlets:

- [Add-AIPScannerRepository](/powershell/module/azureinformationprotection/Add-AIPScannerRepository)

- [Get-AIPScannerConfiguration](/powershell/module/azureinformationprotection/Get-AIPScannerConfiguration)

- [Get-AIPScannerRepository](/powershell/module/azureinformationprotection/Get-AIPScannerRepository)

- [Install-AIPScanner](/powershell/module/azureinformationprotection/Install-AIPScanner)

- [Remove-AIPScannerRepository](/powershell/module/azureinformationprotection/Remove-AIPScannerRepository)

- [Set-AIPScanner](/powershell/module/azureinformationprotection/Set-AIPScanner)

- [Set-AIPScannerConfiguration](/powershell/module/azureinformationprotection/Set-AIPScannerConfiguration)

- [Set-AIPScannerRepository](/powershell/module/azureinformationprotection/Set-AIPScannerRepository)

- [Uninstall-AIPScanner](/powershell/module/azureinformationprotection/Uninstall-AIPScanner)


## Event log IDs and descriptions

Use the following sections to identify the possible event IDs and descriptions for the scanner. These events are logged on the server that runs the scanner service, in the Windows **Applications and Services** event log, **Azure Information Protection**.

-----

Information **910**

**Scanner cycle started.**

This event is logged when the scanner service is started and begins to scan for files in the data repositories that you specified.

----

Information **911**

**Scanner cycle finished.**

This event is logged when the scanner has finished its one-time scan since the server started, or the scanner has finished a cycle for a continuous schedule.

----

Information **913**

**Scanner is stopped because scanner is set to Never.**

This event is logged when the scanner is configured to run one time rather than continuously, and the Azure Information Protection scanner service has been manually restarted since the computer started.  

To scan the files again, you must set the schedule to **OneTime** or **Continuous**, and then manually restart the service. To change the schedule, use the [Set-AIPScannerConfiguration](/powershell/module/azureinformationprotection/Set-AIPScannerConfiguration) cmdlet and the **Schedule** parameter.

----

Error **912**

**Unknown error has occurred.**

More information is logged in the detailed report that is stored in %localappdata%\Microsoft\MSIP\Scanner\Reports\DetailedReport_YYYY_MM_DD_HH_MM.csv.

Contact [Microsoft Support](../get-started/information-support.md#to-contact-microsoft-support) if this event continues to be logged. 

----

Error **914**

**Service was automatically stopped due to bad configuration: policy file is missing or corrupted.**

This event is logged when the Azure Information Protection client does not have a valid policy file for the scanner to run.

The Azure Information Protection policy is stored in %localappdata%\Microsoft\MSIP and it must be configured with labels that have conditions to apply automatic classification. Or, the policy must be configured for a default label.

Make sure that firewalls are not blocking the required connectivity to the Internet. For more information, see the [Firewalls and network infrastructure](../get-started/requirements.md#firewalls-and-network-infrastructure) requirements for Azure Information Protection. If Internet connectivity is not possible, follow the instructions for supporting [disconnected computers](../rms-client/client-admin-guide-customizations.md#support-for-disconnected-computers).


## Next steps

You might be wondering: [What’s the difference between Windows Server FCI and the Azure Information Protection scanner?](../get-started/faqs.md#whats-the-difference-between-windows-server-fci-and-the-azure-information-protection-scanner)

You can also use PowerShell to interactively classify and protect files from your desktop computer. For more information about this and other scenarios that use PowerShell, see [Using PowerShell with the Azure Information Protection client](../rms-client/client-admin-guide-powershell.md).


[!INCLUDE[Commenting house rules](../includes/houserules.md)]
