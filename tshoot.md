# Troubleshooting Guide

## API Gateway Endpoint Returns “Missing Authentication Token”

Cause: The frontend is calling the wrong URL — usually missing the resource path.

Fix: Ensure the full endpoint includes the stage and resource path:

`https://<api-id>.execute-api.<region>.amazonaws.com/<stage>/<resource>`

Example:

```js
const API_ENDPOINT = 'https://your-api-id.execute-api.your-region.amazonaws.com/prod/greet';
```

>**Resource Paths** explanation incoming...

## CORS Error in Browser (Preflight or POST Request)

Cause: Missing or incorrect CORS headers in either the OPTIONS method or Lambda response.

Fix: Create an OPTIONS method for the resource in API Gateway - Use Mock Integration and create the following `Method Response headers`:

```sh
Access-Control-Allow-Origin
Access-Control-Allow-Methods
Access-Control-Allow-Headers
Content-Type
```

After adding the `Method Response headers` you then need to provide mappings for these headers under `Integration Response` > `header mappings`:

```sh
Access-Control-Allow-Origin: '*'
Access-Control-Allow-Methods: 'OPTIONS,POST'
Access-Control-Allow-Headers: 'Content-Type'
Content-Type: 'application/json'
```

## Lambda Response Missing CORS Headers

Cause: Lambda returns a response without CORS headers.

Fix: Add these headers to every return block in the Lambda function

```python
'headers': {
    'Content-Type': 'application/json',
    'Access-Control-Allow-Origin': '*',
    'Access-Control-Allow-Headers': 'Content-Type',
    'Access-Control-Allow-Methods': 'OPTIONS,POST'
}
```

### Extra Checks


- **Wrong HTTP Method**

    Cause: You're sending a POST request, but the API might only accept GET, or vice versa.

    Fix: Check your API Gateway method configuration and ensure it matches the method used in your fetch() call.

- **Missing Stage Name**

    Cause: You might be calling `https://your-api-id.execute-api.your-region.amazonaws.com` without /prod or whatever stage name you deployed to.

    Fix: Always include the stage name in the endpoint.

- **Deployment Not Updated**

    Cause: You made changes to the API but didn’t redeploy.

    Fix: In API Gateway, click Deploy API after making changes.

---

Try calling the endpoint using curl to isolate whether the issue is with the frontend or the API itself:

```sh
curl -X POST https://your-api-id.execute-api.your-region.amazonaws.com/prod/greet \  
-H "Content-Type: application/json" \  
-d '{"name": "myName"}'
```
