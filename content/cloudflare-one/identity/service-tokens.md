---
pcx_content_type: how-to
title: Service tokens
weight: 6
---

# Service tokens

You can provide automated systems with service tokens to authenticate against your Zero Trust policies. Cloudflare Access will generate service tokens that consist of a Client ID and a Client Secret. Automated systems or applications can then use these values to reach an application protected by Access.

This section covers how to create, renew, and revoke a service token.

## Create a service token

1. In the [Zero Trust dashboard](https://dash.teams.cloudflare.com), navigate to **Access** > **Service Auth** > **Service Tokens**.

2. Select **Create Service Token**.

3. Name the service token. The name allows you to easily identify events related to the token in the logs and to revoke the token individually.

4. Choose a **Service Token Duration**. This sets the expiration date for the token.

5. Select **Generate token**. You will see the generated Client ID and Client Secret for the service token, as well as their respective request headers.

6. Copy the Client Secret.

    {{<Aside type="warning" header="Important">}}This is the **only time** Cloudflare Access will display the Client Secret. If you lose the Client Secret, you must generate a new service token.
    {{</Aside>}}

## Set up your Access application

You can now use the service token in your [Access policies](/cloudflare-one/policies/access/) and [device enrollment rules](/cloudflare-one/connections/connect-devices/warp/warp-settings/#device-enrollment-permissions). When creating these policies, select the `Service Auth` action to ensure that the identity provider login screen is not required for end users.

For a simple RESTful backend application, your setup would include
- an ALLOW rule that grant your developer team access to the service directly
- a SERVICE AUTH rule that allows any request with your service token access to the application
- an optional device enrollment rule with action SERVICE AUTH that allows the service token to be used from machines that have not connected with Access before


## Connect your service to Access

To authenticate to an Access application using your service token, add the following to the headers of any request:

`CF-Access-Client-Id: <Client ID>`

`CF-Access-Client-Secret: <Client Secret>`

Send one request with these headers to any URL belonging to your Access application.  If the service token is valid, Access will return the application's responose and a `CF_Authorization` cookie, containing a JWT scoped to the application.  All subsequent requests with that JWT cookie will succeed until the expiration of the JWT (default 
TODO).

Using the JWT cookie instead of the request headers makes programmatic interaction with the application easier and safer:  You can use standard cookie mechanisms, avoid sending the secret headers over the wire, and isolate knowledge of the secret headers to a part of your application.

## Renew service tokens

Service tokens expire according to the token duration you selected when you created the token.

To renew the service token:

1. In the [Zero Trust dashboard](https://dash.teams.cloudflare.com), navigate to **Access** > **Service Auth** > **Service Tokens**.
2. Locate the token you want to renew.
3. To extend the token's lifetime by one year, select **Refresh**.
4. To extend the token's lifetime by more than a year:
    1. Select **Edit**.
    2. Choose a new **Service Token Duration**.
    3. Select **Save**. The expiration date will be extended by the selected amount of time.

## Revoke service tokens

If you need to revoke access before the token expires, simply delete the token.

1. In the [Zero Trust dashboard](https://dash.teams.cloudflare.com), navigate to **Access** > **Service Auth** > **Service Tokens**.
2. **Delete** the token you need to revoke.

Services that rely on a deleted service token can no longer reach your application.

{{<Aside type="note">}}
When editing an Access application, clicking **Revoke existing tokens** revokes existing sessions but does not prevent the user from starting a new session. As long as the Client ID and Client Secret are still valid, they can be exchanged for a new token on the next request. To revoke access, you must delete the service token.
{{</Aside>}}

## Set a token expiration alert

An alert can be configured to notify a week before a service token expires to allow an administrator to invoke a token refresh.

To configure a service token expiration alert:

1. In the [Cloudflare dashboard](https://dash.cloudflare.com), navigate to the **Notifications** tab.
2. Select **Add**.
3. Select _Expiring Access Service Token_.
4. Enter a name for your alert and an optional description.
5. (Optional) Add other recipients for the notification email.
6. Select **Save**.

Your alert has been set and is now visible in the **Notifications** tab of the Cloudflare dashboard.
