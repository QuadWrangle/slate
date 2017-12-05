# Errors


Error Code | Meaning
---------- | -------
400 | Bad Request 
401 | Unauthorized -- Invalid API Key and/or App Client ID
403 | Forbidden -- Shouldn't happen, but typically this is an API Key is wrong error
404 | Not Found -- Entity was not found
500 | Internal Server Error -- An error occurred, typically a bad request or something internal happend. Most 500 errors are logged
503 | Service Unavailable -- We're temporarily offline for maintenance. You will know most of the time if this happens. It's pretty rare
