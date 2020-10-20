---
title: "Setting up Snowflake OAuth"
id: "setting-up-enterprise-snowflake-oauth"
---

:::info Enterprise Feature

This guide describes a feature of the dbt Cloud Enterprise plan. If you’re interested in learning more about an Enterprise plan, contact us at sales@getdbt.com.

:::

dbt Cloud Enterprise supports [OAuth authentication](https://docs.snowflake.net/manuals/user-guide/oauth-intro.html) with Snowflake. When Snowflake OAuth is enabled, users can authorize their Development credentials using Single Sign On (SSO) via Snowflake rather than submitting a username and password to dbt Cloud.

### Configuring a security integration
To enable Snowflake OAuth, you will need to create a [security integration](https://docs.snowflake.net/manuals/sql-reference/sql/create-security-integration.html) in Snowflake to manage the OAuth connection between dbt Cloud and Snowflake.

### Create a security integration

In Snowflake, execute a query to create a security integration. Please find the complete documentation on creating a security integration for custom clients [here](https://docs.snowflake.net/manuals/sql-reference/sql/create-security-integration.html#syntax). You can find a sample `create or replace security integration` query below.

```
CREATE OR REPLACE SECURITY INTEGRATION DBT_CLOUD_<PROJECT_NAME>
  TYPE = OAUTH
  ENABLED = TRUE
  OAUTH_CLIENT = CUSTOM
  OAUTH_CLIENT_TYPE = 'CONFIDENTIAL'
  OAUTH_REDIRECT_URI = 'https://cloud.getdbt.com/complete/snowflake'
  OAUTH_ISSUE_REFRESH_TOKENS = TRUE
  OAUTH_REFRESH_TOKEN_VALIDITY = 7776000;
```

:::caution Permissions

  Note: Only Snowflake account administrators (users with the `ACCOUNTADMIN` role) or a role with the global `CREATE INTEGRATION` privilege can execute this SQL command.

:::

| Field | Description |
| ----- | ----------- |
| TYPE  | Required |
| ENABLED  | Required |
| OAUTH_CLIENT  | Required |
| OAUTH_CLIENT_TYPE  | Required |
| OAUTH_REDIRECT_URI  | Required. If dbt Cloud is deployed on-premises, use the domain name of your application instead of `cloud.getdbt.com` |
| OAUTH_ISSUE_REFRESH_TOKENS  | Required |
| OAUTH_REFRESH_TOKEN_VALIDITY  | Required. This configuration dictates the number of seconds that a refresh token is valid for. Use a smaller value to force users to re-authenticate with Snowflake more frequently. |

Additional configuration options may be specified for the security integration as needed.

### Configure a Connection in dbt Cloud

The Database Admin is responsible for creating a Snowflake Connection in dbt Cloud. This Connection is configured using a Snowflake Client ID and Client Secret. When configuring a Connection in dbt Cloud, select the "Allow SSO Login" checkbox. Once this checkbox is selected, you will be prompted to enter an OAuth Client ID and OAuth Client Secret. These values can be determined by running the following query in Snowflake:

```
select SYSTEM$SHOW_OAUTH_CLIENT_SECRETS('DBT_CLOUD_<PROJECT_NAME>');
```

This query should return a single variant column containing three fields:
- `OAUTH_CLIENT_ID`
- `OAUTH_CLIENT_SECRET`
- `OAUTH_CLIENT_SECRET_2`

Enter the Client ID and Client Secret into dbt Cloud to complete the creation of your Connection. Note that the `OAUTH_CLIENT_SECRET_2` field is unused in dbt Cloud configuration.

<Lightbox src="/img/docs/dbt-cloud/dbt-cloud-enterprise/1bd0c42-Screen_Shot_2020-03-10_at_6.20.05_PM.png" title="Configuring OAuth credentials in the dbt Cloud UI" />

### Authorize Developer Credentials

Once Snowflake SSO is enabled, users on the project will be able to configure their credentials in their Profiles. By clicking the "Connect to Snowflake Account" button, users will be redirected to Snowflake to authorize with the configured SSO provider, then back to dbt Cloud to complete the setup process. At this point, users should now be able to use the dbt IDE with their development credentials.

### SSO OAuth Flow Diagram

![image](https://user-images.githubusercontent.com/46451573/84427818-841b3680-abf3-11ea-8faf-693d4a39cffb.png)

Once a user has authorized dbt Cloud with Snowflake via their identity provider, Snowflake will return a Refresh Token to the dbt Cloud application. dbt Cloud is then able to exchange this refresh token for an Access Token which can then be used to open a Snowflake connection and execute queries in the dbt Cloud IDE on behalf of users.

**NOTE**: The lifetime of the refresh token is dictated by the OAUTH_REFRESH_TOKEN_VALIDITY parameter supplied in the “create security integration” statement. When a user’s refresh token expires, the user will need to re-authorize with Snowflake to continue development in dbt Cloud.

### Setting up multiple dbt Cloud projects with Snowflake 0Auth
If you are planning to set up the same Snowflake account to different dbt Cloud projects, you can use the same security integration for all of the projects. 

