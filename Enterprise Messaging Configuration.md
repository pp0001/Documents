# Enterprise Messaging Configuration

#### Setup
  * create enterprise messaging instance in mta.yaml
  * add dependency
  * create MessagingServiceFactory and ConnectionFactory, in this step should consider multi-tenant situation
  * create publish class 
  * prepare subscribe enterprise message method
    * check enterprise message is ready, Readiness Check.
    * create queue.
    * create webhook.
    * subscribe queue to topic.
    
	APIs and models can get from Messaging management REST API.
  
#### Onboarding
> If you want to enable customer to subscribe your service which contains enterprise messaging service instance, you should create a ticket to add provider tenant to enterprise messaging white list. 
Please follow this ticket link: https://sapjira.wdf.sap.corp/browse/COCOMEF-417

>	DefaultSubscribeExit registers the app with Enterprise Messaging's REST gateway web hook, to enable it to receive messages from the tenant's event bus.

> In reverse, DefaultUnsubscribeExit unregisters the web hook.

> DefaultDependencyExit adds Enterprise Messaging to the list of required backing services. This ensures that Enterprise Messaging receives an onboarding callback that instantiates the tenant's event bus, in case this has not happened, yet.

#### Reference Link
[SAP Help]( https://help.sap.com/viewer/product/SAP_ENTERPRISE_MESSAGING/Cloud/en-US?task=discover_task)

[Wiki](https://wiki.wdf.sap.corp/wiki/display/MDM/SCP+Platform+-+Enterprise+messaging+service)

[Enterprise messaging Source Code Github](https://github.wdf.sap.corp/enterprise-messaging/xbem-sb/tree/master/sb-app)

[Messaging management REST API](https://help.sap.com/doc/75c9efd00fc14183abc4c613490c53f4/Cloud/en-US/rest-management-messaging.html#_overview)

[REST endpoints of Enterprise Messaging - Messaging REST](https://help.sap.com/doc/3dfdf81b17b744ea921ce7ad464d1bd7/Cloud/en-US/messagingrest-api-spec.html)

[Guidelines for Eventing in the GRC Platform](https://wiki.wdf.sap.corp/wiki/pages/viewpage.action?spaceKey=GRCPLAT&title=Guidelines+for+Eventing+in+the+GRC+Platform)

[AutomatedProcedures Project](https://github.wdf.sap.corp/GRC-CH/AutomatedProcedures)

[Control Library Project](https://github.wdf.sap.corp/grchcpcf/control-library)

[Enterprise messaging team ticket link](https://sapjira.wdf.sap.corp/projects/COCOMEF/issues/COCOMEF-496?filter=allopenissues)

#### Issue List
  * messages' content-type is 'application/octet-stream': You should add a message converter bean in you service
  
  * Messages from other services cannot be consumed
    * bind service broker to backend service
    * registry service broker, redefined callback controller
    * when create webhook add service broker's client id and client secret
