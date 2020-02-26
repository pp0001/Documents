# Enterprise Messaging Configuration

#### Setup
  * create enterprise messaging instance in [mta.yaml](https://github.com/pp0001/Documents/blob/master/EM%20service%20instance.png)
  
  	> ename: cannot have a duplicate name on the same global account
	>
	> namespace: case sensitive
	
  ```
  name: controls-em
  type: org.cloudfoundry.managed-service
  active: true
  parameters:
    service: enterprise-messaging
    service-plan: default
    namespace: "sap/grc/ctrldev"
    config:
      instanceType: "reuse"
      emname: controls-em-${space}
      namespace: "sap/grc/ctrldev"
      version: 1.1.0
      options:
        management: true
        messagingrest: true
        messaging: true
      rules:
        queueRules:
          publishFilter:
            - "${namespace}/*"
          subscribeFilter:
            - "${namespace}/*"
        topicRules:
          publishFilter:
            - "${namespace}/*"
            - "sap/grc/ap/*"
            - "SAP/GRC/Ctrl/*"
          subscribeFilter:
            - "${namespace}/*"
            - "sap/grc/ap/*"
  ```
  * add [dependency](https://github.com/pp0001/Documents/blob/master/EM%20dependency.png)
  
  	> If meet localcloudconfig issue after deploy to cloud or other strange issue, check dependency maybe it's dependency conflict issues.
  * create MessagingServiceFactory and ConnectionFactory, in this step should consider multi-tenant situation
  * create publish class 
  * prepare subscribe enterprise message method
  ![em process](https://github.com/pp0001/Documents/blob/master/EM%20process.png)
    * check enterprise message is ready, Readiness Check.
        * Endpoint: "/hub/rest/api/v1/management/messaging/readinessCheck"
    * create queue.
        * Endpoint: "/hub/rest/api/v1/management/messaging/queues/"
        * accessType: EXCLUSIVE, NON_EXCLUSIVE (default: NON_EXCLUSIVE)
		> EXCLUSIVE: Only one consumer can receive messages at a specific point in time. The consumer holds an exclusive 			> connection to the queue at that specific moment. If this consumer disconnects from the queue, then the next consumer 			> can connect to the queue and start receiving messages. An exclusive queue always delivers messages in the order they 			> are received.
		> 
		> NON_EXCLUSIVE: Multiple consumers can connect to the queue at a specific point in time. The messages are delivered in 		> a round-robin fashion. This approach enables load balancing. However, if for some reason the connection fails, then 			> the messages are delivered to another customer. In this way, messages can be delivered out of order.
		
    * subscribe queue to topic.
        * subscription path: "https://enterprise-messaging-pubsub.cfapps.sap.hana.ondemand.com/messagingrest/v1/subscriptions"
        
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
