# Enterprise Messaging Configuration

#### Setup
  * create enterprise messaging instance in [mta.yaml](https://github.com/pp0001/Documents/blob/master/EM%20service%20instance.png)
  
  	> ename: cannot have a duplicate name on the same global account
	>
	> namespace: case sensitive
	>
	> Enterprise Messaging offers the plans dev, lite and default. We use the default plan for production use.
	
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
    
   	1. check enterprise message is ready, [Readiness Check](https://help.sap.com/doc/75c9efd00fc14183abc4c613490c53f4/Cloud/en-US/rest-management-messaging.html#_readinesscheckusingget)
   
	2. [queue creation](https://help.sap.com/doc/75c9efd00fc14183abc4c613490c53f4/Cloud/en-US/rest-management-messaging.html#_putqueue)
		* The URI for this put request needs to be specifically encoded and the slashes to be converted into unicode characters '%2FA'.
		* accessType: EXCLUSIVE, NON_EXCLUSIVE (default: NON_EXCLUSIVE)	

		> EXCLUSIVE: Only one consumer can receive messages at a specific point in time. The consumer holds an exclusive 			> connection to the queue at that specific moment. If this consumer disconnects from the queue, then the next consumer 			> can connect to the queue and start receiving messages. An exclusive queue always delivers messages in the order they 			> are received.
		> 
		> NON_EXCLUSIVE: Multiple consumers can connect to the queue at a specific point in time. The messages are delivered in 		> a round-robin fashion. This approach enables load balancing. However, if for some reason the connection fails, then 			> the messages are delivered to another customer. In this way, messages can be delivered out of order.
	
	3. [subscribe queue to topic](https://help.sap.com/doc/75c9efd00fc14183abc4c613490c53f4/Cloud/en-US/rest-management-messaging.html#_putqueuesubscription_1).
        
   APIs and models can get from Messaging management REST API in the Reference Link.
  
#### Onboarding
> If you want to enable customer to subscribe your service which contains enterprise messaging service instance, you should create a ticket to add provider tenant to enterprise messaging white list. 
Please follow this ticket link: https://sapjira.wdf.sap.corp/browse/COCOMEF-417

>	DefaultSubscribeExit registers the app with Enterprise Messaging's REST gateway web hook, to enable it to receive messages from the tenant's event bus.

> In reverse, DefaultUnsubscribeExit unregisters the web hook.

> DefaultDependencyExit adds Enterprise Messaging to the list of required backing services. This ensures that Enterprise Messaging receives an onboarding callback that instantiates the tenant's event bus, in case this has not happened, yet.

#### Reference Link
[SAP Help]( https://help.sap.com/viewer/product/SAP_ENTERPRISE_MESSAGING/Cloud/en-US?task=discover_task)

[Wiki](https://wiki.wdf.sap.corp/wiki/display/MDM/SCP+Platform+-+Enterprise+messaging+service)

[Guidelines for Eventing in the GRC Platform](https://wiki.wdf.sap.corp/wiki/display/GRCPLAT/Guidelines+for+Eventing+in+the+GRC+Platform)

[Enterprise messaging Source Code Github](https://github.wdf.sap.corp/enterprise-messaging/xbem-sb/tree/master/sb-app)

[Messaging management REST API](https://help.sap.com/doc/75c9efd00fc14183abc4c613490c53f4/Cloud/en-US/rest-management-messaging.html#_overview)

[REST endpoints of Enterprise Messaging - Messaging REST](https://help.sap.com/doc/3dfdf81b17b744ea921ce7ad464d1bd7/Cloud/en-US/messagingrest-api-spec.html)

[Guidelines for Eventing in the GRC Platform](https://wiki.wdf.sap.corp/wiki/pages/viewpage.action?spaceKey=GRCPLAT&title=Guidelines+for+Eventing+in+the+GRC+Platform)

[AutomatedProcedures Project](https://github.wdf.sap.corp/GRC-CH/AutomatedProcedures)

[Control Library Project](https://github.wdf.sap.corp/grchcpcf/control-library)

[Enterprise messaging team ticket link](https://sapjira.wdf.sap.corp/projects/COCOMEF/issues/COCOMEF-496?filter=allopenissues)

#### Issue List
  * Messages from other services cannot be consumed because the messages' content-type is 'application/octet-stream': You should add a message converter bean in you service
  ```
  @Configuration
  public class MessageConverterConfiguration {
      @Bean
      public MappingJackson2HttpMessageConverter customizedJacksonMessageConverter() {
	  MappingJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter();
	  converter.setSupportedMediaTypes(
	  Arrays.asList(MediaType.APPLICATION_JSON, new MediaType("application", "*+json"),
	      MediaType.APPLICATION_OCTET_STREAM));
	  return converter;
	  }
   }
   ```
  
  * Webhook Subscription Issue, in PaaS tenant cannot subscribe webhook
    1. MTA: Create instance of backednd service broker named as “enterprise-messaging-client”
    2. bind this instance to srv module
    3. MTA: Create new saas-registry instance, use xsappname from "enterprise-messaging-client", provide new callback urls to differentiate between "real" subscription to automated procedures and this workaround subscription
    4. Implement new callbacks for new url
    5. In Webhook subscription object in code level, read client id and secret from binding to "enterprise-messaging-client"
    
