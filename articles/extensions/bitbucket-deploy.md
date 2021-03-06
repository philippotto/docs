---
toc: true
description: The Bitbucket Deployments extension allows you to deploy Rules and Database Connection scripts from Bitbucket to Auth0.
topics:
  - extensions
  - bitbucket-deployments
contentType:
  - how-to
useCase: extensibility-extensions
---

# Bitbucket Deployments

The **Bitbucket Deployments** extension allows you to deploy [rules](/rules), rules configs, connections, database connection scripts, clients, client grants, resource servers, Universal Login pages and email templates from Bitbucket to Auth0. You can configure a Bitbucket repository, keep all of your Rules and Database Connection scripts there, and have them automatically deployed to Auth0 whenever you push changes to your repository.

## Configure the Extension

To install and configure this extension, click on the **Bitbucket Deployments** box in the list of provided extensions on the [Extensions](${manage_url}/#/extensions) page of the Auth0 Management Dashboard. The **Install Extension** window will open.

![](/media/articles/extensions/bitbucket-deploy/configure-extension.png)

Set the following configuration variables:

* **REPOSITORY**: The repository from which you want to deploy your Rules and Database Connection scripts. This can be either a public or private repository
* **BRANCH**: The branch the extension will monitor for changes
* **USER**: The username used to access the Bitbucket account. Make sure you use the username, and not the email
* **PASSWORD**: The user password or an app password you create through the Bitbucket settings to grant permissions to certain apps (`Repositories: Read` permission is required)
* **BASE_DIR**: The base directory, where all your tenant settings are stored
* **AUTO_REDEPLOY**: If enabled, the extension redeploys the last successful configuration in the event of a deployment failure. Manual deployments and validation errors does not trigger auto-redeployment
* **SLACK_INCOMING_WEBHOOK**: The Webhook URL for Slack used to notify you of successful and failed deployments

::: note
Some of the configuration variables were changed in version **2.6.0** of this extension. If you are updating the extension from a prior version, make sure that you update your configuration accordingly.
:::

Once you have provided this information, click **Install**.

Navigate to the [Extensions](${manage_url}/#/extensions) page and click on the **Installed Extensions** tab.

![](/media/articles/extensions/bitbucket-deploy/installed-extensions.png)

Click on the row for the **Bitbucket Deployments** extension. The first time you click on your installed extension, you will be asked to grant it to access your Bitbucket account. Once you agree, you will be directed to the **Bitbucket Integration** page.

The **Configuration** page will display the **Payload URL** you will need to create a [webhook](https://confluence.atlassian.com/bitbucket/manage-webhooks-735643732.html) to your Bitbucket repository pointing to the extension.

![](/media/articles/extensions/bitbucket-deploy/config-parameters.png)

Copy and paste this value into the **Add Webhook** page for your Bitbucket Repository.

![](/media/articles/extensions/bitbucket-deploy/webhook-setup.png)

::: note
You can find details on how to configure a webhook at [Creating Webhooks](https://confluence.atlassian.com/bitbucket/manage-webhooks-735643732.html#Managewebhooks-create_webhookCreatingwebhooks) on Bitbucket.
:::

## Deployment

Once you have set up the webhook in Bitbucket using the provided information, you are ready to start committing to your repository.

With each commit you push to your configured Bitbucket repository, the webhook will call the extension to initiate a deployment if changes were made to one of these folders:
- `clients`
- `grants`
- `emails`
- `resource-servers`
- `connections`
- `database-connections`
- `rules-configs`
- `rules`
- `pages`

The **Deploy** button on the **Deployments** tab of the extension allows you to manually deploy the Rules and Database Connection scripts that you already have in your Bitbucket repository. This is useful if your repository already contains items that you want to deploy once you have set up the extension or if you have accidentally deleted some scripts in Auth0 and need to redeploy the latest version of your repository.

::: panel-warning Full Deployment
To maintain a consistent state, the extension will always do a full deployment of the contents of these folders. Any rules or database connection scripts that exist in Auth0 but not in your GitHub repository will be **deleted**.
:::

### Deploy Database Connection Scripts

To deploy Database Connection scripts, you must first create a directory under `database-connections`. The name of the directory must match **exactly** the name of your [database connection](${manage_url}/#/connections/database) in Auth0. You can create as many directories as you have Database Connections.

Under the created directory, create one file for each script you want to use. The allowed scripts are:

- `get_user.js`
- `create.js`
- `verify.js`
- `login.js`
- `change_password.js`
- `delete.js`

For a generic Custom Database Connection, only the `login.js` script is required. If you enable the migration feature, you will also need to provide the `get_user.js` script.

You can find examples in [the Auth0 Samples repository](https://github.com/auth0-samples/github-source-control-integration/tree/master/database-connections/my-custom-db). While the samples were authored for GitHub, it will work for a Bitbucket integration as well.

#### Deploy Database Connection Settings

To deploy Database Connection settings, you must create `database-connections/[connection-name]/database.json`.

_This will work only for Auth0 connections (strategy === auth0); for non-Auth0 connections use `connections`._

_Support for using `settings.json` has been deprecated in favor of `database.json` since v3.1.1 of the extension and may be dropped in a future release._

See [Management API v2 Docs](https://auth0.com/docs/api/management/v2#!/Connections/patch_connections_by_id) for more info on allowed attributes for Connections.

### Deploy Connections

To deploy a connection, you must create a JSON file under the `connections` directory of your Bitbucket repository. Example:

__facebook.json__
```json
{
  "name": "facebook",
  "strategy": "facebook",
  "enabled_clients": [
    "my-client"
  ],
  "options": {}
}
```

<%= include('./_includes/_embedded-clients-array') %>

_This will work only for non-Auth0 connections (`strategy !== auth0`); for Auth0 connections, use `database-connections`._

For more info on the allowed attributes for connections, see the [Post Connections endpoint] (/api/management/v2#!/Connections/post_connections).

### Deploy Universal Login Pages

The supported pages are:

- `error_page`
- `guardian_multifactor`
- `login`
- `password_reset`

To deploy a page, you must create an HTML file under the `pages` directory of your Bitbucket repository. For each HTML page, you need to create a JSON file (with the same name) that will be used to mark the page as enabled or disabled. For example, to deploy a `password_reset`, you would create two files:

```text
your-bitbucket-repo/pages/password_reset.html
your-bitbucket-repo/pages/password_reset.json
```

To enable the page, the `password_reset.json` would contain the following:

```json
{
  "enabled": true
}
```

<%= include('./_includes/_use-default-error') %>

### Deploy Rules

To deploy a rule, you must first create a JavaScript file under the `rules` directory of your Bitbucket repository. Each Rule must be in its own JavaScript file.

For example, if you create the file `rules/set-country.js`, the extension will create a Rule in Auth0 with the name `set-country`.

::: note
If you plan to use source control integration for an existing account, first rename your Rules in Auth0 to match the name of the files you will be deploying to this directory.
:::

You can control the Rule order and status (`enabled`/`disabled`) by creating a JSON file with the same name as your JavaScript file. For this example, you would create a file named `rules/set-country.json`.

__set-country.js__
```javascript
function (user, context, callback) {
  if (context.request.geoip) {
    user.country = context.request.geoip.country_name;
  }
  callback(null, user, context);
}
```

__set-country.json__
```json
{
  "enabled": false,
  "order": 15,
  "stage": "login_success"
}
```

You can find a `login_success` example in [the Auth0 Samples repository](https://github.com/auth0-samples/github-source-control-integration/tree/master/rules). While the sample was authored for GitHub, it will work for a Bitbucket integration as well.

#### Set Rule Order

To avoid conflicts, you cannot set multiple Rules of the same order. However, you can create a JSON file for each rule, and within each file, assign a value for `order`. We suggest using number values that allow for reordering with less risk of conflict. For example, assign a value of `10` to the first Rule and `20` to the second Rule, rather than using values of `1` and `2`, respectively).

### Deploy Rules Configs

To deploy a rule config, you must create a JSON file under the `rules-configs` directory of your Bitbucket repository. Example:

__secret_number.json__
```json
{
  "key": "secret_number",
  "value": 42
}
```

### Deploy Clients

To deploy a client, you must create a JSON file under the `clients` directory of your Bitbucket repository. Example:

__my-client.json__
```json
{
  "name": "my-client"
}
```

See [Management API v2 Docs](https://auth0.com/docs/api/management/v2#!/Clients/post_clients) for more info on allowed attributes for Clients and Client Grants.

### Deploy Clients Grants

You can specify the client grants for each client by creating a JSON file in the `grants` directory.

__my-client-api.json__
```json
{
  "client_id": "my-client",
  "audience": "https://myapp.com/api/v1",
    "scope": [
      "read:users"
    ]
}
```

<%= include('./_includes/_deployment-extension') %>

### Deploy Resource Servers

To deploy a resource server, you must create a JSON file under the `resource-servers` directory of your Bitbucket repository. Example:

__my-api.json__
```json
{
  "name": "my-api",
  "identifier": "https://myapp.com/api/v1",
  "scopes": [
    {
      "value": "read:users",
      "description": "Allows getting user information"
    }
  ]
}
```

See [Management API v2 Docs](https://auth0.com/docs/api/management/v2#!/Resource_Servers/post_resource_servers) for more info on allowed attributes for Resource Servers.

### Deploy Email Provider

To deploy an email provider, you must create `provider.json` file under the `emails` directory of your Bitbucket repository. Example:

__provider.json__
```json
{
    "name": "smtp",
    "enabled": true,
    "credentials": {
        "smtp_host": "smtp.server.com",
        "smtp_port": 25,
        "smtp_user": "smtp_user",
        "smtp_pass": "smtp_secret_password"
    }
}
```

See [Management API v2 Docs](https://auth0.com/docs/api/management/v2#!/Emails/patch_provider) for more info on allowed attributes for Email Provider.

### Deploy Email Templates

The supported email templates are:
- `verify_email`
- `reset_email`
- `welcome_email`
- `blocked_account`
- `stolen_credentials`
- `enrollment_email`
- `mfa_oob_code`

To deploy an email template, you must create an HTML file under the `emails` directory of your Bitbucket repository. For each HTML file, you need to create a JSON file (with the same name) with additional options for that template. For example, to deploy a `blocked_account` template, you would create two files:

```text
your-bitbucket-repo/emails/blocked_account.html
your-bitbucket-repo/emails/blocked_account.json
```

__blocked_account.json__
```json
{
    "template": "blocked_account",
    "from": "",
    "subject": "",
    "resultUrl": "",
    "syntax": "liquid",
    "body": "./blocked_account.html",
    "urlLifetimeInSeconds": 432000,
    "enabled": true
}
```

## Excluded records

You can exclude the following records from the deployment process: `rules`, `clients`, `databases`, `connections` and `resourceServers`. If excluded, the records will not be modified by deployments.

![](/media/articles/extensions/deploy-extensions/excluded-rules.png)

## Keywords Mapping

Beginning with version **3.0.0**, you can use keywords mapping to manage your secrets and tenant-based environment variables.

There are two ways to use the keyword mappings. You can either wrap the key using `@` symbols (e.g., `@@key@@`), or you can wrap the key using `#` symbols (e.g., `##key##`). 

  - If you use `@` symbols, your value will be converted from a JavaScript object or value to a JSON string.

  - If you use `#` symbols, Auth0 will perform a literal replacement.

This is useful for something like specifying different variables across your environments. For example, you could specify different JWT timeouts for your Development, QA/Testing, and Production environments.

Refer to the snippets below for sample implementations:

__Client.json__
```json
{
  ...
  "callbacks": [
    "##ENVIRONMENT_URL##/auth/callback"
  ],
  "jwt_configuration": {
    "lifetime_in_seconds": ##JWT_TIMEOUT##,
    "secret_encoded": true
  }
  ...
}
```

![](/media/articles/extensions/deploy-extensions/mappings.png)

## Track Deployments

To track your deployments, navigate to the [Extensions](${manage_url}/#/extensions) page, click on the row for the **Bitbucket Deployments** extension, and select the **Deployments** tab. You will see a list of all deployments.

![](/media/articles/extensions/bitbucket-deploy/deploy-tracking.png)

If a deployment fails, you can examine the details of the deployment to determine why. Details are also available for successful deployments.

![](/media/articles/extensions/bitbucket-deploy/logs.png)

If you configured a **Slack Incoming Webhook**, you will be notified on Slack anytime a deployment occurs.
