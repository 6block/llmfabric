# API Key Management

{{ brand.name }} supports authentication using API keys. Each {{ brand.name }} user can generate and manage their own API keys.

## Create API Key

1. Hover over the user avatar and navigate to the `API Keys` page.
2. Click the `Add API Key` button.
3. Fill in the `Name`, `Description`, and select the `Expiration` of the API key.
4. In the Model Access section, select either **All models** or **Allowed models**, and if choosing **Allowed models**, select which models this API key can access from the list.
5. Click the `Save` button.
6. Copy and store the key somewhere safe, then click the `Done` button.

!!! note

    Please note that you can only see the generated API key once upon creation.

## Edit Model Access

1. Hover over the user avatar and navigate to the `API Keys` page.
2. Find the API key you want to edit.
3. Click the `Edit` button in the `Operations` column.
4. In the Model Access section, select either **All models** or **Allowed models**, and if choosing **Allowed models**, select which models this API key can access from the list.
5. Click the `Save` button.

!!! note

    Changes will take effect within one minute.

## Delete API Key

1. Hover over the user avatar and navigate to the `API Keys` page.
2. Find the API key you want to delete.
3. Click the `Delete` button in the `Operations` column.
4. Confirm the deletion.

## Use API Key

{{ brand.name }} supports using the API key as a bearer token. The following is an example using curl:

```bash
export {{ brand.env_prefix }}_API_KEY=your_api_key
curl http://your_{{ brand.executable_name }}_server_url/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${{ brand.env_prefix }}_API_KEY" \
  -d '{
    "model": "qwen3",
    "messages": [
      {
        "role": "system",
        "content": "You are a helpful assistant."
      },
      {
        "role": "user",
        "content": "Hello!"
      }
    ],
    "stream": true
  }'
```
