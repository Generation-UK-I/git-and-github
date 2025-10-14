# JS Script Breakdown

## API Endpoint Setup

```js
JavaScriptconst API_ENDPOINT = 'https://YOUR_API_ID.execute-api.YOUR_REGION.amazonaws.com/prod';``Show more lines
```

- This is a placeholder for your actual AWS API Gateway endpoint.
- You need to replace YOUR_API_ID and YOUR_REGION with your actual values.

## DOM Element References

```js
const form = document.getElementById('greetingForm');const nameInput = document.getElementById('nameInput');const responseDiv = document.getElementById('response');const loadingDiv = document.getElementById('loading');``Show more lines
```

These lines grab references to HTML elements:

- `form`: the form element the user submits.
- `nameInput`: the input field where the user types their name.
- `responseDiv`: where the server's response is displayed.
- `loadingDiv`: a visual indicator (like a spinner) shown during the request.

## Form Submission Handler

```js
form.addEventListener('submit', async (e) => {
    e.preventDefault();
```

- Prevents the default form submission behavior (page reload).
- Uses async to handle asynchronous operations (like fetch).

## Input Validation

```js
const name = nameInput.value.trim();if (!name) {
    showResponse('Please enter your name', 'error');
    return;}
```

- Trims whitespace and checks if the name is empty.
- If empty, shows an error message and stops further execution.

## Loading State

```js
loadingDiv.classList.add('show');
responseDiv.classList.remove('success', 'error');
```

- Shows the loading indicator.
- Clears any previous success or error styling.

## Sending the Request

```js
const response = await fetch(API_ENDPOINT, {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({ name: name })
});
```

- Sends a POST request to the API with the name in JSON format.

## Handling the Response

```js
const data = await response.json();

if (response.ok) {
    showResponse(data.message, 'success');
    nameInput.value = ''; // Clear input
} else {
    showResponse(data.error || 'An error occurred', 'error');
}
```

Parses the JSON response.

- If the response is successful (response.ok), shows the message.
- Otherwise, shows an error message.

## Error Handling

```js
} catch (error) {
    console.error('Error:', error);    
    showResponse('Failed to connect to the server. Check the console.', 'error');
}
```

Catches any network or unexpected errors and logs them.

## Cleanup

```js
finally {
    loadingDiv.classList.remove('show');
}
```

- Hides the loading indicator regardless of success or failure.

## Helper Function

```js
function showResponse(message, type) {
    responseDiv.textContent = message;
    responseDiv.className = `response ${type}`;
}
```

- Updates the response message and applies a CSS class (success or error) for styling.
