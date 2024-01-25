# KeyCloak and Discourse Docker dev environment
* KeyCloak URL: http://localhost:8080
* Discourse URL: http://localhost:3000

### 1. Clone Discourse 
```bash
$ git clone https://github.com/discourse/discourse.git
```

Modify `discourse/config/database.yml` to add `username: discourse` to the development database connection configuration:

```yml
development:
  prepared_statements: false
  username: discourse
  ...
```

### 2. Start the containers
```bash
$ docker-compose up
```

### 3. Set-up Discourse
```bash
$ docker-compose exec -w /src discourse bundle install
$ docker-compose exec -w /src discourse yarn install
$ docker-compose exec -w /src discourse rake db:migrate
$ docker-compose exec -w /src discourse rake admin:create
Email:
...
```
Complete the email and password and set the user to be an admin.

### 4. Start Discourse server
```bash
$ docker-compose exec -w /src discourse rails server
I, [2024-01-25T14:25:22.938512 #5719]  INFO -- : Refreshing Gem list
Starting CSS change watcher
I, [2024-01-25T14:25:25.889439 #5719]  INFO -- : listening on addr=0.0.0.0:3000 fd=24
...
```
### 5. Configure Discourse to rely on Keycloak for authentication
These instructions were writting for **Keycloak version XX** and **Discourse version yy**.
Original documention:
- https://www.keycloak.org/getting-started/getting-started-docker
- https://meta.discourse.org/t/discourse-openid-connect/103632
- https://meta.discourse.org/t/install-plugins-in-discourse/19157

From now on, we'll consider the following:
- Keycloak is hosted at: `https//keycloak.local`
- Discourse is hosted at: `https//discourse.local`

#### Configure keycloak
1. Open the [Keycloak Admin Console](http://keycloak.local/admin).
2. Create a new Realm called `home`
3. Select the Realm `home`
4. Create a client and fill the form with the following values:
    - **Client type**: `OpenID Connect`
    - **Client ID**: `discourse`
5. On the next page, confirm that **Standard flow** and **Client authentication** is enabled.
6. Make these changes under **Login settings**
    - **Valid redirect URIs**: `https//discourse.local/auth/oidc/callback`
    - **Web origins**: `https://discourse.local`
    - (optional) **Home URL**: If your keycloak instance is not reachable from the browser the same way as it is than for discourse (e.g. if a proxy is involved): Default URL to use when the auth server needs to redirect or link back to the client. E.g. `http://localhost:8080/`
7. Click Save.
8. Verify on the `discourse` client in the **Credentials** tab that the **Client Authenticator** is set to `Client Id and Secret`. Take note of the **Client Secret**, we'll refer to this secret with `{client_secret}` for now on.
9. [optional] In **Realm settings** under the **General** tab
    - **Frontend URL**: If your keycloak instance is not reachable from the browser the same way as it is than for discourse (e.g. if a proxy is involved): Set the frontend URL for the realm. Use in combination with the default hostname provider to override the base URL for frontend requests for a specific realm. E.g. `http://localhost:8080`
10. Create a user `myuser`, and manually set add the password `Password1234` under the **Credentials** panel.


#### Install the `discourse-openid-connect` plugin and configure Discourse
##### Install the `discourse-openid-connect` plugin
1. On the server where discourse is installed, access your container's `app.yml` file (present in `/var/discourse/containers/`)
2. Add the plugin's repository URL to your container's `app.yml` file:
```yaml
hooks:
  after_code:
    - exec:
        cd: $home/plugins
        cmd:
          - git clone https://github.com/discourse/docker_manager.git
          - git clone https://github.com/discourse/discourse-spoiler-alert.git
```
3. Rebuild the container. That might take some time
```bash
cd /var/discourse
./launcher rebuild app
```

##### Configure discourse
1. Log in as an **admin** user in discourse
2. Go to the **Admin** page, then **Settings**
3. Open the **Discourse OpenID Connect** category
    - /admin/site_settings/category/discourse_openid_connect
4. Set the following settings
    - **openid connect enabled**: Enable the checkbox
    - **openid connect discovery document**: `https//keycloak.local/realms/home/.well-known/openid-configuration`
    - **openid connect client id**: `discourse`
    - **openid connect client secret**: `{client_secret}`
    - (optional) **openid connect verbose logging**: Enable if you have issues when trying to log in
        - You can access the logs at `https://discourse.local/logs/`
    - **openid connect authorize scope**: `openid`
5. Save all settings
6. (optional) In case your keycloak is running in an internal host, you have to put this IP address in an allowlist.
    - Go in the **Settings** under the **Security** category and add any relevant IP or CDIR block in the **allowed internal hosts**. E.g. `172.17.0.2` and `172.17.0.0/16`
7. Try to connect log in using the user `myuser` with password `Password1234`
8. Success!
