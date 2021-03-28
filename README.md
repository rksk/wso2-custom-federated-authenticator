# Writing a Custom Federated Authenticator

A custom federated authenticator can be written to authenticate a user with an external system. 
So the external system can be any Identity provider such as Facebook, Twitter, Google and Yahoo. 
It is possible to use the extension points available in the WSO2 Identity Server to create custom 
federated authenticators.

### Writing a custom federated authenticator


1. First create a maven project for the custom federated authenticator. Refer the pom.xml file used for the sample 
   custom federated authenticator.
2. Refer the [service component class](https://github.com/pamodaaw/sample-local-authenticator/blob/main/src/main/java/org/wso2/carbon/auth/local/sample/internal/SampleLocalAuthenticatorServiceComponent.java) 
   as well since the authenticator is written as an OSGI service to deploy in the WSO2 Identity Server and register 
   it as a federated authenticator
3. The [custom federated authenticator](https://github.com/pamodaaw/sample-local-authenticator/blob/main/src/main/java/org/wso2/carbon/auth/local/sample/SampleLocalAuthenticator.java) 
   should be written by extending the [AbstractApplicationAuthenticator](https://github.com/wso2/carbon-identity-framework/blob/v5.18.187/components/authentication-framework/org.wso2.carbon.identity.application.authentication.framework/src/main/java/org/wso2/carbon/identity/application/authentication/framework/AbstractApplicationAuthenticator.java) class and implementing the[ FederatedApplicationAuthenticator](https://github.com/wso2/carbon-identity-framework/blob/v5.18.187/components/authentication-framework/org.wso2.carbon.identity.application.authentication.framework/src/main/java/org/wso2/carbon/identity/application/authentication/framework/LocalApplicationAuthenticator.java) class.
4. You can find a custom federated authenticator from here for your reference

The important methods in the AbstractApplicationAuthenticator class, and the FederatedApplicationAuthenticator interface are listed as follows.

*   **public String getName()**

Return the name of the authenticator

*   **public String getFriendlyName()**

Returns the display name for the custom federated authenticator. In this sample we are using custom-federated-authenticator

*   **public String getContextIdentifier(HttpServletRequest request)**

Returns a unique identifier that will map the authentication request and the response. The value returned by the invocation of authentication request and the response should be the same.

*   **public boolean canHandle(HttpServletRequest request)** -

Specifies whether this authenticator can handle the authentication response.

*   **protected void initiateAuthenticationRequest(HttpServletRequest request,HttpServletResponse response, AuthenticationContext context)**

Redirects the user to the login page in order to authenticate and in this sample, the user is redirected to the login page of the application which is configured in the partner identity server which acts as the external service.

*   **protected void processAuthenticationResponse(HttpServletRequest request,HttpServletResponse response, AuthenticationContext context)**

Implements the logic of the custom federated authenticator.

### Deploy the custom federated authenticator in WSO2 IS

1. Once the implementation is done, navigate to the root of your project and run the following command to compile the service
2. Copy the compiled jar file insider _<Custom-federated-authenticator>/target._
3. Copy the jar file **org.wso2.carbon.identity.custom.federated.authenticator-1.0.0.jar** file to the _<IS_HOME>/repository/components/dropins._

### Configure the partner identity server

In this sample the partner identity server acts as the external system. 
Therefore, that partner identity server will be running on the same machine in a different port 
by adding the following config to the deployment.toml file.

```
[server]
offset=1
```

After starting that partner identity server, it will run on [localhost:9444](https://localhost:9444/carbon).

1. Access the Management console of the partner identity server.
2. Then go to the Service Providers under the main tab. 
   Then add Service Provider Name and register it. 
   Let’s use the playground app and refer to this to configure the playground app.
3. Then List the Service Providers and edit the service provider by navigation to the** 
   OAuth/OpenID Configuration** under **Inbound Authentication Configuration** and add 
   [https://localhost:9443/commonauth](https://localhost:9443/commonauth) as the callback URL.
4. Create a user **Alex** in the partner identity server.

### Configure Federated Authenticator

To configure the federated authenticator, click the add button under **Identity Providers** and add an IDP name as 
**Partner-Identity-Server** and register the new IDP.

Click on the **Federated Authenticators **and expand the **custom-federated-authenticator configurations** 
and configure it as follows.

Here, the Client Id and Client Secret are the values of external service provider from the Partner-Identity-Server.

*   _Enable / Default - You can **enable** and set to **default**_
*   _Authorization Endpoint UR - [https://localhost:9444/oauth2/authorize/](https://localhost:9444/oauth2/authorize/)_
*   _Token Endpoint URL - [https://localhost:9444/oauth2/token/](https://localhost:9444/oauth2/token/)_
*   _Client Id - The value generated by the service provider of the partner IS_
*   _Client Secret - The value generated by the service provider of the partner IS_

### Configure an application with the custom federated authenticator

1. Start the server and log in to the WSO2 IS Management Console.

2. Then go to the Service Providers under the main tab. Then add Service Provider Name as follows and Register it. Let’s use the playground app and refer this to configure playground app.

3. Then List the Service Providers and edit the service provider as follows by navigation to the** OAuth/OpenID Configuration** under **Inbound Authentication Configuration **as explained above.

4. Then click Configure and add[ http://localhost:8080/playground2/oauth2client](http://localhost:8080/playground2/oauth2client) as the call back URL. Click Update.

5. Navigate to **Local & Outbound Authentication Configuration** as follows, and you can find Authentication Type. 
   Select **Federated Authentication** and select the configured federated authenticator and update to save the changed 
   configurations.

### Try the scenario

1. Access the playground app by using[ http://localhost:8080/playground2](http://localhost:8080/playground2). 
   
2. When it asks for the username password give Alex's username and password who is in the partner identity server.

3. Then the federated authentication can be experienced. 
