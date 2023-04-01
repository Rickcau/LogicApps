# Logic App WebHook for Graph Change Notifications
### Important caveats to be aware of ###
1. Graph uses a Validation Token to verify that the notification URL is available when the subscription is created.  The notification URL must respond with a status code of 200 and the response body must include the validation token, otherwise the subscription will not be created. 

2. When a change notification occurs the notification URL called via a POST request with the following details: <br>
<TBD>
<p>Graph will continue to retry the notification if it does not receive a 202 status within 3 seconds.  It's a best practice to send the 202 status as soon as possible for you execute any business logic because Graph does not care if you business logic fails are succeeds. </p>  
ss
