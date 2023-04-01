# Logic App WebHook for Graph Change Notifications
### Important caveats to be aware of ###
1. Graph uses a Validation Token to verify that the notification URL is available when the subscription is created.  The notification URL must respond with a status code of 200 and the response body must include the validation token, otherwise the subscription will not be created. 

2. Graph will continue to retry the notification if it does not receive a 202 status within 3 seconds.  It's a best practice to send the 202 status as soon as possible befor you execute any business logic. Graph does not care if your business logic fails are succeeds.

3. When a change notification occurs the notification URL is called via a POST request with the following details: <br>
~~~
{
  "value": [
    {
      "subscriptionId": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXX",
      "subscriptionExpirationDateTime": "2023-04-04T11:00:00+00:00",
      "changeType": "created",
      "resource": "Users/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXX/Messages/<ID OF MESSAGE",
      "resourceData": {
        "@odata.type": "#Microsoft.Graph.Message",
        "@odata.id": "Users/<ID OF USER>/Messages/<ID OF MESSAGE>,
        "@odata.etag": "W/\"FwAAABYAAACLnjs0xXQOS6FM1c3UJ3VxAAAAAHeU\"",
        "id": "<ID OF MESSAGE>"
      },
      "clientState": "SecretClientState",
      "tenantId": "<TENANT ID>"
    }
  ]
}

~~~
4. You would use the details provided by the change notification outlined in step 3 and parse the JSON for the ID of the message, then take whatever action on the message you would to perform i.e. move the message, get the message body or subject line etc.  Any additional action will require you to make a call to the appropriate Graph API passing the Message ID and a Authorization Token.
