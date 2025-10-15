# Missing Authentication Token - Common Causes & Fixes

1. Incorrect URL

Cause: The URL you're calling might be missing the correct resource path or stage name.
Fix: Make sure your API_ENDPOINT includes the full path, like:

```js
const API_ENDPOINT = 'https://your-api-id.execute-api.your-region.amazonaws.com/prod/greet';
```

If your Lambda is behind a resource like /greet, you must include it.

2. Wrong HTTP Method

Cause: You're sending a POST request, but the API might only accept GET, or vice versa.

Fix: Check your API Gateway method configuration and ensure it matches the method used in your fetch() call.

3. Missing Stage Name

Cause: You might be calling `https://your-api-id.execute-api.your-region.amazonaws.com` without /prod or whatever stage name you deployed to.

Fix: Always include the stage name in the endpoint.

4. CORS Misconfiguration

Cause: If you're testing from a browser and CORS isn't enabled properly, the preflight request might fail silently.

Fix: Ensure your API Gateway method has:

- OPTIONS method enabled.  
- CORS headers like:

```sh
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: POST, OPTIONS Access-Control-Allow-Headers: Content-Type
```

5. Deployment Not Updated

Cause: You made changes to the API but didn’t redeploy.
Fix: In API Gateway, click Deploy API after making changes.

**How to Test**

Try calling the endpoint using curl or Postman to isolate whether the issue is with the frontend or the API itself:

```sh
curl -X POST https://your-api-id.execute-api.your-region.amazonaws.com/prod/greet \  
-H "Content-Type: application/json" \  
-d '{"name": "Antony"}'
```
