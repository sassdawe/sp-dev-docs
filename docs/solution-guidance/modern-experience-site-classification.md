---
title: SharePoint "modern" sites classification
description: Configure out-of-the-box site classification for modern SharePoint sites.
ms.date: 08/05/2020
localization_priority: Priority
---

# SharePoint "modern" sites classification

> [!NOTE]
> You can now use sensitivity labels (/microsoft-365/compliance/sensitivity-labels-teams-groups-sites) instead of sites classification to help protect your SharePoint sites.

When you create "modern" sites in SharePoint Online, you can optionally select a site classification to define the sensitivity of your site data. The goal of site classification is to allow managing clusters of sites based on their classification from a governance and compliance perspective, as well as to automate governance processes based on site classification.

> [!IMPORTANT]
> The site classification option is not enabled by default, and you need to configure it at the Azure AD level.

## Enable site classification in your tenant

To benefit from site classification, you need to enable this capability at the Azure AD level, in your target tenant. After you have enabled this capability, you see an additional field **How sensitive is your data?** while creating new "modern" sites. In the following figure, you can see what the site classification field looks like.

![The site classification option while creating a "modern" site in SharePoint Online](media/modern-experiences/site-classification-ui.png)

<br/>

While in the following figure, you can see the classification highlighted in the header of a "modern" site.

![The site classification highlighted in the header of a "modern" site](media/modern-experiences/site-classification-ui-output.png)

### Enable site classification with PowerShell

To enable the site classification capability, you can execute PowerShell code like the following sample.

```powershell
# Install the Azure AD Preview Module for PowerShell
Install-Module AzureADPreview

# Connect to Azure AD
Connect-AzureAD

# Create new directory setting and set initial values
$template = Get-AzureADDirectorySettingTemplate | where { $_.DisplayName -eq "Group.Unified" }

$setting = $template.CreateDirectorySetting()
$setting["UsageGuidelinesUrl"] = "https://aka.ms/sppnp"
$setting["ClassificationList"] = "HBI, MBI, LBI, GDPR"
$setting["DefaultClassification"] = "MBI"
New-AzureADDirectorySetting -DirectorySetting $setting

```

> [!NOTE]
> Consider that it takes a while (about 1 hour or more) to have the settings available in the UI of Office 365.

About the previous code snippet, it worth saying that at the time of this writing you have to use the preview version of the Azure AD cmdlets, but soon you will be able to use the RTM version. After you have installed the Azure AD cmdlets, you simply need to connect to the target Azure AD tenant, which is backing your target SharePoint Online tenant.

Using the **Get-AzureADDirectorySettingTemplate** cmdlet, you get a reference to the **Setting Template** for the **Unified Groups**, which is the one with **DisplayName** of **Group.Unified**.

After you have the template, you can configure the settings of that template by creating a new **DirectorySetting** and providing the setting values through a dictionary.

The main settings, from a site classification perspective are:
* **UsageGuidelinesUrl**: the URL of a page where you can explain the different classification options. A link to that page shows up in the site creation form and in the header of every classified site.
* **ClassificationList**: a comma separated list of values for the site classification options list.
* **DefaultClassification**: the default value for the site classification.

### Enable site classification with PnP Core library

Another option that you have to enable the site classification capability is to leverage the PnP Core library, which provides a few extension methods to manage classification in C# code.

For example, to enable the site classification, you can write C# code like the following.

```csharp
// Connect to the admin central of your tenant
using (var adminContext = new ClientContext("https://[tenant]-admin.sharepoint.com/"))
{
    // Provide a valid set of credentials
    adminContext.Credentials = OfficeDevPnP.Core.Utilities.CredentialManager.GetSharePointOnlineCredential("[name-of-your-credentials]");

    // Create a new instance of the Tenant class of CSOM
    var tenant = new Tenant(adminContext);

    // Get an Azure AD Access Token using ADAL, MSAL, or whatever else you like
    var accessToken = getAzureADAccessToken();

    // Define the list of classifications
    var newClassificationList = new List<String>();
    newClassificationList.Add("HBI");
    newClassificationList.Add("MBI");
    newClassificationList.Add("LBI");
    newClassificationList.Add("GDPR");

    // Use the PnP extension method to enable site classification
    // Including a default classification value and the URL to an informative page
    tenant.EnableSiteClassifications(accessToken, newClassificationList, "MBI", "https://aka.ms/OfficeDevPnP");
}

```

## Update or remove site classification in your tenant

Similar to enabling the site classification, updating or removing the setting can be done either by using PowerShell or the PnP Core library.

### Update or remove site classification with PowerShell

If you need to update the site classification settings afterwards, you can use the following PowerShell snippet.

```powershell
# Connect to Azure AD
Connect-AzureAD

# Read current settings
(Get-AzureADDirectorySetting | where { $_.DisplayName -eq "Group.Unified" }).Values

# Update settings
$currentSettings = Get-AzureADDirectorySetting | where { $_.DisplayName -eq "Group.Unified" }
$currentSettings["UsageGuidelinesUrl"] = "https://aka.ms/sppnp"
$currentSettings["ClassificationList"] = "HBI, MBI, LBI, GDPR"
$currentSettings["DefaultClassification"] = "MBI"
Set-AzureADDirectorySetting -Id $currentSettings.Id -DirectorySetting $currentSettings
```

> [!NOTE]
> It may take an hour or more to have the settings updated in the UI of Office 365.

As you can see, you simply need to search for the Directory Setting with a **DisplayName** value of **Group.Unified**, and then you can update its setting values and apply the changes by using the **Set-AzureADDirectorySetting** cmdlet.

To remove a site classification, you can use the **Remove-AzureADDirectorySetting** cmdlet, like in the following code snippet.

```powershell
# Delete settings
$currentSettings = Get-AzureADDirectorySetting | where { $_.DisplayName -eq "Group.Unified" }
Remove-AzureADDirectorySetting -Id $currentSettings.Id
```

### Update or remove site classification with PnP Core library

Another option that you have is to use the PnP Core library.

For example, to update the site classification, you can write C# code like the following.

```csharp
// Connect to the admin central of your tenant
using (var adminContext = new ClientContext("https://[tenant]-admin.sharepoint.com/"))
{
    // Provide a valid set of credentials
    adminContext.Credentials = OfficeDevPnP.Core.Utilities.CredentialManager.GetSharePointOnlineCredential("[name-of-your-credentials]");

    // Create a new instance of the Tenant class of CSOM
    var tenant = new Tenant(adminContext);

    // Get an Azure AD Access Token using ADAL, MSAL, or whatever else you like
    var accessToken = getAzureADAccessToken();

    // Retrieve the current set of values for site classification
    var classificationList = tenant.GetSiteClassificationList(accessToken);
    
    // And update it by adding a new value
    var updatedClassificationList = new List<String>(classificationList);
    updatedClassificationList.Add("TopSecret");

    // Update the site classification settings accordingly
    tenant.UpdateSiteClassificationSettings(accessToken, updatedClassificationList, "MBI", "https://aka.ms/SharePointPnP");
}
```

<br/>

Moreover, to disable and remove the site classification settings, you can use a code snippet like the following.

```csharp
// Connect to the admin central of your tenant
using (var adminContext = new ClientContext("https://[tenant]-admin.sharepoint.com/"))
{
    // Provide a valid set of credentials
    adminContext.Credentials = OfficeDevPnP.Core.Utilities.CredentialManager.GetSharePointOnlineCredential("[name-of-your-credentials]");

    // Create a new instance of the Tenant class of CSOM
    var tenant = new Tenant(adminContext);

    // Get an Azure AD Access Token using ADAL, MSAL, or whatever else you like
    var accessToken = getAzureADAccessToken();

    // Disable the site classification settings
    tenant.DisableSiteClassifications(accessToken);
}
```

## Manage the classification of a site

The value of classification for a site can be read or updated later on by using the UI of SharePoint Online, as you can see in the following figure, by editing the **Site Information** settings.

![The site classification option while editing the Site Information settings of a "modern" site in SharePoint Online](media/modern-experiences/site-classification-update-ui.png)

### Programmatically read the classification of a site

From a developer's perspective, you can use CSOM and the REST API of SharePoint Online to read and write the value of classification for a specific site. In fact, every SharePoint Online site collection has the **Classification** property that you can use to read the site classification. 

Here you can see a PowerShell snippet to do that.

```powershell
# Delete settings
Connect-PnPOnline "https://[tenant].sharepoint.com/sites/[modernsite]" -Credentials [credentials]

$site = Get-PnPSite
$classificationValue = Get-PnPProperty -ClientObject $site -Property Classification
Write-Host $classificationValue
```

If you like to read the site classification value using REST, for example within a SharePoint Framework client-side web part, you can use the following REST endpoint:

```TEXT
https://[tenant].sharepoint.com/sites/[modernsite]/_api/site/Classification
```

Based on the classification value of a site, you can define automation and custom policy rules.

In the PnP Core library, there is an extension method for the **Site** object of CSOM that allows you to easily read the classification value of a site. In the following code snippet you can see how to leverage this extension method.

```csharp
// Connect to the target site collectiion
using (var clientContext = new ClientContext("https://[tenant].sharepoint.com/sites/[modernsite]"))
{
    // Provide a valid set of credentials
    clientContext.Credentials = OfficeDevPnP.Core.Utilities.CredentialManager.GetSharePointOnlineCredential("[name-of-your-credentials]");

    // Read the current classification value
    var currentClassification = clientContext.Site.GetSiteClassification();
}
```

### Programmatically update the classification of a site

If your target is a  "modern" communication site, you can use the **Classification** property of CSOM to update the value, too.

If your target is a "modern" team site and you want to update the classification value, you should use the Microsoft Graph because the **Classification** property of CSOM simply replicates the value of the **classification** property of the Microsoft 365 group.

> [!NOTE]
> You can find further details about how to update a Microsoft 365 group using the Microsoft Graph in the document [Update group](https://developer.microsoft.com/graph/docs/api-reference/v1.0/api/group_update).

To make it easier for you to update the classification of a site, in the PnP Core library there is an extension method that applies the right behavior for you, depending on the "modern" site type. In the following code excerpt you can see how to use it.

```csharp
// Connect to the target site collectiion
using (var clientContext = new ClientContext("https://[tenant].sharepoint.com/sites/[modernsite]"))
{
    // Provide a valid set of credentials
    clientContext.Credentials = OfficeDevPnP.Core.Utilities.CredentialManager.GetSharePointOnlineCredential("[name-of-your-credentials]");

    // Get an Azure AD Access Token using ADAL, MSAL, or whatever else you like
    // This is needed only if your target is a "modern" team site
    var accessToken = getAzureADAccessToken();

    // Update the classification value, where the accessToken is an optional argument
    clientContext.Site.SetSiteClassification("MBI", accessToken);

    // Read back the new classification value (it can take a while to get back the new value)
    var currentClassification = clientContext.Site.GetSiteClassification();
}
```

## See also

- [Customizing the "modern" experiences in SharePoint Online](modern-experience-customizations.md)
- [Implement a SharePoint site classification solution](implement-a-sharepoint-site-classification-solution.md)
- [Manage Teams connected sites and channel sites](/SharePoint/teams-connected-sites)
