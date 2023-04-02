# Logic App Webhook 
In this example, we create a Logic App that is used as a Webhook to Graph Change Notifications.  To achieve this, there are several requirements that our Logic App must deal with.  When subscribing to Graph Change Notifications, Graph will first send a POST request to the Webhook (Logic App) with a Validation Token.  The Logic App must respond with a 200 status-code and must include in the response body the Validation Token as Plain Text that was sent to the Logic App.  Once the Validation Token Logic is in-place, you will need to implement logic to process the Change Notifications.  When Graph sends a change notification, it will send in the Request body a JSON array with the change notification details.  The Logic App should respond as quickly possible with a 202 status-code, the Response body should be empty.  At this point, you process the change notification JSON details per your needs.

### Let's implement the logic to parse the Validation Token ###

1. The first thing we need to do is add an HTTP Trigger <br>
![HTTP Trigger](/assets/HTTP.jpg)

2. Now we need to parse the Headers so we can extract the Content-Type (we will make use of these to determine if we are receiving a Validation Token or a Change Notification). Add a Parse JSON Action and use the schema below.<br>

![Parse Headers](/assets/ParseHeaders1.jpg)

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
