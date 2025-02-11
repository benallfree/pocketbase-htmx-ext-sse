# HTMX SSE for PocketBase

A custom Server-Sent Events (SSE) extension for HTMX that integrates with PocketBase's realtime API.

## Overview

This extension builds upon the [official HTMX SSE extension](https://htmx.org/extensions/sse/) but is specifically designed to work with [PocketBase's realtime API](https://pocketbase.io/docs/api-realtime/). It automatically handles PocketBase's connection protocol and subscription management.

## Features

- Automatic connection handling with PocketBase's realtime API
- Seamless integration with HTMX's event system
- Automatic subscription to specified topics
- Support for dynamic content updates
- Compatible with existing HTMX SSE event names and triggers

## Installation

```bash
npm install pocketbase-htmx-ext-sse
```

or

```html
<script src="https://unpkg.com/pocketbase-htmx-ext-sse"></script>
```

## Usage

Add the extension to your HTML and specify the topics to subscribe to:

```html
<div hx-ext="pocketbase-sse">
  <div sse-swap="chat" hx-swap="beforeend">
    <!-- Content from SSE events will be appended here -->
  </div>
</div>

<form hx-post="/api/chat" hx-swap="none">
  <input type="text" name="message" />
  <button type="submit">Send</button>
</form>
```

### Backend Implementation

#### Using PocketPages

> _See [PocketPages send()](https://pocketpages.dev/docs/context-api/send) for more information._

Create an endpoint at `/api/chat.ejs`:

```html
<script server>
  const { message } = body()
  send('chat', `<div>${message}</div>`)
  return { result: 'ok' }
</script>
```

Or using PocketPage's render capture feature for more complex content:

```html
<script server>
  const { message } = body()
  send('chat')
</script>
<div class="chat-message">
  <div class="message-sender"><%= auth.email() %></div>
  <div class="message-content"><%= message %></div>
  <div class="message-timestamp"><%= new Date().toLocaleTimeString() %></div>
</div>
```

#### Using PocketBase JSVM

Create a custom API endpoint in your PocketBase app:

```javascript
routerAdd('POST', '/api/chat', (c) => {
  const data = $apis.requestInfo(c).data
  const message = data.message

  // Get all clients from the subscription broker
  const clients = $app.subscriptionsBroker().clients()

  // Filter clients that are subscribed to the 'chat' topic
  Object.entries(clients)
    .filter(([_, client]) => client.hasSubscription('chat'))
    .forEach(([_, client]) => {
      // Send the message to each subscribed client
      client.send({
        name: 'chat',
        data: `<div>${message}</div>`,
      })
    })

  return c.json({ result: 'ok' })
})
```

This implementation:

1. Gets all connected clients from PocketBase's subscription broker
2. Filters for clients that are subscribed to the 'chat' topic
3. Sends the formatted message directly to each subscribed client
4. Returns a success response

### SSE Message Format

When you connect to the realtime endpoint, you'll receive messages in this format:

```
id:NJMhEaQbJ4lJ7L5QIsenCCG8wzdH8mETTp2ncclX
event:PB_CONNECT
data:{"clientId":"NJMhEaQbJ4lJ7L5QIsenCCG8wzdH8mETTp2ncclX"}

id:NJMhEaQbJ4lJ7L5QIsenCCG8wzdH8mETTp2ncclX
event:chat
data:<div>asdf</div>

id:NJMhEaQbJ4lJ7L5QIsenCCG8wzdH8mETTp2ncclX
event:chat
data:"<div>\n  <p>This is a multiline message</p>\n  <p>That was JSON encoded</p>\n</div>"
```

The extension handles:

1. The initial `PB_CONNECT` event to establish the connection
2. Automatic subscription to your specified topics
3. Swapping the `data` content into your elements based on the matching `event` name

> **Note:** The extension will attempt to parse any JSON-parsable strings in the `data` field. This is particularly useful for multiline content, which should be JSON encoded to properly fit within a single line in the SSE data stream. If parsing fails (i.e., the content is not valid JSON), the original string is used.

### Configuration Attributes

- `hx-ext="pocketbase-sse"` - Activates the SSE extension
- `sse-swap="topic"` - Defines which topic(s) to subscribe to (comma-separated for multiple topics)
- `hx-swap="beforeend"` - (Optional) Controls how new content is inserted

## How It Works

1. When the extension initializes, it establishes a connection to PocketBase's realtime API
2. Upon successful connection (`PB_CONNECT` event), it automatically subscribes to the specified topics
3. Incoming messages on the subscribed topics trigger content updates according to the configured swap behavior

## Events

This extension uses the same event names as the official HTMX SSE extension for compatibility:

- `htmx:sseOpen` - Triggered when the SSE connection is established
- `htmx:sseMessage` - Triggered when a message is received
- `htmx:sseError` - Triggered when an error occurs
- `htmx:sseClose` - Triggered when the connection is closed

You can use these events in your triggers and event handlers just like with the standard SSE extension.

## Example with Multiple Topics

```html
<div hx-ext="pocketbase-sse">
  <!-- Chat messages -->
  <div sse-swap="chat" hx-swap="beforeend">
    <!-- Chat messages will append here -->
  </div>

  <!-- User status updates -->
  <div sse-swap="users" hx-swap="innerHTML">
    <!-- User status will replace content here -->
  </div>
</div>
```

## Advanced Usage

### Custom Event Handling

You can listen for specific SSE events:

```html
<div hx-ext="pocketbase-sse" sse-swap="chat" hx-trigger="sse:message">
  <!-- Handle specific message events -->
</div>
```

### Error Handling

The extension automatically handles connection errors and reconnection attempts. You can style disconnected states using the `sse-disconnected` class that's added to the parent element when connection is lost.

```css
.sse-disconnected {
  opacity: 0.5;
}
```

## License

MIT

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
