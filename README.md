---
services: active-directory
platforms: dotnet
author: dstrockis
---

Integrating a Windows Phone application with Azure AD
=========================

This sample demonstrates a Windows Store application for Windows Phone 8.1 calling a web API that is secured using Azure AD. The application uses the Active Directory Authentication Library (ADAL) to obtain a JWT access token through the OAuth 2.0 protocol. The access token is sent to the web API to authenticate the user.

For more information about how the protocols work in this scenario and other scenarios, see [Authentication Scenarios for Azure AD](http://go.microsoft.com/fwlink/?LinkId=394414).

## How To Run This Sample

To run this sample you will need:
- Visual Studio 2013 Update 2 or later
- Windows 8.1 or higher
- A machine supporting Client Hyper-V and Second Level Address Translation (SLAT)
- An Internet connection
- An Azure subscription (a free trial is sufficient)
- A Microsoft account

Every Azure subscription has an associated Azure Active Directory tenant.  If you don't already have an Azure subscription, you can get a free subscription by signing up at [https://azure.microsoft.com](https://azure.microsoft.com).  All of the Azure AD features used by this sample are available free of charge.

### Step 1:  Clone or download this repository

From your shell or command line:

`git clone https://github.com/Azure-Samples/active-directory-dotnet-windowsphone-8.1.git`

### Step 2:  Create a user account in your Azure Active Directory tenant

If you already have a user account in your Azure Active Directory tenant, you can skip to the next step.  This sample will not work with a Microsoft account, so if you signed in to the Azure portal with a Microsoft account and have never created a user account in your directory before, you need to do that now.  If you create an account and want to use it to sign-in to the Azure portal, don't forget to add the user account as a co-administrator of your Azure subscription.

### Step 3:  Register the sample with your Azure Active Directory tenant

There are two projects in this sample.  Each needs to be separately registered in your Azure AD tenant.

#### Register the TodoListService web API

1. Sign in to the [Azure management portal](https://manage.windowsazure.com).
2. Click on Active Directory in the left hand nav.
3. Click the directory tenant where you wish to register the sample application.
4. Click the Applications tab.
5. In the drawer, click Add.
6. Click "Add an application my organization is developing".
7. Enter a friendly name for the application, for example "TodoListService", select "Web Application and/or Web API", and click next.
8. For the sign-on URL, enter the base URL for the sample, which is by default `https://localhost:44321`.
9. For the App ID URI, enter `https://<your_tenant_name>/TodoListService`, replacing `<your_tenant_name>` with the name of your Azure AD tenant.  Click OK to complete the registration.
10. While still in the Azure portal, click the Configure tab of your application.
11. Find the Client ID value and copy it aside, you will need this later when configuring your application.

#### Find the TodoListClient app's redirect URI

Before you can register the TodoListClient application in the Azure portal, you need to find out the application's redirect URI.  Windows Phone 8.1 provides each application with a unique URI and ensures that messages sent to that URI are only sent to that application.  To determine the redirect URI for your project:

1. Open the solution in Visual Studio 2013.
2. In the TodoListClient project, open the `MainPage.xaml.cs` file.
3. Find this line of code and set a breakpoint on it.

```C#
 Uri redirectURI = Windows.Security.Authentication.Web.WebAuthenticationBroker.GetCurrentApplicationCallbackUri();
```

4. Right-click on the TodoListClient project and Debug --> Start New Instance.
5. When the breakpoint is hit, use the debugger to determine the value of redirectURI, and copy it aside for the next step.
6. Stop debugging, and clear the breakpoint.

The redirectURI value will look something like this:

```
ms-app://s-1-15-2-2123189467-1366327299-2057240504-936110431-2588729968-1454536261-950042884/
```

#### Register the TodoListClient app

1. Sign in to the [Azure management portal](https://manage.windowsazure.com).
2. Click on Active Directory in the left hand nav.
3. Click the directory tenant where you wish to register the sample application.
4. Click the Applications tab.
5. In the drawer, click Add.
6. Click "Add an application my organization is developing".
7. Enter a friendly name for the application, for example "TodoListClient-WindowsPhone", select "Native Client Application", and click next.
8. Enter the Redirect URI value that you obtained during the previous step.  Click finish.
9. Click the Configure tab of the application.
10. Find the Client ID value and copy it aside, you will need this later when configuring your application.
11. In "Permissions to Other Applications", click "Add Application."  Select "Other" in the "Show" dropdown, and click the upper check mark.  Locate & click on the TodoListService, and click the bottom check mark to add the application.  Select "Access TodoListService" from the "Delegated Permissions" dropdown, and save the configuration.

### Step 4:  Configure the sample to use your Azure AD tenant

#### Configure the TodoListService project

1. Open the solution in Visual Studio 2013.
2. Open the `web.config` file.
3. Find the app key `ida:Tenant` and replace the value with your AAD tenant name.
4. Find the app key `ida:Audience` and replace the value with the App ID URI you registered earlier, for example `https://<your_tenant_name>/TodoListService`.
5. Find the app key `ida:ClientId` and replace the value with the Client ID for the TodoListService from the Azure portal.

#### Configure the TodoListClient project

1. Open `MainPage.xaml.cs'.
2. Find the declaration of `tenant` and replace the value with the name of your Azure AD tenant.
3. Find the declaration of `clientId` and replace the value with the Client ID from the Azure portal.
4. Find the declaration of `todoListResourceId` and `todoListBaseAddress` and ensure their values are set properly for your TodoListService project.

### Step 5:  Trust the IIS Express SSL certificate from the Windows Phone Emulator

Since the web API is SSL protected, the client of the API will refuse the SSL connection to the web API unless it trusts the API's SSL certificate.  Use the following steps to add the IIS Express development certificate in the Windows Phone emulator. You will need to repeat the import operation every time you restart the emulator.  If you fail to do this step, calls to the TodoListService will always fail with a 404 code.
> Note: if you want to avoid having to repeat this task every time, you can deploy the web API to a host which uses an SSL certificate that the emulator already trusts. An Azure Web Site is a good example.

This task includes two operations: exporting the certificate from your development machine and importing it in the emulator. The first operation needs to be performed only once; the second operation will have to be repeated every time the emulator restarts.

#### Export the certificate from the development machine

1. While on the Start screen, type "certificates". Choose "Manage Computer Certificates" from the suggested results list.
2. On the left side tree, open Personal/Certificates. On the right pane you will see a list of certificates, including one entry named "localhost".
3. Right click on "localhost", choose "All Tasks->Export...".
4. Click "next" without changing the defaults until you reach the "Export file format" screen. Choose the 3rd option (.P7B) and click "next"
5. Save the certificate as "localhost.p7b" in <YourCloneFolder>\NativeClient-WindowsPhone8.1\TodoListService\Content.

#### Import the certificate in the emulator

1. Start the TodoListService project in the debugger (right-click on the project in Solution Explorer, select "Debug->Start New Instance"). Ensure that the browser appears and is correctly populated with the default ASP.NET UI
2. Start the emulator by starting the client project (right-click on the project in Solution Explorer, select "Debug->Start New Instance").
3. As soon as the emulator boots, hit the Windows button to get to the tiles list. Launch Mobile Internet Explorer.  
4. Navigate to "https://localhost:44321/Content/localhost.p7b". Choose "continue to web site".
5. When prompted, choose to open the file. You will be asked if you want to install the certificate. Choose "Install". If the operation succeeds, you will see a confirmation dialog. 

From now on, your emulator instance will be able to perform web API calls against projects running on your local IIS Express. Please note that as soon as you will close the emulator, the settings will be lost and you'll need to repeat the import operation.

### Step 7:  Run the sample

Clean the solution, rebuild the solution, and run it.  You might want to go into the solution properties and set both projects as startup projects, with the service project starting first.

On the main screen, hit the refresh button. You will be prompted to sign in. Once done so explore the sample by adding items to the To Do list, removing the user account, and starting again.  Notice that if you stop the application without removing the user account, the next time you run the application you won't be prompted to sign-in again - that is because ADAL has a persistent cache, and remembers the tokens from the previous run.

## How To Deploy This Sample to Azure

Coming soon.

## About The Code

Coming soon.

## How To Recreate This Sample

Coming soon.
