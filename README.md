# DEH-Admin-Manual
DEH Admin Manual (temporary docs)

Contents
1. Base knowledge
   - Terminology
   - DEH in detail
   - EspoCRM DEH Data model
   - Lifecycle Diagrams
3. DEH Configuration by admin
   - [Best practises](https://github.com/rodekruis/EspoCRM-knowledge-base/wiki/Best-practices)
   - Add/edit/remove fields
   - Add/edit/remove layouts
   - Translations in Flex (ask 510)
   - Translations in Espo
   - Roles
   - Edit opening/closing times in chatbot
   - Edit introduction message in chatbot
   - Adding/removing geographic permissions for messages/calls from foreign phone numbers
   - TaskRouter (for future)
   - Kobo to EspoCRM connection
   - Reports
   - Install notifications
   - Changing WhatsApp profile information
4. Operational admin jobs
   - Add/configure/remove users in Flex
   - Add/configure/remove users in EspoCRM
   - Add/remove WhatsApp templates
   - 
7. Troubleshooting guidelines
   - Steps to save browser logs
   - Steps to get to technical error screenshot
   - Known issues
   - Monitoring logs, insights and costs 
8. Other important aspects
   - DPIA

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

### 
