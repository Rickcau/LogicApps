# Logic App WebHook for Graph Change Notifications
### Important caveats to be aware of ###
1. Graph uses a Validation Token to verify that the notification URL is available when the subscription is created.  The notification URL must respond with a status code of 200 the response body must include the validation token, otherwise the subscription will not be created. 
2. When a change notification occurs the notification URL called via a POST request with the following details:
<TBD>
  
