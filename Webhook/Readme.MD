# Logic App Webhook 
In this example, we create a Logic App that is used as a Webhook to Graph Change Notifications.  To achieve this, there are several requirements that our Logic App must deal with.  When subscribing to Graph Change Notifications, Graph will first send a POST request to the Webhook (Logic App) with a Validation Token.  The Logic App must respond with a 200 status-code and must include in the response body the Validation Token as Plain Text that was sent to the Logic App.  Once the Validation Token Logic is in-place, you will need to implement logic to process the Change Notifications.  When Graph sends a change notification, it will send in the Request body a JSON array with the change notification details.  The Logic App should respond as quickly possible with a 202 status-code, the Response body should be empty.  At this point, you process the change notification JSON details per your needs.

### Let's implement the logic to parse the Validation Token and Change Notifications ###

1. The first thing we need to do is add an HTTP Trigger <br>
![HTTP Trigger](/assets/HTTP.jpg)

2. Now we need to parse the Headers so we can extract the Content-Type (we will make use of these to determine if we are receiving a Validation Token or a Change Notification). Add a Parse JSON Action and use the schema below.<br>
![Parse Headers](/assets/ParseHeaders1.jpg) <br>

    ~~~
    {
        "properties": {
            "headers": {
                "properties": {
                    "Accept": {
                        "type": "string"
                    },
                 "Connection": {
                        "type": "string"
                 },
                "Content-Length": {
                        "type": "string"
                 },
                 "Content-Type": {
                        "type": "string"
                 },
                 "Host": {
                        "type": "string"
                 }
            },
            "type": "object"
        }
        },
      "type": "object"
    }
    ~~~
    
 3. We need to initialize 3 variables and extract the Content-Type from the Headers and store that in a variable for processing later.<br>
 ![Initi Vars Store Content-Type](/assets/InitiVars-ContentType.jpg)
 
 4. Let's add a Switch Action so we can check the Content-Type for **text/plain** or for **application/json**. If the Content-Type is **text/plain** we know we have a Validation Token, if it’s **application/json** we know we have a change notification and we can process accordingly. 
 ![Switch](/assets/Switch.jpg)
 
 5. Let's add the logic for the Validation Token, which means the Content-Type is set to **text/plain**.  We know that when we get a Validation Token that we need to respond with a 200 and also set the response body to be the Validation Token that was sent to us.<br>
![Switch](/assets/SwitchPlain.jpg)

6. Now, we need to add the logic for Change Notifications, which means the Content-Type is set to **application/json**.  In this case, the request body will need a different schema.<br>
![Switch](/assets/SwitchJSON.jpg)<br>
    The schema that you will use for the **Parse JSON HTTP Trigger Body** is as follows:
    ~~~
    {
    "properties": {
        "value": {
            "items": {
                "properties": {
                    "changeType": {
                        "type": "string"
                    },
                    "clientState": {
                        "type": "string"
                    },
                    "resource": {
                        "type": "string"
                    },
                    "resourceData": {
                        "properties": {
                            "@@odata.id": {
                                "type": "string"
                            },
                            "@@odata.type": {
                                "type": "string"
                            },
                            "activity": {
                                "type": "string"
                            },
                            "availability": {
                                "type": "string"
                            },
                            "id": {}
                        },
                        "type": "object"
                    },
                    "subscriptionExpirationDateTime": {
                        "type": "string"
                    },
                    "subscriptionId": {
                        "type": "string"
                    },
                    "tenantId": {
                        "type": "string"
                    }
                },
                "required": [
                    "subscriptionId",
                    "clientState",
                    "changeType",
                    "resource",
                    "subscriptionExpirationDateTime",
                    "resourceData",
                    "tenantId"
                ],
                "type": "object"
            },
            "type": "array"
        }
    },
    "type": "object"
    }
    ~~~
    Note that in the **Compose MessageID** action I am able to extract that **id** of the message without using a Loop, which is much more efficient.  I am using the following expression to drill into the **id**.<br>
    ~~~
        body('Parse_JSON')['value'][0]['resourceData']['id']
    ~~~
<br>

### Add business logic (optional) ### 
At this point, you would add your business logic to the ***application/json Case section***. For example, if you wanted to get the subject line of the message for a Shared Mailbox, you would need to implement the following logic.<br>

1. Send an HTTP POST request to https://login.microsoftonline.com/ to acquire an oAuth Token.  Since we are using Shared Mailboxes the Graph Permissions, we need can only be granted when using delegation, so we must use a Service Account for this.
![HTTP oAuth](/assets/HTTPGetAuth.jpg)<br>

2. Now, we need to extract the oAuth Token from the POST request, so we use a Parse JSON action to do this.
![Parse Auth](/assets/ParseAuth.jpg)<br>

    The schema that you need to use in the Parse JSON action is as follows.
    ~~~
        {
        "properties": {
            "access_token": {
                "type": "string"
            },
            "expires_in": {
                "type": "string"
            },
            "expires_on": {
                "type": "string"
            },
            "ext_expires_in": {
                "type": "string"
            },
            "not_before": {
                "type": "string"
            },
            "resource": {
                "type": "string"
            },
            "token_type": {
                "type": "string"
            }
          },
          "type": "object"
       }
    ~~~

3. Next, we set the **szAuthorizationHeaderValue** variable that we created eariler as we will use this to pass the **Bearer** Token to the Graph API so we can read the subject line of the message.<br>
![Set Var Call Graph](/assets/SetAuthGetMessage.jpg)<br>

### Create the Graph Subscription ###
In order to use this Logic App as a Webhook for a Graph Subscription, you will need to create the subscription.  Read the details in the following section for how to create the subscription.<br>
<a href="https://github.com/Rickcau/LogicApps/blob/main/Graph%20Subscriptions.MD" target="_blank">How To - Create Graph Subscriptions</a>

### Create the Service Account and App Registeration in Azure Active Directory ###
Since we are subscribing to change notifications for Share Mailboxes, if we plan to take action on those notifications i.e. move message, read message or other actions, we will need use a Service Account and Create an App Registeration. <br>

1. You will need proper permissions to create a user in Azure Active Directory and assign it the Security Reader Role.<br>
![Create User](/assets/NewUser.jpg)<br>

2. Create an App Registeration as outlined in the following section.<br>
<a href="https://github.com/Rickcau/LogicApps/blob/main/AppRegisteration.MD" target="_blank">How To - Create App Registeration</a>

3. The details from the above two steps are to be used in the **HTTP Get oAuthToken** action.
![GetAuthToken](/assets/HTTPGetAuth.jpg)<br>

# ARM Template - For this example
You will find the ARM template for this Logic App via the following link.<br>
<a href="https://github.com/Rickcau/LogicApps/blob/main/Webhook/assets/LA-WebHook-ARM-Template.zip" target="_blank">Logic App Webhook ARM Template</a>

You will need to search the ARM Template for all items wrapped in **< >** and replace those items to match your environment.

