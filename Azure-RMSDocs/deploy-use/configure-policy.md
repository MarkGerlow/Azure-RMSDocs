---
# required metadata

title: Configure the Azure Information Protection policy
description: To configure classification, labeling, and protection, you must configure the Azure Information Protection policy. 
author: cabailey
ms.author: cabailey
manager: mbaldwin
ms.date: 03/15/2017
ms.topic: article
ms.prod:
ms.service: information-protection
ms.technology: techgroup-identity
ms.assetid: ba0e8119-886c-4830-bd26-f98fb14b2933

# optional metadata

#ROBOTS:
#audience:
#ms.devlang:
ms.reviewer: eymanor
ms.suite: ems
#ms.tgt_pltfrm:
#ms.custom:

---

# Configuring the Azure Information Protection policy

>*Applies to: [Azure Information Protection](https://azure.microsoft.com/pricing/details/information-protection)*

To configure classification, labeling, and protection, you must configure the Azure Information Protection policy. This policy is then downloaded to computers that have installed the [Azure Information Protection client](https://www.microsoft.com/en-us/download/details.aspx?id=53018).

## Subscription support

Azure Information Protection supports different levels of subscriptions:

- Azure Information Protection P2: Support for all classification, labeling, and protection features.

- Azure Information Protection P1: Support for most classification, labeling, and protection features, but not automatic classification or HYOK.

- Office 365 that includes the Azure Rights Management service: Support for protection but not classification and labeling.

Options that require an Azure Information Protection P2 subscription are identified in the portal.

If your organization has a mix of subscriptions, it is your responsibility to make sure that users do not use features that their account is not licensed to use. The Azure Information Protection client does not do license checking and enforcement. When you configure options that not all users have a license for, use scoped policies or a registry setting to ensure that your organization stays in compliance with your licenses:

- **When your organization has a mix of Azure Information Protection P1 and Azure Information Protection P2 licenses**: For users who have a P2 license, create and use one or more [scoped policies](configure-policy-scope.md) when you configure options that require an Azure Information Protection P2 license. Make sure that your global policy does not contain options that require an Azure Information Protection P2 license.

- **When your organization has a subscription for Azure Information Protection but some users have only a license for Office 365 that includes the Azure Rights Management service**: For the users who do not have a license for Azure Information Protection, edit the registry on their computers so they do not download the Azure Information Protection policy. For instructions, see the admin guide for the following customization: [Enforce protection-only mode when your organization has a mix of licenses](../rms-client/client-admin-guide-customizations.md#enforce-protection-only-mode-when-your-organization-has-a-mix-of-licenses).

For more information about the subscriptions, see [What subscription do I need for Azure Information Protection and what features are included?](../get-started/faqs.md#what-subscription-do-i-need-for-azure-information-protection-and-what-features-are-included)

## Signing in to the Azure portal

To sign in to the Azure portal, to configure and manage Azure Information Protection:

- Use the following link: https://portal.azure.com

- Use an account that has one of the following [administrator roles](/azure/active-directory/active-directory-assign-admin-roles-azure-portal):
    
    - **Information Protection Administrator**

    - **Security Administrator**

    - **Global Administrator / Company Administrator**


## To access the Azure Information Protection blade for the first time

1. Sign in to the Azure portal.

2. On the hub menu, click **Create a resource**, and then, from the **MARKETPLACE** list, select **Security + Identity**. 
    
3. On the **Security + Identify** blade, from the **FEATURED APPS** list, select **Azure Information Protection**. Then, on the **Azure Information Protection** blade, click **Create**.
    
    This action creates the **Azure Information Protection** blade for your tenant so that the next time you sign in to the portal, you can select the service from the hub **All services** list. 
    
    > [!TIP] 
    > Select **Pin to dashboard** to create an **Azure Information Protection** tile on your dashboard, so that you can skip browsing to the service the next time you sign in to the portal.

4. You see the **Quick start** page that automatically opens the first time you connect to the service. Browse the suggested resources, or use the other menu options. To configure the labels that users can select, use the following procedure.

Next time you access the **Azure Information Protection** blade, it automatically selects the **POLICIES** > **Global policy** option so that you can configure labels for all users. You can return to the **Quick start** page by selecting it from the **GENERAL** menu.

## How to configure the Azure Information Protection policy

1. Make sure that you are signed in to the Azure portal by using one of these administrative roles: Information Protection Administrator, Security Administrator, or Global Administration. See the [preceding section](#signing-in-to-the-azure-portal) for more information about these administrative roles.

2. If necessary, navigate to the **Azure Information Protection** blade: For example, on the hub menu, click **All services** and start typing **Information Protection** in the Filter box. From the results, select **Azure Information Protection**. 
    
    The **Azure Information Protection - Global policy** blade automatically opens for you to view and edit the global policy that all users get. 
    
    The Azure Information Protection policy contains the following elements that you can configure:
    
    - Labels that let you and users classify documents and emails.
    
    - Title and tooltip for the Information Protection bar that users see in their Office applications.
    
    - The option to enforce classification when users save documents and send emails.
    
    - The option to set a default label as a starting point for classifying documents and emails.
    
    - The option to prompt users to provide a reason when they select a label that has a lower sensitivity level than the original.
    
    - The option to automatically label an email message, based on its attachments.
    
    - The option to provide a custom help link for users.

Azure Information Protection comes with a [default policy](configure-policy-default.md), which contains five main labels. Two of these labels contain sublabels to provide subcategories, when needed. When a label is configured for sublabels, users cannot select the main label but must select one of the sublabels.

The Azure Information Protection labels can be used with the full range of data that an organization typically creates and stores, from the lowest classification of personal data, to the highest classification of highly confidential data. 

You can use the default labels without changes, or you can customize them, or you can delete them, and you can create new labels. For more information, use the links in the next section to help you locate the relevant options and how to configure them.

You can create any number of labels. However, when they start to get too many for users to easily see and select the right label, create scoped policies so that users see only the labels that are relevant to them. There is an upper limit for labels that apply protection, which is 500.

When you make any changes on an Azure Information Protection blade, click **Save** to save the changes, or click **Discard** to revert to the last saved settings.

When you have finished making the changes that you want, click **Publish**. 

The Azure Information Protection client checks for any changes whenever a supported Office application starts, and downloads the changes as its latest Azure Information Protection policy. Additional triggers that refresh the policy on the client:

- Right-click to classify and protect a file or folder.

- Running the [PowerShell cmdlets](../rms-client/client-admin-guide-powershell.md) for labeling and protection (Get-AIPFileStatus, Set-AIPFileClassification, and Set-AIPFileLabel).

- Every 24 hours.

- For the [Azure Information Protection Scanner](deploy-aip-scanner.md): When the service starts (if the policy is older than an hour), and every hour during operation.


>[!NOTE]
>When the client downloads the policy, be prepared to wait a few minutes before it's fully operational. The actual time varies, according to factors such as the size and complexity of the policy configuration, and the network connectivity. If the resulting action of your labels does not match your latest changes, allow up to 15 minutes and then try again.

### Configuring your organization's policy

Use the following information to help you configure your Azure Information Protection policy:

- [The default Information Protection policy](configure-policy-default.md)

- [How to configure the policy settings](configure-policy-settings.md)

- [How to create a new label](configure-policy-new-label.md)

- [How to delete or reorder a label](configure-policy-delete-reorder.md)

- [How to change or customize an existing label](configure-policy-change-label.md)

- [How to configure a label for protection](configure-policy-protection.md)

- [How to configure a label to apply visual markings](configure-policy-markings.md)

- [How to configure conditions for automatic and recommended classification](configure-policy-classification.md)

- [How to configure the policy for specific users by using scoped policies](configure-policy-scope.md)

- [How to configure and manage templates](configure-policy-templates.md)

- [How to configure labels for different languages](configure-policy-languages.md)

## Next steps

For an example of how to customize the default policy, and see the resulting behavior in an Office application, try the [Quick start tutorial for Azure Information Protection](../get-started/infoprotect-quick-start-tutorial.md).

[!INCLUDE[Commenting house rules](../includes/houserules.md)]
