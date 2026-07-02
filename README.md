# Genesys Cloud Web Messaging — Debug Client

A single-file, dependency-free browser client for exercising and debugging the
[Genesys Cloud Web Messaging](https://developer.genesys.cloud/commdigital/digital/webmessaging/) WebSocket API.
No build step, no server, no npm install — open the HTML file in a browser and go.

## Usage

Open [`webmessagingclientStandalone.html`](webmessagingclientStandalone.html) directly in a browser (double-click, or `File > Open`).

1. Pick a **Region** and enter your **Deployment ID**.
2. Click **Connect**.
3. Send messages, attachments, and presence events like a guest chat widget would.

Region and Deployment ID are saved to `localStorage` and restored on the next visit.

> Note: `localStorage` persistence can behave inconsistently when opening the file directly via `file://`
> in some browsers. For fully consistent persistence, serve the file from a local static server instead.

## Features

- **Connect / Disconnect** with a live status indicator (Disconnected, Connecting, Connected, Connection error).
  The Region and Deployment ID fields lock while connected/connecting to avoid editing values that
  wouldn't take effect until reconnecting.
- **Text messaging**, typing indicator, and Presence Join / auto Presence-Clear on an idle timer.
- **File attachments** via the presigned-URL upload flow.
- **Debug mode** (toggle in the top controls) reveals additional panels for deeper inspection:
  - **API token** field — a Genesys Cloud OAuth bearer token, used only to call the Platform API below.
    This is separate from the anonymous guest session used for the WebSocket connection itself.
  - **Message history** — a collapsible log of every sent message, with round-trip timing and the
    resolved Conversation ID. Collapsed entries show just the send time and content on one line.
  - **Attribute Search** — looks up participant custom attributes for the current conversation via the
    Conversations API.
  - **Custom Attributes editor** — attach arbitrary `channel.metadata.customAttributes` key/value pairs
    to outgoing messages, to verify they propagate correctly.

The layout fills the browser window; the message list and side panels scroll independently so the
input bar and send/attach controls stay visible regardless of how much debug content accumulates.

## How it works

- **WebSocket session**: connects to `wss://webmessaging.<region>/v1?deploymentId=<id>` and speaks the
  Web Messaging protocol (`configureSession`, `onMessage`, `onAttachment`, presence/typing events).
- **Conversation lookup** (debug mode only): when the server acknowledges a sent message, the client
  calls `GET /api/v2/conversations/messages/{messageId}/details` on the Genesys Cloud Platform API to
  resolve the Conversation ID, which in turn powers Attribute Search
  (`POST /api/v2/conversations/participants/attributes/search`). This requires the separate OAuth
  bearer token described above — the guest WebSocket session has no permission to call the Platform API.

## Supported regions

Americas (N. Virginia, N. Virginia FedRAMP, Oregon, Montreal, São Paulo), EMEA (Dublin, London,
Frankfurt, Zurich), Asia Pacific (Mumbai, Seoul, Osaka, Tokyo, Sydney), and Middle East (UAE).

## Security notes

- The API token field is never persisted — it lives only in memory for the current page session.
- The Deployment ID field ships empty; you must enter your own before connecting. Once entered, it's
  saved to `localStorage` on your machine only — it's never written back into this file. Treat your
  Deployment ID and any Platform API tokens as you would any other environment-specific credential
  when sharing or publishing this repo.
