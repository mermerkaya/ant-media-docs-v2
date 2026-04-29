---
title: Keycloak Integration
description: Integrate Keycloak Identity Management with Ant Media Server for SSO-based WebRTC stream security.
keywords: [Keycloak, SSO, Stream Security, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 9
---

# Keycloak Integration

[Keycloak](https://www.keycloak.org/) is an Identity Management tool that makes authentication and authorization easy for different services by providing a single sign-on (SSO) solution. We can use Keycloak to make WebRTC pages secure by Keycloak authentication.

Ant Media default streaming application `StreamApp.war` has Keycloak integration disabled. If you create your own streaming application or configure an existing application from installation, you should enable and configure Keycloak.

## Keycloak Configuration

1. Check [Keycloak Getting Started](https://www.keycloak.org/guides#getting-started) documentation to set up Keycloak.

2. After making it run, create a **Realm** from Keycloak Dashboard. Let's name it **antmedia**.

3. Create an **Open ID Client** in the Realm (**antmedia**). Let's name it **stream-application**. Then set the URL as in the image below.

   ![](@site/static/img/stream-security/keycloak-client-creation.png)

4. Create a role in the client (**stream-application**). Let's make a role name **user**.

   ![](@site/static/img/stream-security/keycloak-role.png)

5. Create a User in Realm with the role (**user**) we created in step 4. Let's make the user name **streamer1**.

   ![](@site/static/img/stream-security/keycloak-user.png)

   Create the password from: `Users → Click streamer1 → Credentials → Set Password`

With the above configurations, the Keycloak side is ready.

## AMS Configuration

1. Navigate to the application folder in your AMS installation:

   ```bash
   cd /usr/local/antmedia/webapps/{APP-NAME}/WEB-INF
   ```

2. Uncomment the following lines in `red5-web.xml` and set the values according to your Keycloak server configurations:

   ```xml
   <!-- For Keycloak Integration -->
   <bean id="openid.config" class="io.antmedia.SecurityConfiguration">
     <property name="realmUrl" value="http://keycloak.antmedia.cloud:8080/realms/antmedia" />
     <property name="appName" value="live" />
     <property name="clientId" value="stream-application" />
     <property name="role" value="user" />
   </bean>
   ```

   :::info
   The `appName` should be the same as the application name you are configuring. All parameters should be compatible with the configuration in Keycloak.
   :::

3. Uncomment the following lines in `web.xml`:

   ```xml
   <!-- For Keycloak Integration -->
   <filter>
     <filter-name>ContentSecurityPolicyHeaderFilter</filter-name>
     <filter-class>io.antmedia.filter.ContentSecurityPolicyHeaderFilter</filter-class>
     <async-supported>true</async-supported>
   </filter>
   <filter-mapping>
     <filter-name>ContentSecurityPolicyHeaderFilter</filter-name>
     <url-pattern>/*</url-pattern>
   </filter-mapping>
   ```

4. Restart the antmedia service:

   ```bash
   sudo service antmedia restart
   ```

## Test Keycloak Integration

- Try to publish a WebRTC stream:

  ```
  https://{AMS-URL}:5443/{APP-NAME}/samples/publish_webrtc.html
  ```

- Try to play a stream:

  ```
  https://{AMS-URL}:5443/{APP-NAME}/player.html
  ```

When you try to publish or play, it will first ask you to authenticate with the Keycloak user that was created:

![](@site/static/img/stream-security/keycloak-login.png)

Once authenticated, you will be able to publish or play the stream via the sample page.
