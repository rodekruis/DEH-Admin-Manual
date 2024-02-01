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
   - [Best practises](https://github.com/rodekruis/EspoCRM-knowledge-base/wiki/Best-practices)
   - [Add/edit/remove fields](https://docs.espocrm.com/administration/fields/) 
   - [Add/edit/remove layouts](https://docs.espocrm.com/administration/layout-manager/)
   - Translations in Flex (ask 510) (1. current implementation, 2. future situation which jsons in Azure blob which can be edited by NS)
   - Translations in Espo (two step approach: 1. [automatically translate ](https://espo-translate.azurewebsites.net/docs#/default/translate_translate_post) 2. edit translations in label manager)
   - User management 
   - Roles (1. [explain how roles work](https://docs.espocrm.com/administration/roles-management/) 2. explain DEH specific roles)
   - Teams (1. explain how [teams](https://www.espocrm.com/features/teams/) work [knowledge base])
   - Edit opening/closing times in chatbot
   - Edit introduction message in chatbot
   - Adding/removing geographic permissions for messages/calls from foreign phone numbers
   - TaskRouter (for future)
   - [Kobo to EspoCRM connection](https://kobo-connect.azurewebsites.net/docs#/default/kobo_to_espocrm_kobo_to_espocrm_post) 
   - Reports (1.[ general explanation](https://docs.espocrm.com/user-guide/reports/) 2. [DEH specific reports](https://github.com/rodekruis/RED-X-DEH/blob/main/docs/deployment/deh-and-advanced-package.md) (Emie)
   - Install notifications and [stream](https://docs.espocrm.com/user-guide/stream/#notifications)
   - Changing WhatsApp profile information in Console
4. Operational admin jobs
   - Add/configure/remove users in Flex (1. current situation 2. future situation when unified user management is ready)
   - Add/configure/remove users in EspoCRM (1. current situation)
   - Add/remove WhatsApp templates (add in console, add in Espo as canned response)
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

An omni-channel contact center with CRM integration for use across all Red Cross National Societies. This is a product for deploying digital services worldwide and leverages Twilio's communication capabilities mainly via [Twilio Flex](https://www.twilio.com/docs/flex) with [EspoCRM](https://www.espocrm.com/) as an underlying data storage and agent interface. The backend of this product is partly made possible via shared resources in [Azure](https://azure.microsoft.com/en-gb/). 

The product comprises two main tangible components:

- A Twilio Flex instance providing agents with functionality for receiving calls, Facebook Messenger, and WhatsApp via their device.
- An EspoCRM instance (embedded in Twilio Flex) allowing agents to retrieve and store relevant information relating to a person affected (PA) such as personal details, case information, interactions, etc.


### Often used Terminology
| Terminology          | Definition                                                                                                                     |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| EspoCRM              | An open source CRM solution                                                                                                    |
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

#### EspoCRM DEH Data model 

EspoCRM is used as a CRM solution for storing information related to PA's, open cases, and interactions between agents & PA's. In addition to this the underlying database is used for storing extracted Twilio data utilised for additional context and reporting.

The default entity structure has been modified to suit the Product X use case. If starting from a fresh EspoCRM instance some modifications need to applied to match this configuration. The below image details the key entities and their relationships:

![EspoCRm data model](https://github.com/rodekruis/DEH-Admin-Manual/assets/110089322/0daf4b95-642a-49c9-895e-5d3a43a55e29)

#### Lifecycle Diagrams

##### Conversation Lifecycle
![image](https://github.com/rodekruis/DEH-Admin-Manual/assets/110089322/b0937075-20b4-4b02-8d85-61f06e3fd8d8)

##### Call lifecycle
![image](https://github.com/rodekruis/DEH-Admin-Manual/assets/110089322/9e4e661e-a936-4975-b01e-79b3385dda37)

##### Outbound Conversation Lifecycle
![image](https://github.com/rodekruis/DEH-Admin-Manual/assets/110089322/1ffd209c-1434-4033-a264-a05fa66f93af)

### Work environments
To effectively work with RedLine as an admin, various environments are essential: 

1. Twilio Flex
2. Twilio Console
3. Azure tenant for SSO
4. EspoCRM
5. Virtual Machine

**Twilio Flex** is the contact center product from Twilio where RedLine is built on top of. See it as the engine of RedLine, where RedLine is the custom car built around this engine.. Each RedLine user should be assigned to one of the three default Twilio Flex roles: agents, supervisors, and admins.  

The **Twilio Console** is the cockpit of all things related to Twilio. From the Console you can access RedLine as an admin by opening Twilio Flex via the Console. In the Console you will also find products related to RedLine which are not Flex, for example: Twilio Studio for chatbots and IVR flows, WhatsApp templates, Twilio TaskRouter for configuring the way tasks (conversation or calls) are presented to users and there is the possibility to see logs and monitor errors via the Console. 

**Azure tenant for SSO** is currently a separate Azure tenant (next to the existing NLRC Azure environment) where users can be added, configured, and removed from RedLine.  

**EspoCRM** 

**Virtual Machine** 

### DEH Configuration by Admin

> [!NOTE]
> This manual tries to refer as much to other documentation. This is because EspoCRM has good [general documentation](https://docs.espocrm.com/) which is updated regularly and should be the one source of truth. If there is generic EspoCRM knowledge missing in the EspoCRM docs, there is another place to capture relevant knowledge for the humanitarian sector specifically: [EspoCRM Knowledge Base by 510](https://github.com/rodekruis/EspoCRM-knowledge-base/wiki). If something is not mentioned in the General EspoCRM docs AND not generic enough to place in the Knowledge Base, it will be described on this page.

#### Best practises
When changing EspoCRM, there are certain best practises to follow. It is highly recommended to read these at least once. These can be found in the [EspoCRM Knowledge Base Best Practises Page](https://github.com/rodekruis/EspoCRM-knowledge-base/wiki/Best-practices)

#### Add/Edit/Remove fields
To add, edit or remove a field, please refer to the [field section in the EspoCRM General Docs](https://docs.espocrm.com/administration/fields/), the [best practises around fields in the EspoCRM Knowledge Base](https://github.com/rodekruis/EspoCRM-knowledge-base/wiki/Best-practices#fields) and the [DPIA page](https://github.com/rodekruis/EspoCRM-knowledge-base/wiki/Best-practices#data-responsibility)

Note: there are certain fields which are impossible to remove. These are part of the DEH extension and DEH will not work properly without those fields. There is always the possibility to not use those fields in the layout.

#### Add/Edit/Remove layouts
To add, edit or remove layouts, please refer to the [EspoCRM docs](https://docs.espocrm.com/administration/layout-manager/) or the [best practises around layouts in the EspoCRM Knowledge Base](https://github.com/rodekruis/EspoCRM-knowledge-base/wiki/Best-practices#layout)

#### Translations in Twilio Flex
Translations of everything in Twilio Flex (so not the EspoCRM part) can currently only be [configured](https://github.com/rodekruis/RED-X-DEH/blob/main/docs/deployment/translation-twilio-flex.md) by 510. The text of the translations can be of Twilio Flex can be found in the following [JSON](https://github.com/rodekruis/RED-X-DEH/tree/main/terraform/twilio/language-files). 

The current process to change translations text in Twilio Flex would be the following: 
1. Get the JSON for the language and translations you want changed. 510 can assist here.
2. Change the words/sentences that you want to change.
3. 510 will upload the edited version of the JSON. Please note: this will affect the translation of the changed language globally for every instance.

Future process: there will probably be a translation JSON per langauge hosted in a blob storage. Admins have access to edit this JSON online which means changes are immediately deployed. 

#### Translations in EspoCRM
The [EspoCRM Knowledge base translations section](https://github.com/rodekruis/EspoCRM-knowledge-base/wiki/Customization#translations) describes how to change labels in different languages, both manually and automatic.

#### User management 

#### Roles
Everything about generic roles is described in the [EspoCRM Docs](https://docs.espocrm.com/administration/roles-management/). DEH specific roles can be found [here](https://github.com/rodekruis/RED-X-DEH/blob/main/docs/deployment/default-espocrm-roles-teams.md) 

#### Teams
Teams are a way to group users together. Everything generic about teams in EspoCRM is described in the [EspoCRM Docs](https://www.espocrm.com/features/teams/). 

Teams in Twilio Flex are grouped using Skills and these are used for routing certain messages/calls to a certain team first before routing to everyone (for example). To add team in Twilio Flex, ask 510 to assist since this requires some [configuration steps](https://github.com/rodekruis/RED-X-DEH/blob/main/docs/operation/worker-skills-structure.md#project). To add a user to a team in Twilio Flex, you need the admin or supervisor role in Twilio Flex and then you can add/remove a skill for a user in Flex.

#### Edit opening/closing times in chatbot

#### Edit introduction message in chatbot

#### Adding/removing geographic permissions for messages/calls from foreign phone numbers


- TaskRouter (for future)
- [Kobo to EspoCRM connection](https://kobo-connect.azurewebsites.net/docs#/default/kobo_to_espocrm_kobo_to_espocrm_post) 
- Reports (1.[ general explanation](https://docs.espocrm.com/user-guide/reports/) 2. [DEH specific reports](https://github.com/rodekruis/RED-X-DEH/blob/main/docs/deployment/deh-and-advanced-package.md) (Emie)
- Install notifications and [stream](https://docs.espocrm.com/user-guide/stream/#notifications)
- Changing WhatsApp profile information in Console

### Other important aspects

#### DPIA
   - [DPIA](https://github.com/rodekruis/EspoCRM-knowledge-base/wiki/Best-practices#data-responsibility) (Emie)


#### Forbidden areas
There are certain areas which can have devastating consequences when changed. Only an admin has access to these areas, but should not touch this. This is a list of potential destructive actions to be aware of:
1. Changing the global UI (Administration>User Interface). This affects UI for all users of EspoCRM.
2. Uninstall the EspoCRM Flex DEH Extension (Adminstration>Extensions). This will remove all DEH relevant extensions and DEH related data will not be visible anymore in EspoCRM.
3. Uninstall the Advanced Pack Extension (Administration>Extensions). Only relevant if this Advanced Package is installed. Uninstalling it will remove all the reports (dashboards) and flowcharts/workflows.
4. Upgrade EspoCRM (Administration>Upgrade). DEH is currently made for EspoCRM version 8.1.0. A newer version will not ensure compatibility with DEH.
5. Remove the API role or API user called automatic-system-deh (Administration>Roles and Administration>API Users). This will break the connection between Twilio Flex and EspoCRM and no data will be transferred anymore.
6. Certain role settings for users (Administration>Roles). The role permissions 'Export Permission' and 'Mass Update Permission' are very powerfull permissions which should be considered carefully before assigning to a role. This would allow a user to quickly export large volumes of personal data or change large volumes of records in EspoCRM.
7. Disabling or removing an entity (Administration>Entity Manager>[Entity]>Edit).
8. Removing relationships between entities (Administration>Entity Manager>[Entity]>Relationships).
9. Removing Email configuration settings (Administration>Outbound Emails). Removing this will stop the email notifications to users and password reset emails.
