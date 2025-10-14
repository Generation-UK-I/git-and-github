# Lambda Function Breakdown

## Imports and Logger Setup

```python
import json
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)
```

- Prepares logging and JSON handling.
- `logger` is used to record events and errors for debugging or monitoring.

## Main Entry Point

```python
def lambda_handler(event, context):
```

This is the function AWS Lambda calls when triggered.

- `event`: contains request data (e.g., HTTP body, headers).
- `context`: contains runtime info (e.g., function name, memory limit)

## Log the Incoming Event

```python
logger.info(f'Event received: {event}')
```

Logs the full incoming request for visibility.

## Parse the Request Body

```python
if isinstance(event.get('body'), str):
    body = json.loads(event['body'])
else:
    body = event.get('body', {})
```

Checks if the body is a string (typical in API Gateway):

- If so, parses it as JSON.
- If not, assumes it's already a dictionary.

## Handle Invalid JSON

```python
except json.JSONDecodeError:
    return {
        'statusCode': 400,
        ...
        'body': json.dumps({'error': 'Invalid JSON in request body'})
    }
```

If parsing fails, returns a 400 Bad Request with an error message.

## Extract and Validate `name`

```python
name = body.get('name', '').strip()

if not name:
    return {
        'statusCode': 400,
        ...
        'body': json.dumps({'error': 'Name is required'})
    }
```

- Tries to get the name field from the body.
- Strips whitespace and checks if it’s empty.
- If missing or blank, returns a 400 error.

## Return a Greeting

```python
response = {
    'statusCode': 200,
    ...
    'body': json.dumps({
        'message': f'Hello {name}!'
    })
}
```

If everything is valid, returns a `200` (OK) with a personalized greeting.

## Send the Response

```python
return response
Final step: sends the response back to the caller (e.g., API Gateway).
```

## Example Input / Output

**INPUT**:

```json
{
  "body": "{\"name\": \"Frankie\"}"
}
```

**OUTPUT**:

```json
{
  "statusCode": 200,
  "body": "{\"message\": \"Hello Frankie!\"}"
}
```
