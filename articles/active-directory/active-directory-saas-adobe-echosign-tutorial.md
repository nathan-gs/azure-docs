---
title: 'Tutorial: Azure Active Directory integration with Adobe Sign | Microsoft Docs'
description: Learn how to configure single sign-on between Azure Active Directory and Adobe Sign.
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila

ms.assetid: f9385723-8fe7-4340-8afb-1508dac3e92b
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/24/2017
ms.author: jeedes

---
# Tutorial: Azure Active Directory integration with Adobe Sign

In this tutorial, you learn how to integrate Adobe Sign with Azure Active Directory (Azure AD).

Integrating Adobe Sign with Azure AD provides you with the following benefits:

- You can control in Azure AD who has access to Adobe Sign
- You can enable your users to automatically get signed-on to Adobe Sign (Single Sign-On) with their Azure AD accounts
- You can manage your accounts in one central location - the Azure portal

If you want to know more details about SaaS app integration with Azure AD, see [what is application access and single sign-on with Azure Active Directory](active-directory-appssoaccess-whatis.md).

## Prerequisites

To configure Azure AD integration with Adobe Sign, you need the following items:

- An Azure AD subscription
- An Adobe Sign single sign-on enabled subscription

> [!NOTE]
> To test the steps in this tutorial, we do not recommend using a production environment.

To test the steps in this tutorial, you should follow these recommendations:

- Do not use your production environment, unless it is necessary.
- If you don't have an Azure AD trial environment, you can get a one-month trial [here](https://azure.microsoft.com/pricing/free-trial/).

## Scenario description
In this tutorial, you test Azure AD single sign-on in a test environment. 
The scenario outlined in this tutorial consists of two main building blocks:

1. Adding Adobe Sign from the gallery
2. Configuring and testing Azure AD single sign-on

## Adding Adobe Sign from the gallery
To configure the integration of Adobe Sign into Azure AD, you need to add Adobe Sign from the gallery to your list of managed SaaS apps.

**To add Adobe Sign from the gallery, perform the following steps:**

1. In the **[Azure portal](https://portal.azure.com)**, on the left navigation panel, click **Azure Active Directory** icon. 

	![Active Directory][1]

2. Navigate to **Enterprise applications**. Then go to **All applications**.

	![Applications][2]
	
3. To add new application, click **New application** button on the top of dialog.

	![Applications][3]

4. In the search box, type **Adobe Sign**.

	![Creating an Azure AD test user](./media/active-directory-saas-adobe-echosign-tutorial/tutorial_adobesign_search.png)

5. In the results panel, select **Adobe Sign**, and then click **Add** button to add the application.

	![Creating an Azure AD test user](./media/active-directory-saas-adobe-echosign-tutorial/tutorial_adobesign_addfromgallery.png)

##  Configuring and testing Azure AD single sign-on
In this section, you configure and test Azure AD single sign-on with Adobe Sign based on a test user called "Britta Simon."

For single sign-on to work, Azure AD needs to know what the counterpart user in Adobe Sign is to a user in Azure AD. In other words, a link relationship between an Azure AD user and the related user in Adobe Sign needs to be established.

In Adobe Sign, assign the value of the **user name** in Azure AD as the value of the **Username** to establish the link relationship.

To configure and test Azure AD single sign-on with Adobe Sign, you need to complete the following building blocks:

1. **[Configuring Azure AD Single Sign-On](#configuring-azure-ad-single-sign-on)** - to enable your users to use this feature.
2. **[Creating an Azure AD test user](#creating-an-azure-ad-test-user)** - to test Azure AD single sign-on with Britta Simon.
3. **[Creating an Adobe Sign test user](#creating-an-adobe-sign-test-user)** - to have a counterpart of Britta Simon in Adobe Sign that is linked to the Azure AD representation of user.
4. **[Assigning the Azure AD test user](#assigning-the-azure-ad-test-user)** - to enable Britta Simon to use Azure AD single sign-on.
5. **[Testing Single Sign-On](#testing-single-sign-on)** - to verify whether the configuration works.

### Configuring Azure AD single sign-on

In this section, you enable Azure AD single sign-on in the Azure portal and configure single sign-on in your Adobe Sign application.

**To configure Azure AD single sign-on with Adobe Sign, perform the following steps:**

1. In the Azure portal, on the **Adobe Sign** application integration page, click **Single sign-on**.

	![Configure Single Sign-On][4]

2. On the **Single sign-on** dialog, select **Mode** as	**SAML-based Sign-on** to enable single sign-on.
 
	![Configure Single Sign-On](./media/active-directory-saas-adobe-echosign-tutorial/tutorial_adobesign_samlbase.png)

3. On the **Adobe Sign Domain and URLs** section, perform the following steps:

	![Configure Single Sign-On](./media/active-directory-saas-adobe-echosign-tutorial/tutorial_adobesign_url.png)

    a. In the **Sign-on URL** textbox, type a URL using the following pattern: `https://<companyname>.echosign.com/`

	b. In the **Identifier** textbox, type a URL using the following pattern: `https://<companyname>.echosign.com`

	> [!NOTE] 
	> These values are not real. Update these values with the actual Sign-On URL and Identifier. Contact [Adobe Sign Client support team](https://helpx.adobe.com/in/contact/support.html) to get these values. 
 
4. On the **SAML Signing Certificate** section, click **Certificate(Base64)** and then save the certificate file on your computer.

	![Configure Single Sign-On](./media/active-directory-saas-adobe-echosign-tutorial/tutorial_adobesign_certificate.png) 

5. Click **Save** button.

	![Configure Single Sign-On](./media/active-directory-saas-adobe-echosign-tutorial/tutorial_general_400.png)

6. On the **Adobe Sign Configuration** section, click **Configure Adobe Sign** to open **Configure sign-on** window. Copy the **Sign-Out URL, SAML Entity ID, and SAML Single Sign-On Service URL** from the **Quick Reference section.**

	![Configure Single Sign-On](./media/active-directory-saas-adobe-echosign-tutorial/tutorial_adobesign_configure.png) 


7. In a different web browser window, log in to your Adobe Sign company site as an administrator.

8. In the menu on the top, click **Account**, and then, in the navigation pane on the left side, click **SAML Settings** under **Account Settings**.
   
   ![Account](./media/active-directory-saas-adobe-echosign-tutorial/ic789520.png "Account")

9. In the SAML Settings section, perform the following steps:
   
   ![SAML Settings](./media/active-directory-saas-adobe-echosign-tutorial/ic789521.png "SAML Settings")
   
   a. As **SAML Mode**, select **SAML Mandatory**.
   
   b. Select **Allow EchoSign Account Administrators to log in using their EchoSign Credentials**.
   
   c. As **User Creation**, select **Automatically add users authenticated through SAML**.

10. Move on, performing the following steps:

	   ![SAML Settings](./media/active-directory-saas-adobe-echosign-tutorial/ic789522.png "SAML Settings")

	a. Paste **SAML Entity ID**, which you have copied from Azure portal into the **IdP Entity ID** textbox.
   	
	b. Paste **SAML Single Sign-On Service URL**, which you have copied from Azure portal into the **IdP Login URL** textbox.
   
    c. Paste **Sign-Out URL**, which you have copied from Azure portal into the **IdP Logout URL** textbox.

	d. Open your downloaded **Certificate(Base64)** file in notepad, copy the content of it into your clipboard, and then paste it to the **IdP Certificate** textbox

	e. Click **Save Changes**.

> [!TIP]
> You can now read a concise version of these instructions inside the [Azure portal](https://portal.azure.com), while you are setting up the app!  After adding this app from the **Active Directory > Enterprise Applications** section, simply click the **Single Sign-On** tab and access the embedded documentation through the **Configuration** section at the bottom. You can read more about the embedded documentation feature here: [Azure AD embedded documentation]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

### Creating an Azure AD test user
The objective of this section is to create a test user in the Azure portal called Britta Simon.

![Create Azure AD User][100]

**To create a test user in Azure AD, perform the following steps:**

1. In the **Azure portal**, on the left navigation pane, click **Azure Active Directory** icon.

	![Creating an Azure AD test user](./media/active-directory-saas-adobe-echosign-tutorial/create_aaduser_01.png) 

2. To display the list of users, go to **Users and groups** and click **All users**.
	
	![Creating an Azure AD test user](./media/active-directory-saas-adobe-echosign-tutorial/create_aaduser_02.png) 

3. To open the **User** dialog, click **Add** on the top of the dialog.
 
	![Creating an Azure AD test user](./media/active-directory-saas-adobe-echosign-tutorial/create_aaduser_03.png) 

4. On the **User** dialog page, perform the following steps:
 
	![Creating an Azure AD test user](./media/active-directory-saas-adobe-echosign-tutorial/create_aaduser_04.png) 

    a. In the **Name** textbox, type **BrittaSimon**.

    b. In the **User name** textbox, type the **email address** of BrittaSimon.

	c. Select **Show Password** and write down the value of the **Password**.

    d. Click **Create**.
 
### Creating an Adobe Sign test user

To enable Azure AD users to log in to Adobe Sign, they must be provisioned into Adobe Sign. In the case of Adobe Sign, provisioning is a manual task.

>[!NOTE]
>You can use any other Adobe Sign user account creation tools or APIs provided by Adobe Sign to provision AAD user accounts. 

**To provision a user account, perform the following steps:**

1. Log in to your **Adobe Sign** company site as administrator.

2. In the menu on the top, click **Account**, and then, in the navigation pane on the left side, click **Users & Groups**, and then, click **Create a new user**.
   
   ![Account](./media/active-directory-saas-adobe-echosign-tutorial/ic789524.png "Account")
   
3. In the **Create New User** section, perform the following steps:
   
   ![Create User](./media/active-directory-saas-adobe-echosign-tutorial/ic789525.png "Create User")
   
   a. Type the **Email Address**, **First Name**, and **Last Name** of a valid AAD account you want to provision into the related textboxes.
   
   b. Click **Create User**.

>[!NOTE]
>The Azure Active Directory account holder receives an email that includes a link to confirm the account before it becomes active. 

### Assigning the Azure AD test user

In this section, you enable Britta Simon to use Azure single sign-on by granting access to Adobe Sign.

![Assign User][200] 

**To assign Britta Simon to Adobe Sign, perform the following steps:**

1. In the Azure portal, open the applications view, and then navigate to the directory view and go to **Enterprise applications** then click **All applications**.

	![Assign User][201] 

2. In the applications list, select **Adobe Sign**.

	![Configure Single Sign-On](./media/active-directory-saas-adobe-echosign-tutorial/tutorial_adobesign_app.png) 

3. In the menu on the left, click **Users and groups**.

	![Assign User][202] 

4. Click **Add** button. Then select **Users and groups** on **Add Assignment** dialog.

	![Assign User][203]

5. On **Users and groups** dialog, select **Britta Simon** in the Users list.

6. Click **Select** button on **Users and groups** dialog.

7. Click **Assign** button on **Add Assignment** dialog.
	
### Testing single sign-on

When you click the Adobe Sign tile in the Access Panel, you should get automatically signed-on to your Adobe Sign application.
 For more information about the Access Panel, see [Introduction to the Access Panel](active-directory-saas-access-panel-introduction.md).

## Additional resources

* [List of Tutorials on How to Integrate SaaS Apps with Azure Active Directory](active-directory-saas-tutorial-list.md)
* [What is application access and single sign-on with Azure Active Directory?](active-directory-appssoaccess-whatis.md)

<!--Image references-->

[1]: ./media/active-directory-saas-adobe-echosign-tutorial/tutorial_general_01.png
[2]: ./media/active-directory-saas-adobe-echosign-tutorial/tutorial_general_02.png
[3]: ./media/active-directory-saas-adobe-echosign-tutorial/tutorial_general_03.png
[4]: ./media/active-directory-saas-adobe-echosign-tutorial/tutorial_general_04.png

[100]: ./media/active-directory-saas-adobe-echosign-tutorial/tutorial_general_100.png

[200]: ./media/active-directory-saas-adobe-echosign-tutorial/tutorial_general_200.png
[201]: ./media/active-directory-saas-adobe-echosign-tutorial/tutorial_general_201.png
[202]: ./media/active-directory-saas-adobe-echosign-tutorial/tutorial_general_202.png
[203]: ./media/active-directory-saas-adobe-echosign-tutorial/tutorial_general_203.png

