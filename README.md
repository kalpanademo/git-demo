[[_TOC_]]
# <span style="color:#00bfff">Introduction</span>
Deloitte Cloud Accelerator (DCA) is a multi-cloud deployment solution that is designed to help enterprises rapidly build cloud competencies, accelerate transition to Cloud, and transform their business in a cloud-first world.

Enterprise Scale Landing Zone Helper is a re-usable asset which can be used across engagement to build an Enterprise Standard Azure based Landing Zone using Azure native tools. This can be customized based on the various use cases and requirements specific to the engagement.

# <span style="color:#00bfff">Intended Audience<span>
This document is intended for the Azure Cloud Engineering and DevOps teams which are in process of building an Azure Landing Zone.

# <span style="color:#00bfff">DCA Enterprise Scale landing Zone Implementation<span>
##What is an Enterprise Scale Landing Zone?
Enterprise Scale Landing Zones are a core capability enabled through the Microsoft Azure Cloud Adoption Framework. ESLZ can be implemented on the following topologies.
1. Hub & Spoke Network Topology
A hub and spoke network topology allow you to create a central Hub Virtual Network that contains shared networking components (such as Azure Firewall) that can then be used by spoke Virtual Networks, connected to the Hub Virtual Network via VNET Peering, to centralize connectivity in your environment. Gateway transit in Virtual Network peering allows spokes to have connectivity to/from on-premises via ExpressRoute or VPN (Virtual Private Network), and transitive connectivity across spokes can be implemented by deploying User Defined Routes (UDR) on the spokes and using Azure Firewall or an NVA in the hub as the transit resource. For more information about Hub & Spoke network topology, please refer to this [link](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?tabs=cli).
2. Azure Virtual WAN Topology
Azure Virtual WAN is a Microsoft-managed solution that provides end-to-end, global, and dynamic transit connectivity by default. Virtual WAN hubs eliminate the need to manually configure network connectivity. For example, you don't need to manage user-defined routes (UDR) or network virtual appliances (NVAs) to enable global transit connectivity.
Virtual WAN simplifies end-to-end network connectivity in Azure, and to Azure from on-premises, by creating a hub-and-spoke network architecture. The architecture can easily scale to support multiple Azure regions and on-premises locations (any-to-any connectivity).

<u>Note:</u> The current version of the ESLZ implementation supports Hub & Spoke Network Topology implementation. Azure Virtual WAN will be supported in the later versions.

##What does the Enterprise Scale Landing Zone Implementation provide?
ESLZ Implementation provides the following
1. Readymade YAML based pipelines for deployment using ARM, Bicep and PowerShell
ESLZ provides readymade Azure DevOps YAML based pipelines which can be configured on your DevOps project with minimal changes. These pipelines have well defined stages and tasks. These pipelines are as follows:
   - Validation Pipeline – Used to validate the scripts when a change is detected in the designated branch.
   - Build Pipeline – Used to build the foundation elements when a change is detected in the designated branch.
   - Teardown Pipeline – Used to tear down the environment.


   Each of the above pipelines in turn uses the following
   - PowerShell scripts to orchestrate the build of the foundational elements
   - ARM Templates to build the foundational elements like Management Groups, Resource Groups, Policies, Hub Virtual Network, and other network elements
   - Biceps modules to deploy ESLZ solution at different scopes.

   <u>Note:</u> The ARM templates are being planned to be replaced by Biceps scripts as part of future enhancements of the ESLZ implementation.

2. A landing zone with the following components:
   - Standard Management Group Hierarchy which can customized.
   - Standard set of Azure Policies and Initiatives assigned to the respective Management Groups
   - Subscription Assignment to the respective Management Groups
   - Services within the identified Management Subscription, which include
     - Log Analytics Workspace
     - Automation Account
     - Azure Security Center
     - Azure Sentinel
     - Configuration of Diagnostic settings of Activity Logs, VMs and PaaS resources to be sent to Log Analytics
   - Core Networking Resources within the identified Connectivity Subscription, which include
     - Hub Virtual Network
     - Azure Firewall (with option to deploy across Availability Zones)
     - ExpressRoute Gateway (with option to deploy across Availability Zones)
     - VPN Gateway (with option to deploy across Availability Zones)
     - Azure Private DNS Zones for Private Link
     - Azure DDoS Standard Protection Plan

3. Deploys a landing zone conforming to the following design
The landing zone being built by the ESLZ implementation conforms to the design based on Azure Landing Zone defined within Microsoft’s Cloud Adoption Framework (More details are available [here](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/landing-zone/)). The below diagram indicates the design of the landing zone.

   ![Arch2.png](/.attachments/Arch2-e93b1080-48af-4613-b4b7-135d9d52c9da.png)

   <u>Note:</u> For better visibility, please open the image available [here](https://resources.deloitte.com/:p:/r/sites/MSAzureOfferings/Shared%20Documents/Azure%20COE/Microsoft%20GSSC%20documents/Deloitte%20Cloud%20Accelerator%20(DCA)%20-%20Azure/Reference%20Architecture%20Diagrams/DCA%20-%20Ref%20Architecture%20Diagram%20Reformatted%20July.pptx?d=we9ecb1d272ae457aa2aa5bee6e4c3951&csf=1&web=1&e=EsMwCq)

This design consists of the following: 
   - A scalable Management Group hierarchy aligned to core platform capabilities, allowing operationalization at scale using centrally managed Azure RBAC and Azure Policy where platform and workloads have clear separation.
   - Azure Policies that will enable autonomy for the platform and the landing zones.
   - A management Subscription, which enables core platform capabilities at scale using Azure Policy such as a Log Analytics workspace and an Automation account, Azure Security Center monitoring, Azure Security Center (Standard or Free tier), Azure Sentinel, Diagnostics settings for Activity Logs, VMs, and PaaS resources sent to Log Analytics
   - A connectivity Subscription, which deploys core Azure networking resources such as a hub virtual network, Azure Firewall (optional - deployment across Availability Zones), ExpressRoute Gateway (optional - deployment across Availability Zones), VPN Gateway (optional - deployment across Availability Zones), Azure Private DNS Zones for Private Link (optional), Azure DDoS Standard protection plan (optional)
   - An Identity Subscription in case there is a requirement to have Active Directory Domain Controllers in a dedicated subscription.
   - A virtual network will be deployed and will be connected to the hub VNet via VNet peering.
   - Landing Zone Management Group for corporate connected applications that require connectivity to on-premises, to other landing zones or to the internet via shared services provided in the hub virtual network. This is where you will create your subscriptions that will host your corp-connected workloads.
   - Landing Zone Management Group for online applications that will be internet-facing, where a virtual network is optional and hybrid connectivity is not required. This is where you will create your Subscriptions that will host your online workloads.
   - Landing zone subscriptions for Azure native, internet-facing online applications, and resources.
   - Azure Policies for online and corp-connected landing zones, which include enforce VM monitoring (Windows & Linux), enforce VMSS monitoring (Windows & Linux), enforce Azure Arc VM monitoring (Windows & Linux), enforce VM backup (Windows & Linux), enforce secure access (HTTPS) to storage accounts, enforce auditing for Azure SQL, enforce encryption for Azure SQL, prevent IP forwarding, prevent inbound RDP from internet , ensure subnets are associated with NSG, associate private endpoints with Azure Private DNS Zones for Azure PaaS services.
   - Sandbox is the dedicated Management Group for subscriptions that will solely be used for testing and exploration by an organization’s application teams. These subscriptions will be securely disconnected from the Corp and Online landing zones.

4. Infra state management using AzOps
AzOps is an open-source project from Microsoft, which is an atomic, primitive CI/CD pipeline for the Azure platform scopes (tenant, management groups, and subscriptions), which is available [here](https://github.com/Azure/AzOps). This is used to extract the state as ARM templates from the subscriptions and store the same within the designated Git repository. 
ESLZ implementation uses AzOps to pull the state from the subscriptions and validate if there are changes done on the subscription outside of the designated pipelines.

5. Script validation using ARM ttk
The ESLZ implementation has integrated script validation for ARM templates and Bicep templates along with the pipeline using ARM ttk. This provides the required validation before executing the templates to build the platform elements.
<br/>
# <span style="color:#00bfff">Enterprise Scale Landing Zone Foundation Setup Execution Flow</span>
The following diagram details the execution flow of the Enterprise Scale Landing Zone foundation setup. 

![Arch1.png](/.attachments/Arch1-5577c8a9-974a-4bf5-8ea9-4cdfe129a77c.png)

Following are the execution steps mentioned in the diagram
- Step 1: Changes in the ESLZ repository’s foundation setup code are detected by Azure DevOps within the target branch and the ESLZ Setup pipeline is triggered automatically. (Note: This can also be manually triggered based on the need)
- Step 2: This is an optional step where the existing landing is torn down completely before starting the setup. (Note: The foundation setup runs in an incremental mode so that only the changes are setup irrespective of whether the teardown is executed or not)
- Step 3: The ESLZ setup pipeline executes its PowerShell based orchestration to initiate the setup of the foundation elements.
- Step 4: The respective ARM / Bicep templates are executed to create the required foundational elements
- Step 5: The foundation elements are deployed in the Azure platform
- Step 6: Once the foundation elements are deployed in the Azure platform, the AzOps Pull pipeline is triggered. 
- Step 7: The AzOps pull pipeline pulls all the deployment configurations in the target subscriptions and stores the same within the AzOps repository to maintain state.
- Step 8: The AzOps pull pipeline triggers the AzOps Push pipeline to push any configuration changes detected. Usually, there are no changes when the ESLZ Setup pipeline is called before the Push Pipeline is triggered.
<br/>
# <span style="color:#00bfff">DCA Enterprise Scale Landing Zone Implementation Extensibility</span>
This section provides the details on how the DCA ESLZ implementation can be extensible for individual project needs.

The various extensibility points are as follows:
- Plugging in Addition build scripts: ARM templates / Bicep scripts can be added for those additional Azure services which are not covered in the current version, as per design needs.
- Modular pipelines: Additional stages can be added within the DevOps pipelines as per the requirement.
- Landing Zone customization: Landing zone build can be customized by updating the PowerShell Orchestration to include additional foundational elements as per design needs.
- Contribution from multiple engagements: This is available on Enterprise GitHub, with options for engagement teams to contribute additional features, which will be merged back into the ESLZ implementation.
<br/>

# <span style="color:#00bfff">Steps to implement the Enterprise Scale Landing zone using the DCA implementation</span>
![Picture1.png](/.attachments/Picture1-2425b69b-47ae-413f-a572-05cc9cde3ca1.png)
##Azure Platform Configuration

_**Pre-requisites**_
- Required subscriptions identified and created
- Management Group enabled with the Azure AD tenant
- Access to the subscriptions to verify the deployment (Note: Access Levels can vary based on the security policies of the organization)
- RBAC Access and / or elevated access to perform privileged actions needed to setup the Azure platform to enable ESLZ deployments.

_**Azure Subscriptions**_

As per the supported design, the following subscriptions are needed. (Note: This is one of the extensibility points, where different kind of subscriptions can be defined based on the requirements and design)
- A management Subscription, which enables core platform capabilities at scale using Azure Policy such as a Log Analytics workspace and an Automation account, Azure Security Center monitoring, Azure Security Center (Standard or Free tier), Azure Sentinel, Diagnostics settings for Activity Logs, VMs, and PaaS resources sent to Log Analytics
- A connectivity Subscription, which deploys core Azure networking resources such as a hub virtual network, Azure Firewall (optional - deployment across Availability Zones), ExpressRoute Gateway (optional - deployment across Availability Zones), VPN Gateway (optional - deployment across Availability Zones), Azure Private DNS Zones for Private Link (optional), Azure DDoS Standard protection plan (optional)
- An Identity Subscription in case your organization requires to have Active Directory Domain Controllers in a dedicated subscription.
- Landing Zone Management Group for corporate connected applications that require connectivity to on-premises, to other landing zones or to the internet via shared services provided in the hub virtual network. This is where you will create your subscriptions that will host your corp-connected workloads.
- Landing zone subscriptions for Azure native, internet-facing online applications, and resources.
- Sandbox is the dedicated Management Group for subscriptions that will solely be used for testing and exploration by an organization’s application teams. These subscriptions will be securely disconnected from the Corp and Online landing zones.

_**Create Service Principal**_

1. Create the Azure Active Directory (AAD) Service Principal that will be used by Azure DevOps’s service connection:
   - a. Make sure you have permissions to create Application Registrations in Azure Active Directory (AAD). Otherwise, activate the role if PIM (Privileged Identity Management) is in place or have someone that has either any of these AAD Roles assigned:
     - i. AAD Application Administrator
     - ii. AAD Application Developer
     - iii. AAD Global Administrator
   - b. Go to AAD
     - i. Click on “App Registrations”
     - ii. Click on “+ New Registration”
     - iii. Fill in the following fields:
       - Name             = Name of the App Registration
       - Supported account types = Typically select “Accounts in this org directory only”
       - Redirect URI (Optional)         = Do not type anything here. We will not need it for pipeline deployments
       - Click on Register

         ![Picture1.png](/.attachments/Picture1-f6c79c86-aeae-49a5-876e-3d747c38cf5d.png)

     - iv. Create an App Registration Secret
       - Once the Application registration is created, perform the following steps:
         - a. Under Manage, select “Certificates & Secrets”
         - b. Click on “+ New Client Secret”
         - c. Add a meaningful description (i.e., “AzDo-ESLZ-Pipelines”)
         - d. Select expiration time as per your organization’s secret management policies (Custom time up to 24 months).
           - i. Note: If the secret expires, a new one must be created and then the corresponding Azure DevOps Service Connection must be updated to use the new secret. Otherwise, the pipeline will fail.
         - e. Outcome:

              ![Picture2.png](/.attachments/Picture2-ac1b35f3-4bdb-406b-a40c-a1fecaad28c8.png)

         - f. <u>Take note of the secret</u> as it will be used when configuring the Azure DevOps Service Connection. Place it in a safe place <u>and treat it like a password</u>.

   - c. Make sure that only authorized users have access to this Application registration. Limit the number of users to a minimum (ideally by assigning an AAD Group as Owner).
     - i. Remember that this Application Registration/SPN will have Owner at the Root Management Group Level. It is of utmost importance to exercise caution and limit the owners
     - ii. This can be set by selecting “Owners” under Manage section of the application registration.
       - You need to have an AAD Group created previously for this.
   - d. Take note of the following
        - i. Application Registration Name 	= Will be used on step 3
        - ii. Application ID = Will be used when creating Azure DevOps Service Connection setup
        - iii. Tenant ID = Will be used when creating Azure DevOps Service Connection setup
        - iv. Secret Value = Will be used when creating Azure DevOps Service Connection setup  

     ![Picture3.png](/.attachments/Picture3-15034199-aad6-4e20-8077-fa54c43f03c6.png)

2. Elevate Access to manage Azure Resources in AAD
   - a. Sign into the Azure portal as a Global Administrator.
     - i. If using AAD PIM, elevate your access to a Global Administrator by activating the role assignment
   - b. Go to Azure Active Directory
   - c. Go to Properties
   - d. Scroll down, under “Access management for Azure resources”, toggle the setting to yes.
     - i. The outcome of this command is that you will be assigned the User Access Administrator Role at the Tenant Root Management Group level.

3. Grant Access to the AAD Service Principal that will be used by Azure DevOps’s service connection.
Note: Please ensure you are logged in as a user with UAA role enabled in AAD tenant and logged in user is not a guest user.
   - a. Manually:
     - i. Type “Management Groups” in the search bar
     - ii. Enable Management Groups if the tenant is brand new by clicking the button in the center of the Management Group splash page
     - iii. Go to the “Tenant Root Management Group” by clicking on that link  

       ![Picture3.png](/.attachments/Picture3-a6ef4a87-ca4f-493c-b6a7-9203b32da701.png)

     - iv. Under “Access Control (IAM (Identity & Access Management))”

       ![Picture4.png](/.attachments/Picture4-ba642062-c5fb-46df-837a-6b81ed76c847.png)

        - Select "+Add" to add role assignment
          
          a. Role = Owner

          b. Members = Type the name of the Application Registration created on step 1

          c. Click on “Review + Assign” button after checking that the correct AAD Application Registration has been selected.

     - v. Additionally, from the [cloud shell](https://docs.microsoft.com/en-us/azure/cloud-shell/overview) of your Azure portal console, run these commands, so you ensure SPN’s access on root tenant level:

       `Connect-AzAccount`
       `New-AzRoleAssignment -Scope '/' -RoleDefinitionName 'Owner' -ObjectId <obj-id>`

       Put the object ID of the Application you registered in the steps above.

To secure access to this Service Principal, please ensure the following:

1. Only authorized users should have access to this Application registration. Limit the number of users to a minimum (ideally by assigning an AAD Group as Owner).
2. Remember that this Application Registration/SPN will have Owner at the Root Management Group Level. It is of utmost importance to exercise caution and limit the owners
3. This can be set by selecting “Owners” under Manage section of the application registration. Note: You need to have an AAD Group created previously for this.

##Azure DevOps Configuration

_**Pre-requisites**_
- DevOps organization
- ARM ttk installed as an extension within the DevOps organization
- Basic User license or Visual Studio License for the user(s) setting up the ESLZ implementation.
- Project administrator access to the project for the users setting up the ESLZ implementation. This can be either through membership of the “Project Collection Administrators” group or a direct project administrator role assignment.

_**Setup Project**_

You create an Azure DevOps project to establish a repository for source code and to plan and track work. This can be done on Browser or Azure DevOps CLI.

![Picture1.png](/.attachments/Picture1-3d910b30-5a81-4f51-80ce-3f9470c33253.png)

For more information on how to setup a project , refer [this]( https://docs.microsoft.com/en-us/azure/devops/organizations/projects/create-project?view=azure-devops&tabs=azure-devops-cli) 

_**Setup Service Connection**_

**<u>Note:</u>** Please make sure you have a valid Application Registration created, the value of the App Registration’s secret and proper role assignments have been granted to the App Registration’s service principal. The process for this covered in the “Azure Platform Section”

To create a service connection in Azure DevOps, please follow the steps below:

- Go to your Azure DevOps project using the appropriate URL.
- Go to Project settings as highlighted.

  ![Picture2.png](/.attachments/Picture2-96fa9f20-de58-4072-81e7-615079ea9aed.png)

- Go to Service Connections under “Pipelines” as highlighted.

  ![Picture3.png](/.attachments/Picture3-0019bab5-f6ab-47ef-ba8c-3f6eb06f3d11.png)

- Click on “New Service Connection” option as highlighted.

  ![2.png](/.attachments/2-1127e5b9-2a57-4e63-8b8a-346e04f896ab.png)

- Choose Azure Resource Manager option as highlighted and click on Next Button which is available at the end of the list.

  ![Picture5.png](/.attachments/Picture5-729b06bc-790c-43f4-bc01-bf21ffdebcbc.png)

- In the authentication method, choose “Service Principal (Manual)” (1) and click on Next button (2)

  ![Picture6.png](/.attachments/Picture6-3a237b23-29c5-45ed-a645-0789dd4286c0.png)

- Provide the following inputs in the screen and click on “Verify” button which is highlighted.
   - Select Scope Level as “Subscription”
   - Subscription Id
   - Subscription Name
   - Service Principal Id
   - Select Credential as “Service principal key”
   - Service Principal key
   - Tenant Id

  ![Picture7.png](/.attachments/Picture7-123e9857-8f70-4aab-810f-47d6f07f80d3.png)

- Once verified, provide a Service Connection name to identify the connection and provide the description for the same for future reference. Once provided, click on “Verify and Save” button which is highlighted. This will save the Service Connection for future use.

  ![Picture8.png](/.attachments/Picture8-6f4e7922-895f-4ce7-9b77-bcc79fcacdc9.png)

**<u>Note:</u>** Please enable “Grant access permission to all pipelines”, only if needed. During the first execution of a pipeline using this service connection, the pipeline will request for an explicit grant of permission to the service connection. This will ensure that only those pipelines which uses the service connection will have the access for the same.

##ESLZ Code Configuration

_**Get Access to ESLZ Repo**_

The ESLZ code repository consists of the base ARM templates, Bicep templates, Azure DevOps pipelines, which can be extended for usage within projects.

The details pertaining to the ESLZ Code configuration is available [here](https://resources.deloitte.com/:w:/r/sites/MSAzureOfferings/_layouts/15/Doc.aspx?sourcedoc=%7B664954BC-9CD4-4FB3-8BCA-E649282A7B3A%7D&file=DCA-ESLZ%20Repo%20Document.docx&action=default&mobileredirect=true).

ESLZ codebase currently sits on [‘Microsoft Technology DevOps’](https://dev.azure.com/Microsoft-Technology-Practice/DCA-Innovation%20Core/_git/dca-core-eslz) organization. Reach out to [Sunil Lamichhane](mailto:sunlamichhane@deloitte.ca) or [Amita Sujith](mailto:asujith@deloitte.com) for access to repo.

_**Setup ESLZ Repo in ADO project**_

For this step, you will be needing the latest version of Visual Studio Code. Use [this link](https://code.visualstudio.com/download) to download the software.
- Once you get a handle on Repo, download the repo as a zip file :

  ![Picture3.png](/.attachments/Picture3-805a2a29-8f90-4243-bd80-9289689e96db.png)

- Unzip the file and save it as ‘Downloaded Repo’ somewhere in your local PC
- Go to OSDisk (C:) and create a folder called DCA-Azure

  ![Picture2.png](/.attachments/Picture2-283c1e9c-cda5-4ef3-8049-26e35128a8f5.png)

- Go into the Repo section of your ADO project and create a new repository:

  ![Picture3.png](/.attachments/Picture3-cc4ed538-ca28-4922-948b-e19f8070ecc9.png)

- On Repository Type, select ‘Git’ > name it ‘ESLZ’ > Click on ‘Create’

  ![Picture4.png](/.attachments/Picture4-057bfdb2-2802-4597-bdbf-6ddb0562ff35.png)

- Click on ‘Clone’ and select ‘Clone in VS Code’

  ![Picture5.png](/.attachments/Picture5-b888f255-af50-4076-af18-e21fc5fe5ede.png)                
![Picture6.png](/.attachments/Picture6-30b506cc-eb99-46d9-be55-1436dfca9144.png)

- Click on ‘Open Visual Studio Code’
- When asked for a Repository location, select OSDisk (C:) > ‘DCA-Azure’ folder
- If promoted for security, select ‘Yes, I trust the authors’
- Go into the ‘Downloaded Repo’ folder and copy all its contents
- Go into OSDisk (C:) > ‘DCA-Azure’ and paste all the contents copied from ‘Downloaded Repo’
- After you paste the content, notice that the files show up in your VS code folder as well

  ![Picture7.png](/.attachments/Picture7-b4d20129-1895-4847-aeee-ef550a291568.png)

- In VSCode, click on source control and type ‘First commit’ in the message section, and press ‘Ctrl + Enter’

  ![Picture8.png](/.attachments/Picture8-28b60f14-903c-4376-9c58-763d2385c1ec.png)

- Click on ‘Sync Changes’ and select ‘OK’

  ![Picture9.png](/.attachments/Picture9-b62348c8-1cdc-4c2c-ac66-63f6c752f56d.png)

- Notice that your Azure DevOps Repo now has the ESLZ repo under ‘Files’ Section

  ![Picture10.png](/.attachments/Picture10-33d103fa-3afd-4c99-ab11-d722622c9f1b.png)

_**Setup Azure DevOps Pipeline Environment**_

- Go into Pipeline > Environment > Click on ‘New Environment’

  ![Picture11.png](/.attachments/Picture11-3c52f91a-0aac-4d33-b10c-cf656330b397.png)

- Create the following environments:

<table>
  <tr>
    <td colspan="2" style="text-align:center"><b>Environment Name</b></td>
  </tr>
  <tr>
    <td>ENV-EntScaleLZ-TearDown-dev</td>
    <td>ENV-EntScaleLZ-TearDown-prd</td>
  </tr>
  <tr>
    <td>ENV-EntScaleLZ-Deploy-dev</td>
    <td>ENV-EntScaleLZ-Deploy-prd</td>
  </tr>
  <tr>
    <td>ENV-EntScaleLZ-General-dev</td>
    <td>ENV-EntScaleLZ-General-prd</td>
  </tr>

</table>

_**Create a Personal Access Token (PAT)**_

- Go to Azure DevOps Organization
- Go to the Azure DevOps Project
- Click on the Person Icon next to your Username in the top right corner

  ![Picture1.png](/.attachments/Picture1-3884c77b-caf3-48b5-b77e-6ba0b11e4aca.png)

- Select Personal Access Tokens
- The personal access tokens splash page will pop up
- Click on “+ New Token”
- The New PAT wizard pops up:
  - Name 	=  Give it a proper name to identify what this PAT is going to be used for
  - Expiration   =  30,60, 90 days or custom (maximum time is 1 year)
  - Scopes
    - Code = Read
    - Build = Read & Execute

  ![Picture2.png](/.attachments/Picture2-4549727c-f5b5-4938-b34e-902cfbc6e9e6.png)

  - Click on create

  ![Picture3.png](/.attachments/Picture3-261d7565-f704-497f-8eaa-da3e8628ef26.png)

- Copy the PAT and place it in a secure place. Treat is as if it were a password

_**Create Variable Groups**_

- Go to Pipelines > Library and select ‘+Variable Group’
- Create 2 variable groups with following names :


<table>
  <tr style="background-color:#B0E0E6">
    <td>Variable Group Name</td>
    <td>Description</td>
    <td>Variable Key</td>
    <td>Variable Value</td>
    <td>Variable Usage Description</td>
  </tr>
  <tr>
    <td>VG-EntScaleLZ-E2E-DevTest</td>
    <td>Variable group used by the “ESLZ-Azurepipeline” Pipeline</td>
    <td> </td>
    <td> </td>
    <td> </td>
  </tr>
  <tr>
    <td> </td>
    <td> </td>
    <td>AzDo_AzOps_PipelinePullName</td>
    <td> </td>
    <td>AzOps Pull Pipeline Build Name</td>
  </tr>
  <tr>
    <td> </td>
    <td> </td>
    <td>AzDo_AzOps_PipelinePushName</td>
    <td> </td>
    <td>AzOps Push Pipeline Build Name</td>
  </tr>
  <tr>
    <td> </td>
    <td> </td>
    <td>pat_build_update</td>
    <td>******* (is a secret)</td>
    <td>Personal Access Token that has the minimum scope permissions to be able to update the pipeline builds to disable/enable them</td>
  </tr>
  

</table>

_**Create ESLZ Pipeline**_

Setup the ESLZ pipeline through following steps:

- Go to pipeline in your Azure DevOps project
- Click on ‘Create Pipeline’
- Select ‘Azure Repos Git’

  ![Picture2.png](/.attachments/Picture2-0b62489b-5aec-4eb8-8d50-4f2cf6f3af88.png)

- Select the ESLZ repo 
- Select ‘Existing Azure Pipeline YAML file’
- In pathway put ‘eslz/ESLZ-Foundation/.azure/azurepipelines-bicep.yml’, referring to the yml file in your repo

  ![Picture3.png](/.attachments/Picture3-22f915f0-e81e-47c0-8c29-85678ccb6fb6.png)

- Click on ‘Continue’ and ‘Save’
- Rename the pipeline to preferred name

##AzOps Repo Configuration

_**Get Access to AzOps Repo**_

AzOps is the repository containing the scripts which are used in obtaining the current state of the landing zone deployed and detect if any changes are done in the landing zone environment.

The details about AzOps repository is [here](https://amedeloitte.sharepoint.com/:w:/r/sites/BECUInternalInfrasTeam/_layouts/15/Doc.aspx?sourcedoc=%7B9666ABB0-6EEE-4EC1-8326-420470E1BAF3%7D&file=AzOps%20Repo%20Documentation.docx&wdLOR=c3316AA1D-E949-4CEE-A88F-97627D7727E5&action=default&mobileredirect=true).

ESLZ codebase currently sits on [‘Microsoft Technology DevOps’](https://dev.azure.com/Microsoft-Technology-Practice/DCA-Innovation%20Core/_git/dca-core-eslz) organization. Reach out to [Sunil Lamichhane](mailto:sunlamichhane@deloitte.ca) or [Amita Sujith](mailto:asujith@deloitte.com) for access to repo.

_**Setup AzOps Repo in ADO project**_

Repeat the process you did for [ESLZ Repo](#eslz-code-configuration), but this time download a different repository called DCA-AzOps Accelerator.

![Picture4.png](/.attachments/Picture4-c6e11aef-841e-4fb0-8157-df54d30193c4.png)

_**Create Variable Groups**_

- Go to Pipelines > Library and select ‘+Variable Group’

  ![Picture5.png](/.attachments/Picture5-c8e22ad4-cd05-442b-8480-ce38bb070a1d.png)

- Put name of Variable Group as ‘Credentials’
- Input following Variables:
  - ARM_CLIENT_ID = Copy AppID of the Pipeline Service Principal here
  - ARM_CLIENT_ID = Use secret created in [this step](#azure-devops-configuration)
    - Must be set to be a secret
  - ARM_SUBSCRIPTION_ID    = put any sub id that this pipeline Service Principal will have access (i.e., “management” subscription id)
  - ARM_CLIENT_ID = Tenant id can be found in AAD or AAD ServicePrincipal/AppRegistration

  ![Picture2.png](/.attachments/Picture2-9f445ac1-6e0a-4690-b680-b0a30b039ddd.png)

- Outcome should look like this:

  ![Picture7.png](/.attachments/Picture7-da764be7-c783-47f8-8fbd-361ae95018a7.png)  

_**Grant Pipeline Service Principal appropriate permissions**_

- Azure Resources
  - i. Owner on Root Management Group1.
      1. Which should be already in place if you configured the ESLZ Repository and pipelines

- Azure Active directory:
The service principal used by the Enterprise-Scale reference implementation requires Azure AD directory reader permissions to be able to discover Azure role assignments. These permissions are used to enrich data around the role assignments with additional Azure AD \

Note: The steps below require you to use an identity that is local to the Azure AD, and not Guest user account due to known restriction
  - I. Manual setup
      1. Sign into Azure Portal as a Global Administrator or elevate your access via PIM to become a Global Administrator
      2. Go to Azure Active Directory
      3. Under Manage
         - i. Select Roles and Administrators
         - ii. Select “Directory Readers” role
         - iii. Click on “+ Add assignments”
         - iv. Find the AAD Application Registration that AzOps Pipelines will be using (it can be the same one as the one that deploys the ESLZ)
         - v. Select the App Registration
         - vi. Click on Add. Ensure that it’s a permanent assignment if using AAD PIM.

  - II. Powershell setup

        $ADServicePrincipal = "< Enter the App Registration Name>"

        #verify if AzureAD module is installed and running a minimum version, if not install with 
        the latest version.
        if ((Get-InstalledModule -Name "AzureAD" -MinimumVersion 2.0.2.130 ` -ErrorAction 
        SilentlyContinue) -eq $null) {

          Write-Host "AzureAD Module does not exist" -ForegroundColor Yellow
          Install-Module -Name AzureAD -Force
          Import-Module -Name AzureAD
          Connect-AzureAD #sign in to Azure from Powershell, this will redirect you to a webbrowser 
          for authentication, if required

        }
        else {
          Write-Host "AzureAD Module exists with minimum version" -ForegroundColor Yellow
          Import-Module -Name AzureAD
          Connect-AzureAD #sign into Azure from Powershell, this will redirect you to a webbrowser 
          for authentication, if required
        }

        #Verify Service Principal and if not pick a new one.
        if (!(Get-AzureADServicePrincipal -Filter "DisplayName eq '$ADServicePrincipal'")) { 
          Write-Host "ServicePrincipal doesn't exist or is not AZOps" -ForegroundColor Red
          break
        }
        else { 
          Write-Host "$ADServicePrincipal exist" -ForegroundColor Green
          $ServicePrincipal = Get-AzureADServicePrincipal -Filter "DisplayName eq 
        '$ADServicePrincipal'"
          #Get Azure AD Directory Role
          $DirectoryRole = Get-AzureADDirectoryRole -Filter "DisplayName eq 'Directory Readers'"
          #Add service principal to Directory Role
          Add-AzureADDirectoryRoleMember -ObjectId $DirectoryRole.ObjectId -RefObjectId 
        $ServicePrincipal.ObjectId
        }

  - III. Outcome

    ![Picture8.png](/.attachments/Picture8-2d4dba54-ca2b-4e08-82c4-a5aa2b4a7ede.png)

_**Modify AzOps Repo Security Permissions**_

- Go to project settings > Select AzOps Repo> Select Security and modify the Build Service Permissions:
![Picture9.png](/.attachments/Picture9-00599d1d-1c58-4018-9f9d-3d4bda01dbaa.png)

- Setting should be set as follows:
![Picture10.png](/.attachments/Picture10-4a9b6273-4127-4211-9b78-13fb0e4e97bf.png)

_**Configure AzOps Pipelines**_

- .pipelines/push.yml
  - I. Go to Azure DevOps 
  - II. Click on pipelines
  - III. Click on New Pipeline

    ![Picture1.png](/.attachments/Picture1-573270ad-cc1a-461a-900f-60b4950ddb8e.png)

  - IV. Select “Azure Repos Git”

    ![Picture2.png](/.attachments/Picture2-e85fc34d-f556-4ab7-82fe-3479a8bec962.png)

  - V. Select “AzOps” as the repo:

    ![Picture3.png](/.attachments/Picture3-7364dbb1-cb80-47df-86cd-63d2d721afcf.png)

  - VI. Select Existing Azure Pipelines YAML File :

    ![Picture4.png](/.attachments/Picture4-29738ef4-2009-4134-af1d-e954e3665941.png)

  - VII. Select the right file, in this case the push.yml

    ![Picture5.png](/.attachments/Picture5-00a519bb-beca-4891-8d45-f0275016ca8d.png)

    - i. Click on ‘Continue’
    - ii. Save it 
    - iii. Rename it as “AzOps - Push”

  - VIII. Repeat the steps above for pipelines with the following differences

    - i. .pipelines/pull.yml
      1. Step viii = use pull.yml
      2. Step x = save it as “AzOps - Pull”
    - ii. .pipelines/validate.yaml
      1. Step viii = use validate.yml
      2. Step x = save it as “AzOps - Validate”
    - iii. Note: create them in the following order:
      1. PUSH
      2. PULL
      3. VALIDATE

  - IX. Result should be:

    ![Picture6.png](/.attachments/Picture6-a1ce9336-1b84-46b0-88c2-9b7834a55be2.png)

  - X. Repo Branch Policies setup

    - i. Go to Repo > AzOps > Branches > Main
    - ii. Click on ...

      ![Picture7.png](/.attachments/Picture7-4f38d0ab-3801-42c8-8bd6-e71245da4865.png)

    - iii. Click on Branch Policies

    - iv. Set the following:
      1. Set Squash Merge

         ![Picture8.png](/.attachments/Picture8-e50b16a3-a261-4c50-9d53-c47214f7ed4f.png)

      2. Create Build validation pipeline:

         ![Picture9.png](/.attachments/Picture9-de5135cc-4ab4-4cab-8d59-31bc9fc60598.png)

##Execution and Validation

_**Pipeline Execution**_

The pipelines configured as per the steps provided earlier are triggered in the following scenarios:

   1. Change in the code which is being checked into the “master” branch: This is configured within the yaml pipeline as part of the trigger.
   2. Manually triggering the pipeline: The pipeline can be triggered from the Azure DevOps portal by following the steps as given below.

       - a. Go to the pipelines configured by clicking on the pipelines option.

         ![Picture1.png](/.attachments/Picture1-cda4455f-1790-4a6e-bfac-7c3be5d562fa.png)

       - b. This will list all the recently run pipelines. Select the pipeline to be executed by clicking on the pipeline name.

         ![Picture2.png](/.attachments/Picture2-fffcc191-a9ed-4e98-80f4-6dae73ee3d66.png)

       - c. This will show the recent runs of the selected pipeline. Click on the “Run Pipeline” button which is highlighted to manually trigger the pipeline.

         ![Picture3.png](/.attachments/Picture3-95c93186-57e3-44fc-9ab8-6568367634ec.png)

       - d. This will display the parameters screen, where the options are available to specify the input arguments for the pipeline to be executed.

         ![Picture4.png](/.attachments/Picture4-be6f8cb4-ec47-4fc6-8601-d7e9618d540f.png)

         - i. The branch from which the pipeline is to be executed. This is useful if there are changes being done in the pipeline in a different branch and these changes are to be tested.
         - ii. Run button, which triggers the pipeline

_**Validation**_

The status of the pipeline execution can be validated by doing the below.

1. List all the recent pipeline executions by clicking on the highlighted areas of the below.

   ![Picture5.png](/.attachments/Picture5-2ea2ef87-2263-4bfe-a71f-8f5f9a0ea460.png)

   ![Picture6.png](/.attachments/Picture6-7e7b6172-9f80-4f36-8eba-fbe77e188d7d.png)

2. Select the pipeline execution which you would like to view. The summary of the overall execution status is provided below the pipeline execution, which is highlighted.

   ![Picture7.png](/.attachments/Picture7-9eec829b-7611-467f-846d-f83aa8f4c919.png)

3. The execution summary is listed with the status of each stage. Clicking on one of the stages will show the execution logs of all the stages.

   ![Picture8.png](/.attachments/Picture8-543ca195-fa7e-466d-aaae-d2e023ddcedd.png)

4. The left-hand side shows all the stages and the jobs & tasks run as part of these stages.

   ![Picture9.png](/.attachments/Picture9-323c02d2-4eed-45c7-8031-971ef4e90282.png)

5. The execution logs are available for each job / task once clicked on the respective job. These are displayed on the right side. There is an option to download the raw logs if needed for offline analysis.

   ![Picture10.png](/.attachments/Picture10-cf8082cf-78dc-4f71-818c-0d251012d25f.png)

6. In case of ARM TTK code analysis issues, the details are available within the “Tests” tab as shown below. The below example shows that of a successful run.

   ![Picture11.png](/.attachments/Picture11-55d0a0dd-1fcc-4b78-b178-0b07c2781ce8.png)

   The following example shows the issues identified by ARM TTK.

   ![Picture12.png](/.attachments/Picture12-291fe1f4-9bac-4924-a085-15b08c0399dc.png)

##Clean up - Tearing down the Environment

In case you want to tear down the environment created by execution of ESLZ pipeline, you can simply:
1. Go to you your ESLZ pipeline 
2. Click on ‘Run Pipeline’ 
3. Select ‘Stages to Run’.

![Picture13.png](/.attachments/Picture13-71444e05-8b17-40f1-bcaf-3f1a8edc81d6.png)  ![Picture14.png](/.attachments/Picture14-80d0b570-e6ae-4b36-8adf-b1f8cb6e3e7f.png)

4.	Unselect ‘Run All Stages’ 
5.	Select ESLZ_PreTearDownStage’ 
6.	Select ‘Use selected Stages’
7.	Click on ‘Run’

This will delete everything that was deployed into your Azure subscription by ESLZ pipeline.
  




