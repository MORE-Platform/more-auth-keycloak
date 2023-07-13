# Keycloak

_Keycloak repackaged and preconfigured for MORE_

----

## Setup

This repository contains a `docker-compose.yaml` that can be used as a (pretty complete) template to run Keycloak for
the MORE Platform.

<details>
<summary>Additional Steps to start Keycloak for <strong>local development</strong></summary>
To start Keycloak for (local) development, you can deactivate some security settings to avoid problems with 
e.g. the SSL configuration:

```shell
docker run --name=keycloak \
  -p 8099:8080 \
  -e KEYCLOAK_ADMIN=admin \
  -e KEYCLOAK_ADMIN_PASSWORD=admin \
  -e KC_HOSTNAME=localhost \
  -e KC_HOSTNAME_PORT=8099 \
  ghcr.io/more-platform/auth-keycloak:latest start-dev
```
After this, Keycloak will be available under http://localhost:8099/. You can login to the `Master`-realm using the
provided admin-credentials (admin/admin).

**Note:** Do not forget to add `http://localhost:8080/*` and `http://localhost:3000/*` to the "Valid redirect URIs"
when configuring the realm for MORE!

</details>

### Hardening for Production
For Details on recommended configuration settings for production usage, please refer to the 
[Keycloak Documentation](https://www.keycloak.org/server/configuration-production). 

### Initial Configuration
However, after the first startup, some initial configuration-steps are required to prepare a dedicated `Realm` for MORE.

Some required variables/values during the setup:
```dotenv
AUTH_URL: https://auth.example.com/
STUDY_MANAGER_URL: https://study-manager.example.com/
LIMESURVEY_URL: https://limesurvey.example.com/
```
1. Point your browser to https://auth.example.com/ and login with the init-credentials from the startup configuration.
2. Create a new **Realm**: `MORE-Platform` and switch to the new Realm to configure it:
3. Create **Roles**: In the navigation, go to `Realm Roles` and create the following roles:
   1. `more-admin` aka. MORE_ADMIN (System Administrator, Platform Administrator): Rights to manage users, emergency functions, no rights to see data or manipulate studies.
   2. `more-operator` aka. MORE_OPERATOR: Can create/initiate a new Study.
   3. `more-viewer` aka. MORE_VIEWER: Can access existing studies (based on assigned study-level roles).
4. Create **Groups**: In the navigation, go to `Groups` and create the following groups:
   1. `MORE Administrators`: After creation, select the created group and go to the `Role mapping`-Tab to assign the role `more-admin`
   2. `MORE Researcher`: After creation, select the created group and go to the `Role mapping`-Tab to assign the role `more-viewer`
      1. Go to the `Child Groups`-Tab and create the group `MORE Study Initiator`: After creation, select the created sub-group and go to the `Role mapping`-Tab to assign the role `more-operator`
5. Create an OpenId-Client for the **Study-Manager**
   * Client-Type: `OpenID Connect`
   * Client-ID: `study-manager`
   * Provide a name and description if applicable, then klick `Next`
   * Enable the `Standard flow` and then `Save` the realm.
   * In the "General Settings"-Section, enter the `${STUDY_MANAGER_URL}` from above into "Root URL", "Home URL"
   * Add `${STUDY_MANAGER_URL}/*` to the "Valid redirect URIs"
   * Press `Save` again.
6. Create an OpenId-Client for **Limesurvey**
    * Client-Type: `OpenID Connect`
    * Client-ID: `limesurvey`
    * Provide a name and description if applicable, then klick `Next`
    * Enable `Client authentication` and the `Standard flow`, then `Save` the realm.
    * In the "General Settings"-Section, enter the `${LIMESURVEY_URL}` from above into "Root URL", "Home URL"
    * Add `${LIMESURVEY_URL}/index.php/admin/authentication/sa/login` to the "Valid redirect URIs"
    * Press `Save` again.
    * Take a note of the `Client secret` on the `Credentials`-tab.

### Connect the Applications to Keycloak:

* For the Studymanager use the following settings:
  ```dotenv
   OAUTH2_SERVER: https://auth.example.com/realms/More-Platform
   OAUTH2_CLIENT_ID: study-manager
   OAUTH2_CLIENT_SECRET: ''

   MORE_FE_KC_SERVER: https://auth.example.com/
   MORE_FE_KC_REALM: More-Platform
   MORE_FE_KC_CLIENT_ID: study-manager
  ```

* For Limesurvey use the following settings:
  ```dotenv
   CLIENT_ID: limesurvey
   CLIENT_SECRET: secret-from-the-credentials-tab
   AUTHORIZE_URL: https://auth.example.com/realms/More-Platform/protocol/openid-connect/auth
   SCOPES: openid
   ACCESS_TOKEN_URL: https://auth.example.com/realms/More-Platform/protocol/openid-connect/token
   USER_DETAILS_URL: https://auth.example.com/realms/More-Platform/protocol/openid-connect/userinfo
  ```