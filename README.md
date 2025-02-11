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

### Backend Implementation (PocketPages)

Create an endpoint at `/api/chat.ejs`:

```javascript
<script server>
  // Send a message to all clients subscribed to the 'chat' topic
  send('chat', `<div>hello</div>`)
  return { result: 'ok' }
</script>
```

This endpoint:

1. Receives POST requests from the form
2. Uses PocketPages' `send()` function to broadcast to the 'chat' topic
3. Returns a success response (the form uses `hx-swap="none"` so the response isn't rendered)

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
