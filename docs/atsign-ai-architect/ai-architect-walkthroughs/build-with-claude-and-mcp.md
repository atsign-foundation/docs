---
icon: mcp
---

# Using Claude with the MCP Server

The **Model Context Protocol (MCP)** is a standard that allows AI models like Claude to securely connect to external tools, services, and data sources. In AI Architect, MCP is what enables Claude to communicate with the **Atsign MCP server** so it can generate blueprints, fetch information, and perform actions based on your instructions.

By connecting Claude to the AI Architect MCP server, you give it controlled access to the tools it needs to create your blueprint directly on the canvas.

{% tabs %}
{% tab title="Claude Desktop" %}
### Connect the AI Architect MCP (Claude Desktop)

Follow these steps to connect Claude Desktop to the Atsign AI Architect MCP server.

#### **1. Open Claude Desktop**

Launch the Claude Desktop application on your device.

#### **2. Open the Connectors menu**

In the left sidebar, click **Customize**, then select **Connectors**.

#### **3. Add a new custom connector**

Click the **+** icon and choose **Add Custom Connector**.

#### **4. Enter the connector details**

You will be prompted to configure the connector:

* **Name:** Enter a name such as _AI Architect MCP_
* **Remote MCP Server URL:** Paste the following URL:

```
https://aiarchitect.atsign.com/mcp
```

* Click **Add** to save the connector.

#### **5. (Optional) Set tool permissions**

Claude may ask for permission to use the connector’s tools. Select **Always allow** for a smoother workflow.

#### **6. Start a new chat**

Click **New Chat** in Claude Desktop.

#### **7. Enable the connector**

Inside the new chat:

* Click the **+** icon
* Select **Connectors**
* Toggle on your newly added **AI Architect MCP** connector

#### **8. Copy your Session ID**

A Session ID will be displayed in the setup dialog. Click **Copy** to copy it to your clipboard.

#### **9. Include the Session ID in your prompt**

Paste the Session ID into your prompt to Claude and explicitly mention that you want an **“Atsign AI Architect blueprint”** so Claude knows to use the MCP server.

Example:

{% code overflow="wrap" %}
```
I want to create an AI Architect blueprint for an IT Helpdesk app. My session ID is: stream-FUW1BbuQ9BTT8cxR5Vv65B
```
{% endcode %}

#### **10. Establish the connection**

Click **Establish Connection** to activate the MCP session. Your blueprint will be generated and displayed on the canvas.
{% endtab %}

{% tab title="Claude Browser" %}
### Connect the AI Architect MCP (Claude Browser)

Follow these steps to connect Claude in the browser to the Atsign AI Architect MCP server.

#### **1. Open Claude.ai**

In your browser, open a new tab and go to [**https://claude.ai**](https://claude.ai).

#### **2. Open the Connectors menu**

In the left sidebar, click **Customize**, then select **Connectors**.

#### **3. Add a new custom connector**

Click the **+** icon and choose **Add Custom Connector**.

#### **4. Enter the connector details**

You will be prompted to configure the connector:

* **Name:** Enter a name such as _AI Architect MCP_
* **Remote MCP Server URL:** Paste the following URL:

```
https://aiarchitect.atsign.com/mcp
```

* Click **Add** to save the connector.

#### **5. (Optional) Set tool permissions**

Claude may ask for permission to use the connector’s tools. Select **Always allow** for a smoother workflow.

#### **6. Start a new chat**

Click **New Chat** in Claude Desktop.

#### **7. Enable the connector**

Inside the new chat:

* Click the **+** icon
* Select **Connectors**
* Toggle on your newly added **AI Architect MCP** connector

#### **8. Copy your Session ID**

A Session ID will be displayed in the setup dialog. Click **Copy** to copy it to your clipboard.

#### **9. Include the Session ID in your prompt**

Paste the Session ID into your prompt to Claude and explicitly mention that you want an **“Atsign AI Architect blueprint”** so Claude knows to use the MCP server.

Example:

{% code overflow="wrap" %}
```
I want to create an AI Architect blueprint for an IT Helpdesk app. My session ID is: stream-FUW1BbuQ9BTT8cxR5Vv65B
```
{% endcode %}

#### **10. Establish the connection**

Click **Establish Connection** to activate the MCP session. Your blueprint will be generated and displayed on the canvas.
{% endtab %}
{% endtabs %}

### FAQs

<details>

<summary>What is MCP?</summary>

MCP is a protocol that lets Claude securely connect to external tools. In AI Architect, it allows

</details>

<details>

<summary>Why do I need to connect Claude to the MCP server?</summary>

Connecting Claude to the MCP server gives it access to the tools required to create and update your blueprint automatically.

</details>

<details>

<summary>Where do I find the MCP server URL?</summary>

The URL is shown in the setup dialog inside AI Architect. You’ll paste it into Claude when adding a custom connector.

{% code overflow="wrap" %}
```
https://aiarchitect.atsign.com/mcp
```
{% endcode %}

</details>

<details>

<summary>What is the Session ID for?</summary>

The Session ID links your Claude chat to your active AI Architect session. Claude uses it to generate your blueprint in the correct workspace.

</details>
