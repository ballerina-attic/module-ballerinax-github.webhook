Registers a GitHub webhook.

# Module Overview

This module allows registering a [GitHub webhook](https://developer.github.com/webhooks/) programmatically, 
to subscribe to interested GitHub events.

The webhook callback is represented by a service that listens on a listener of type `githubwebhook3:Listener`.
The resources allowed in this service map to possible GitHub events (e.g., `onIssueCommentCreated`, 
`onIssueCommentEdited`, `onIssuesAssigned`, `onIssuesClosed`, etc.). 
The first parameter of each resource is the generic `websub:Notification` record. The second parameter of each 
resource is a `record`, mapping the `json` payload that is expected with each event (e.g., `githubwebhook3:IssueCommentEvent`, 
`githubwebhook3:IssuesEvent`, etc.).

This module supports the following functionalities:
- Programmatic subscription (requires a GitHub access token). Alternatively, a webhook can also be registered via the UI.
- Authenticated content distribution (If a webhook was registered specifying a secret, signature validation would 
be done for each content delivery request.)
- Data binding of the `json` payload for each event, to a `record` mapping the same
- Programmatic response to all content delivery requests - an appropriate response is sent for all content delivery 
requests, prior to dispatching them to the relevant resource.

## Compatibility
|                             |       Version               |
|:---------------------------:|:---------------------------:|
| Ballerina Language          |    Swan Lake Preview1       |

### Prerequisites

- Download and install [Ballerina](https://ballerinalang.org/downloads/).
- Install [npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm) and setup the [ngrok](https://developer.bigcommerce.com/api-docs/getting-started/webhooks/setting-up-webhooks).
- Obtaining the Access Token to run the Sample
    - In your [GitHub profile settings](https://github.com/settings/profile), click **Developer settings**, and then click **Personal access tokens**.
    - Generate a new token with at least the `admin:repo_hook` scope for webhook-related functionality.

> Need to be start the `ngork` with `webhook:Listener` service port by using the command `./ngrok http 8080`

## Sample

First, import the `ballerinax/github.webhook` module into the Ballerina project.

```ballerina
import ballerinax/github.webhook;
```

Access token, callback URL(eg: `http://1c9b0ff10cea.ngrok.io/github`), username and repository name need to be specified when configuring the subscription parameters of the service annotation.

```ballerina
oauth2:OutboundOAuth2Provider githubOAuth2Provider = new ({
    accessToken: "<GITHUB_ACCESS_TOKEN>"
});
http:BearerAuthHandler githubOAuth2Handler = new (githubOAuth2Provider);

@websub:SubscriberServiceConfig {
    path: "/github",
    subscribeOnStartUp: true,
    target: [webhook:HUB, "https://github.com/<GITHUB_USERNAME>/<GITHUB_REPO_NAME>/events/*.json"],
    hubClientConfig: {
        auth: {
            authHandler: githubOAuth2Handler
        }
    },
    callback: "<CALLBACK_URL>"
}
```

The following sample code also accepts notifications when an issue is opened (`onIssuesOpened`) and when the repository is starred (`onWatch`).

```ballerina
import ballerina/http;
import ballerina/io;
import ballerina/oauth2;
import ballerina/websub;
import ballerinax/github.webhook;

listener webhook:Listener githubListener = new (8080);

oauth2:OutboundOAuth2Provider githubOAuth2Provider = new ({
    accessToken: "<GH_ACCESS_TOKEN>"
});
http:BearerAuthHandler githubOAuth2Handler = new (githubOAuth2Provider);

@websub:SubscriberServiceConfig {
    path: "/webhook",
    subscribeOnStartUp: true,
    target: [webhook:HUB, "https://github.com/<GH_USERNAME>/<GH_REPO_NAME>/events/*.json"],
    hubClientConfig: {
        auth: {
            authHandler: githubOAuth2Handler
        }
    },
    callback: "<CALLBACK_URL>"
}
service githubWebhook on githubListener {

    resource function onPing(websub:Notification notification, webhook:PingEvent event) {
        io:println("[onPing] Webhook Registered: ", event);
    }

    resource function onIssuesOpened(websub:Notification notification, webhook:IssuesEvent event) {
        io:println("[onIssuesOpened] Issue ID: ", event.issue.number);
    }

    resource function onWatch(websub:Notification notification, webhook:WatchEvent event) {
        io:println("[onWatch] Repository starred by: ", event.sender);
    }
}
```
