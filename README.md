# DEH-Admin-Manual
DEH Admin Manual (temporary docs)

Contents
1. [Base knowledge](https://github.com/rodekruis/DEH-Admin-Manual/blob/main/README.md#base-knowledge)
   - Terminology
   - DEH in detail
   - EspoCRM DEH Data model
   - Lifecycle Diagrams
   - Work environments
3. DEH Configuration by admin
   - Best practises
   - Add/edit/remove fields 
   - Add/edit/remove layouts
   - Translations in Flex (ask 510)
   - Translations in Espo 
   - User management (draft)
   - Roles 
   - Teams 
   - Edit opening/closing times in chatbot
   - Edit introduction message in chatbot
   - Adding/removing geographic permissions for messages/calls from foreign phone numbers
   - TaskRouter (for future)
   - Kobo form submissions to EspoCRM connection
   - Reports/dashboards
   - Configure notifications and stream
   - Changing WhatsApp profile information in Console
4. Operational admin jobs
   - Add/configure/remove users in Flex 
   - Add/configure/remove users in EspoCRM 
   - Add/remove WhatsApp templates
7. Troubleshooting guidelines
   - Steps to save browser logs
   - Steps to get to technical error screenshot
   - Known issues
   - Monitoring logs, insights and costs 
8. Other important aspects
   - [DPIA](https://github.com/rodekruis/EspoCRM-knowledge-base/wiki/Best-practices#data-responsibility) (Emie)
   - Forbidden areas

## Introduction

As an admin you possess the highest level of access rights in DEH. Deviation from the instructions outlined in this manual may result in unintended system disruptions. Depending on the extent of the changes made and the modified settings, recovery could require significant effort from a team of developers, potentially taking several days. Therefore, we urge you to exercise caution and carefully follow this manual. For every production environment, there is also a staging environment to test things on. If you are unsure about changing certain things, you could oftentimes first try this in the staging environment.  

This manual is only useful for already configured instances of DEH and it assumes that the reader has already read the [agent/supervisor manual](futurelinkhere) of DEH. This manual is not meant for setting up DEH from scratch since the GitHub documentation provides this already. 


## Base Knowledge
Why this section? The following gives the necessary background information on DEH. This could be beneficial when encountering an issue or when you want to get an overview of all the product components.

DEH is an omni-channel contact center with CRM integration for use across all Red Cross National Societies. This is a product for deploying digital services worldwide and leverages Twilio's communication capabilities mainly via [Twilio Flex](https://www.twilio.com/docs/flex) with [EspoCRM](https://www.espocrm.com/) as an underlying agent interface and CRM. The backend of this product is partly made possible via shared resources (a.o. data storage) in [Azure](https://azure.microsoft.com/en-gb/). 

The product comprises two main tangible components:

- A Twilio Flex instance providing agents with the functionality of receiving calls, Facebook Messenger, and WhatsApp via their device.
- An EspoCRM instance (embedded in Twilio Flex) allowing agents to retrieve and store relevant information relating to a person affected (PA) such as personal details, case information, interactions, etc.


### Often used Terminology
| Terminology          | Definition                                                                                                                     |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| EspoCRM              | An open source Customer Relationship Management (CRM) solution                                                                 |
| Azure                | Microsoft's cloud computing platform serving as backend                                                                        |
| Twilio Flex          | Cloud-based contact center solution provided by Twilio                                                                         |
| Twilio Console       | UI of Twilio to manage accounts, buy and configure Twilio products such as Flex                                                |
| Agent                | Role of a user in Twilio Flex who interacts with PAs                                                                           |
| Supervisor           | Role in Twilio Flex who manages agents and can see internal dashboards                                                         |
| Admin                | Role in Twilio Flex with privileged access. Admins can login via SSO and via the Console                                       |
| Person Affected (PA) | Person in need of assistance from the Red Cross, or more broadly speaking, these are the persons who engage with the Red Cross |
| Case                 | A request, pertaining to a PA, which relates to a particular service that the Red Cross can assist with                        |
| Interaction          | Details contact between a PA & a Red Cross agent. This may be a call, SMS, WhatsApp, etc.                                      |
| Contact              | The entity name for the a PA in EspoCRM                                                                                        |

### DEH in Detail
Deploying DEH consists of deploying an Azure Infrastructure (one-time-only) **and** at least one DEH instance (one instance per country). The DEH instance is here defined as the combination of Twilio Flex and an instance of EspoCRM. The Azure Infrastructure uses shared resources which are used by multiple DEH instances to send data between Twilio Flex and EspoCRM. Usually there exists only two instances of Azure Infrastructure: one for staging and one for production. A DEH instance is deployed to only one Azure Infrastructure, while an Azure Infrastructure can have more than one DEH instances deployed. The relationship between Twilio Flex and an EspoCRM instance is one-to-one.

See the below data model to see how the three relate to each other:

<img src="https://github.com/rodekruis/RED-X-DEH/assets/110089322/0ac4b642-88e5-4f9f-822c-e15bb0997eac" width="500"> <!-- This image can be found in https://miro.com/app/board/uXjVNLBVnss=/ -->

### EspoCRM DEH Data model 

EspoCRM is used as a CRM solution for storing information related to PA's, open cases, and interactions between agents & PA's. In addition to this the underlying database is used for storing extracted Twilio data utilised for additional context and reporting.

The default entity structure has been modified to suit the Product X use case. If starting from a fresh EspoCRM instance some modifications need to applied to match this configuration. The below image details the key entities and their relationships, where the 1-0 relation represents a one-to-many relationship:

![EspoCRm data model](https://github.com/rodekruis/DEH-Admin-Manual/assets/110089322/0daf4b95-642a-49c9-895e-5d3a43a55e29)

### Work environments
To effectively work with DEH as an admin, various environments are essential: 

1. Twilio Flex
2. Twilio Console
3. Azure tenant for SSO
4. EspoCRM
5. Virtual Machine

**Twilio Flex** is the contact center product from Twilio where DEH is built on top of. See it as the engine of DEH, where DEH is the custom car built around this engine. Each DEH user should be assigned to one of the three default Twilio Flex roles: agents, supervisors, and admins.  

The **Twilio Console** is the cockpit of all things related to Twilio, and change configurations of Twilio Flex. From the Console you can access DEH as an admin by opening Twilio Flex via the Console. In the Console you will also find products related to DEH which are not Flex, for example: Twilio Studio for chatbots and IVR flows (dialmenu for calls), WhatsApp templates, Twilio TaskRouter for configuring the way tasks (conversation or calls) are presented to users (like routing a specific chat or call to a specific agent) and there is the possibility to see logs and monitor errors via the Console. 

**Azure tenant for SSO** is currently a separate Azure tenant (next to the existing NLRC Azure environment) where users can be added, configured, and removed from DEH.  

**EspoCRM** is a open source CRM software that allows us to create, store and adjust contacts, cases and other information of the people we are in contact with. 

The **Virtual Machine** (VM) is a computer resource that uses software instead of a physical computer to run programs and deploy apps. EspoCRM is hosted on a VM that was specifically set-up for DEH. The VM is one of the resources on Azure. 

## DEH Configuration by Admin

> [!NOTE]
> This manual tries to refer as much as possible to other documentation. This is because EspoCRM has good [general documentation](https://docs.espocrm.com/) which is updated regularly and should be the one source of truth. If there is generic EspoCRM knowledge missing in the EspoCRM docs, there is another place to capture relevant knowledge for the humanitarian sector specifically: [EspoCRM Knowledge Base by 510](https://github.com/rodekruis/EspoCRM-knowledge-base/wiki). If something is not mentioned in the General EspoCRM docs AND not generic enough to place in the Knowledge Base, it will be described on this page.

### Best practises
When changing EspoCRM, there are certain best practises to follow. It is highly recommended to read these at least once. These can be found in the [EspoCRM Knowledge Base Best Practises Page](https://github.com/rodekruis/EspoCRM-knowledge-base/wiki/Best-practices)

### Add/Edit/Remove fields
To add, edit or remove a field, please refer to the [field section in the EspoCRM General Docs](https://docs.espocrm.com/administration/fields/), the [best practises around fields in the EspoCRM Knowledge Base](https://github.com/rodekruis/EspoCRM-knowledge-base/wiki/Best-practices#fields) and the [DPIA page](https://github.com/rodekruis/EspoCRM-knowledge-base/wiki/Best-practices#data-responsibility)

Note: there are certain fields which are impossible to remove. These are part of the DEH extension and DEH will not work properly without those fields. There is always the possibility to not use those fields in the layout.

### Add/Edit/Remove layouts
To add, edit or remove layouts, please refer to the [EspoCRM docs](https://docs.espocrm.com/administration/layout-manager/) or the [best practises around layouts in the EspoCRM Knowledge Base](https://github.com/rodekruis/EspoCRM-knowledge-base/wiki/Best-practices#layout)

### Translations in Twilio Flex
>[!NOTE]
>The below translation information is useful for Twilio Flex. If you only use EspoCRM, this is not relevant.

Translations of everything in Twilio Flex (so not the EspoCRM frame) can currently only be [configured](https://github.com/rodekruis/RED-X-DEH/blob/main/docs/deployment/translation-twilio-flex.md) by 510. The text of the translations can be of Twilio Flex can be found in the following [JSON](https://github.com/rodekruis/RED-X-DEH/tree/main/terraform/twilio/language-files). 

The current process to change translations text in Twilio Flex would be the following: 
1. Get the JSON for the language and translations you want changed. 510 can assist here.
2. Change the words/sentences that you want to change.
3. 510 will upload the edited version of the JSON. Please note: this will affect the translation of the changed language globally for every instance.

Future process: there will probably be a translation JSON per langauge hosted in a blob storage. Admins have access to edit this JSON online which means changes are immediately deployed. 

### Translations in EspoCRM
The [EspoCRM Knowledge base translations section](https://github.com/rodekruis/EspoCRM-knowledge-base/wiki/Customization#translations) describes how to change labels in different languages, both manually and automatic.

### User management 
The generic information on user management is very well described in the [EspoCRM Docs](https://docs.espocrm.com/administration/users-management/). 

>[!NOTE]
> From this point ontwards the user management information is useful for Twilio Flex. If you only use EspoCRM, this is not relevant.

DEH specific users are 'admin' and 'regular' (for supervisors and agents). DEH specific API users are 'automatic-system-deh' (Twilio API integration) and can be extended with an Kobo API integration.

### Roles
Everything about generic roles is described in the [EspoCRM Docs](https://docs.espocrm.com/administration/roles-management/). 

>[!NOTE]
> From this point onwards the user management inofrmation is useful for Twilio Flex. If you only use EspoCRM, this is not relevant.

DEH specific roles can be found [here](https://github.com/rodekruis/RED-X-DEH/blob/main/docs/deployment/default-espocrm-roles-teams.md) 

### Teams
Teams are a way to group users together. Everything generic about teams in EspoCRM is described in the [EspoCRM Docs](https://www.espocrm.com/features/teams/). 

Teams are also required if the 'Sensitive case' functionality is added to DEH. Ask 510 to assist since this is an add on to the standard solution.

>[!NOTE]
> From this point onwards, the below teams information is useful for Twilio Flex. If you only use EspoCRM, this is not relevant.
 
Teams in Twilio Flex are grouped using Skills and these are used for routing certain messages/calls to a certain team first before routing to everyone (for example). To add team in Twilio Flex, ask 510 to assist since this requires some [configuration steps](https://github.com/rodekruis/RED-X-DEH/blob/main/docs/operation/worker-skills-structure.md#project). To add a user to a team in Twilio Flex, you need the admin or supervisor role in Twilio Flex and then you can add/remove a skill for a user in Flex.

### Edit opening/closing times/days in chatbot
>[!NOTE]
> The below chatbot information is useful for Twilio Flex. If you only use EspoCRM, this is not relevant.
The opening hours/days determine if conversations/calls can end up with agents in Twilio Flex. If a message/call is received outside of opening times, it will not be shown to in Twilio Flex but it will be logged in EspoCRM as new conversation.

Steps: 
1. Go to Twilio Console
2. Studio
3. Flow
4. Select flow named '(Default) Opening Hours Subflow'
5. Click widget named ‘set_top_variables’
6. Edit ‘openTime’ or 'closeTime' in hh:mm format
7. Edit 'openDay' or 'closeDay'. 1 means Monday, 2 Tuesday, etc..
8. Edit 'timeZone' in Continent/City format
9. Save
10. ‘Publish’ the flow on the top right of the screen

### Edit introduction message in chatbot
>[!NOTE]
> The below chatbot information is useful for Twilio Flex. If you only use EspoCRM, this is not relevant.

The introduction message serves as the initial communication the receiver will receive upon engaging with the chatbot. 

Steps: 
1. Go to Twilio Console
2. Studio
3. Flow
4. Select flow of interest (for example, ‘(Default) Inbound Conversation Flow’)
5. Click widget named ‘set_top_variables’
6. Edit text in ‘welcomeMessage’ 
7. Save
8. ‘Publish’ the flow on the top right of the screen

Note: this is not an instruction to edit the introduction message for calls. That process has not been documented yet.

### Adding/removing geographic permissions for messages/calls from foreign phone numbers
>[!NOTE]
> The below phone number information is useful for Twilio Flex. If you only use EspoCRM, this is not relevant.

When using SMS and Call functionalities, Twilio only allows incoming numbers with a similar country code.  If you require the capability to make and receive calls or SMS to and from other countries, you can enable access via a setting called geo-permissions in the Twilio Console. 

Please keep in mind that: 
- Certain countries carry a higher risk of toll fraud for calls, mostly related to toll-free numbers. More information about this multi-billion dollar fraud business can be found [here](https://www.twilio.com/blog/how-to-protect-your-account-from-toll-fraud-with-voice-dialing-geo-permissions-html)
- Enabling SMS and/or Call functionality to foreign numbers involves higher costs. An estimation of costs can be found in the geopermissions settings, but it is wise to monitoring the costs regardless of whether foreign numbers are enabled or not. 

Steps for geo-permissions regarding **SMS**: 
1. Go to Twilio Console
2. Messaging
3. Settings
4. Geo permissions
5. Filter by country (or Ctrl+F) to find the country of interest
6. Save geo permissions

Steps for geo-permissions regarding **calls**: 
1. Go to Twilio Console
2. Voice
3. Settings
4. Geo permissions
5. Filter by country (or Ctrl+F) to find the country of interest
6. Save geo permissions 

### TaskRouter (for future)
>[!NOTE]
> The taskrouter information is useful for Twilio Flex. If you only use EspoCRM, this is not relevant.

### Send Kobo form submissions to EspoCRM 
There is a publicly available and free product from 510 that allows you to create your own Kobo to EspoCRM connection. Following the steps in [the knowledge base](https://github.com/rodekruis/EspoCRM-knowledge-base/wiki/Third-party-integration#integrate-espocrm-with-kobo), you can send Kobo form submissions to an entity and field of choice in EspoCRM.

### Reports 
Reports are only available when the [Advanced Pack](https://www.espocrm.com/extensions/advanced-pack/) has been purchased. This allows you to create dashboards and reports in EspoCRM. A generic explanation can be found in the [EspoCRM docs](https://docs.espocrm.com/user-guide/reports/) There are also ready-made DEH specific reports which can be found [here](https://github.com/rodekruis/RED-X-DEH/blob/main/docs/deployment/deh-and-advanced-package.md) (Emie)

Install notifications and [stream](https://docs.espocrm.com/user-guide/stream/#notifications)

### Changing WhatsApp profile information in Console
>[!NOTE]
> The below info on WhatsApp profiles is useful for Twilio Flex. If you only use EspoCRM, this is not relevant.
 
This is the information showcased on the WhatsApp number's profile. It includes a profile picture and a concise business description. Additionally, it can be expanded to include hyperlinks to the website, email, and address. 

Steps: 
1. Go to Twilio Console (make sure you are in the correct account)
2. Messaging
3. WhatsApp senders
4. Click on WhatsApp number of interest
5. In the section ‘Business profile information’, you can edit information
6. Update WhatsApp sender 

## Operational admin jobs

### Add/configure/remove users in Flex 

Note: in the future this will be done differently.

### Add/configure/remove users in EspoCRM 


### Add/remove WhatsApp templates 
>[!NOTE]
> The below info on WhatsApp templates is useful for Twilio Flex. If you only use EspoCRM, this is not relevant.
 
Whatsapp templates serve two primary purposes: for agents sending messages in conversations that have been inactive for 24 hours (WhatsApp/Meta policy does not allow an agent to send self-written messages after this period), or when a standardized message is to be send out.  

Steps: 
1. Go to Twilio Console (make sure you are in the correct account)
2. Messaging
3. Content Editor
4. Create new
5. Template Name should be something logic including language (for example: start_conversation_dutch)
6. Select Template Language
7. Select Content Type should be ‘Text’
8. Click Create
9. In Body, type your template (please keep in mind [WhatsApp template rules](https://developers.facebook.com/docs/whatsapp/message-templates/guidelines/))
10. Click ‘Save and submit for WhatsApp approval’
11. Wait until approved (usually under a minute and you will probably get a confirmation email)
12. Add WhatsApp template as Canned Response in EspoCRM so that every user can find and use this
13. Refresh Twilio Flex and then the templates are visible in the Canned Responses

## Troubleshooting guidelines
The following websites are great resources to troubleshoot different areas: 

- Twilio Products [Current Incidents Status](https://status.twilio.com/) (If Twilio Products have an outage, they will communicate all updates via this website)
- [User Network Test](https://networktest.twilio.com/) (If user experiences issues, this is an easy tool to run to see if this relates to network problems)
- A tutorial from Twilio to debug Twilio Flex can be found [here](https://www.twilio.com/blog/collect-browser-logs-debug-flex)
- [Debugging Tools](https://www.twilio.com/docs/sms/troubleshooting/debugging-tools) for Twilio Console errors
- [Debugging your Twilio Application](https://www.twilio.com/docs/usage/troubleshooting/debugging-your-application) (very technical)
- [Error Handling and Diagnostics](https://www.twilio.com/docs/conversations/error-handling-diagnostics) (very technical) 

### Steps to save browser logs

Providing your browser logs can be helpful for the developer (Zing) in understanding and resolving the issue. You or your agent can save the logs by following the steps below. Be sure to include the logs, after removing any sensitive data,  in your conversation with the developer for further assistance. 

Steps: 
1. Right-click in the browser window or tab and select Inspect.
2. Click the Network tab in the panel that appears.
3. Recreate the steps that caused the issue to show, once the error has happened again.
4. Click the download button ("Export HAR" appears when you hold the pointer over it).
5. Name the file.
6. Click Save and send it to help@zing.dev  (Note: the HAR file will have the full contents of the request, so if there is any sensitive data in that request which should not be shown to external parties, you should open the file in a text editor and scrub/remove any sensitive values before sending) 

### Steps to get to technical error screenshot
If you or your agent encounter an issue that is not already known, or cannot be resolved with the information provided in this manual, nor through your expertise, take the following steps: Capture a screenshot of the technical error and forward it to our developer, Zing. They will assist you in addressing the issue. 

Steps: 
1. Right click on the internet page and search for 'Inspect'
2. Click inspect.
3. In the top row you will see the word 'Console' next to 'Elements' and 'Sources", click on this 'Console'
4. On the left bar you will see 'error', click on this.
5. Make a screenshot
6. Save the screenshot and send it to help@zing.dev


### Known issues
This is a list of currently known issues and how to mitigate those: 
1. ....

### Bug reporting process
[see redline admin manual]

### Monitoring logs, insights and costs 
> The below monitoring logs, insights and costs are useful for Twilio Flex. If you only use EspoCRM, this is not relevant.
> 
Steps to find **error logs** in Twilio Flex: 
1. Go to the Twilio Console and make sure you are in the right account
2. Monitor on the top left bar (Ctrl + F is your friend)
3. Logs
4. Errors
5. Error logs 

Steps to find **messaging logs**: 
1. Go to the Twilio Console and make sure you are in the right account
2. Monitor on the top left bar (Ctrl + F is your friend)
3. Logs
4. Messaging (note: during configuration there could be a setting configured which removes message logs after a period of time because of GDPR requirements) 

Steps to find **messaging insights** (aggregated messaging logs): 
1. Go to the Twilio Console and make sure you are in the right account
2. Monitor on the top left bar (Ctrl + F is your friend)
3. Insights
4. ‘Messages’ AND ‘Message Deliverability’ 

Steps to find **call logs**: 
1. Go to the Twilio Console and make sure you are in the right account
2. Monitor on the top left bar (Ctrl + F is your friend)
3. Logs
4. Calls (note: during configuration there could be a setting configured which removes call logs after a period of time because of GDPR requirements) 

Steps to find **call insights** (aggregated call logs): 
1. Go to the Twilio Console and make sure you are in the right account
2. Monitor on the top left bar (Ctrl + F is your friend)
3. Insights
4. Calls 

Steps to find currently **active conversations** (collection of messages) logs: 
1. Go to the Twilio Console and make sure you are in the right account
2. Monitor on the top left bar (Ctrl + F is your friend)
3. Logs
4. Conversation (note: during configuration there could be a setting configured which removes conversation logs after a period of time because of GDPR requirements) 

Steps to find overview of **costs**: 
1. Go to the Twilio Console and make sure you are in the right account
2. Monitor on the top left bar (Ctrl + F is your friend)
3. Insights
4. Billing Usage 


## Other important aspects

### DPIA
   - [DPIA](https://github.com/rodekruis/EspoCRM-knowledge-base/wiki/Best-practices#data-responsibility) (Emie)

### Lifecycle Diagrams: DEH explained in a flowchart
>[NOTE!]
> The below lifecycle diagrams are useful for Twilio Flex. If you only use EspoCRM, this is not relevant.

These flowcharts represent all the tiny steps of DEH that take place from start: the moment a call or message was send in, to end. An admin can use the flowcharts to further understand DEH. The steps are different for incoming messasges (conversation), calls and outbound messages (outbound conversation). We refer to the images below.

#### Conversation Lifecycle
<img src="https://github.com/rodekruis/DEH-Admin-Manual/assets/110089322/b0937075-20b4-4b02-8d85-61f06e3fd8d8" width="600">

#### Call lifecycle
<img src="https://github.com/rodekruis/DEH-Admin-Manual/assets/110089322/9e4e661e-a936-4975-b01e-79b3385dda37" width="800">

#### Outbound Conversation Lifecycle
<img src="https://github.com/rodekruis/DEH-Admin-Manual/assets/110089322/1ffd209c-1434-4033-a264-a05fa66f93af" width="600">

> [!CAUTION]
>### Forbidden areas
>There are certain areas which can have devastating consequences when changed. Only an admin has access to these areas, but should not touch this. 
>
>This is a list of potential destructive actions in EspoCRM to be aware of:
>1. Changing the global UI (Administration>User Interface). This affects UI for all users of EspoCRM.
>2. Remove EspoCRM Single Sign On (SSO) settings (Administration>Authentication). This will disable every user from using SSO to login in EspoCRM.
>3. Uninstall the EspoCRM Flex DEH Extension (Adminstration>Extensions). This will remove all DEH relevant extensions and DEH related data will not be visible anymore in EspoCRM.
>4. Uninstall the Advanced Pack Extension (Administration>Extensions). Only relevant if this Advanced Package is installed. Uninstalling it will remove all the reports (dashboards) and flowcharts/workflows.
>5. Upgrade EspoCRM (Administration>Upgrade). DEH is currently made for EspoCRM version 8.1.0. A newer version will not ensure compatibility with DEH.
>6. Certain role settings for users (Administration>Roles). The role permissions 'Export Permission' and 'Mass Update Permission' are very powerfull permissions which should be considered carefully before assigning to a role. This would allow a user to quickly export large volumes of personal data or change large volumes of records in EspoCRM.
>7. Disabling or removing an entity (Administration>Entity Manager>[Entity]>Edit).
>8. Removing relationships between entities (Administration>Entity Manager>[Entity]>Relationships).
>9. Removing Email configuration settings (Administration>Outbound Emails). Removing this will stop the email notifications to users and password reset emails.
>10. Removing all other users (and admins) so that no one has access anymore.

>[!NOTE]
> From this point onwards, the forbidden areas are useful for Twilio Flex. If you only use EspoCRM, this is not relevant.

> [!CAUTION]
>12. Remove the API role or API user called automatic-system-deh (Administration>Roles and Administration>API Users). This will break the connection between Twilio Flex and EspoCRM and no data will be transferred anymore.
>
>This is a list of potential destructive actions in Twilio to be aware of:
>1. Remove Active Phone Numbers
>2. Remove SSO setting for Twilio Flex (Flex>Manage>Single sign-on). This will disable every user from logging in to Twilio Flex.
>3. Disconnect Studio Flows (chatbot/IVR) with Twilio Flex (Flex>Manage>Messaging>Filter by Address type)
>4. Removing TaskRouter settings
>5. Removing Studio flows
>6. Removing WhatsApp sender (Messaging>Senders>WhatsApp senders)
>7. Removing Facebook Messenger (Channels>Facebook Messenger)
>8. Removing Event Streams
>9. Removing API keys & tokens
