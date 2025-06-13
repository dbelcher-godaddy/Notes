# CaaS-WebUI: Initial Notes

## Links

https://github.com/gdcorp-dna/caas-open-webui

https://github.com/gdcorp-dna/caas-open-webui-pipelines

https://github.com/gdcorp-dna/open-webui

https://caas.open-webui.dev-godaddy.com/workspace/models

## Demo(s)

https://secureservernet-my.sharepoint.com/:v:/g/personal/aaggarwal_godaddy_com/EUP-trI8qblHvlPB9FvBU0sBqxxksbdPqRDolxnd46MvsA?e=q6Trrk

https://secureservernet-my.sharepoint.com/:v:/g/personal/aaggarwal_godaddy_com/ERDqlXAhqGZPjC5iIxqxUfIB9_LKWmOSHY6JaYTx9Ei4ng?e=ZZiG73

https://secureservernet-my.sharepoint.com/:v:/g/personal/aaggarwal_godaddy_com/ETUhztnTY8lKqxcamBTn6yoBwrtMG9PE7SxdmDHz9Q3vgQ?e=3uKcqM

## Customization

Depending upon what you whether you want to customize the frontend or backend, there are several options available.

To truly customize the frontend, it appears the only way to do so would be to fork the open-source repo and modify the UI through
local development.

### [Functions](https://docs.openwebui.com/features/plugin/functions/)

Three main types: Pipe, Filter, and Action. They allow you to extend capabilities like adding support for new AI model providers,
tweaking how messages are processed, or introducing custom buttons.

#### Pipe

A Pipe Function is how you create custom agents/models or integrations, which then appear in the interface as if they were
standalone models.

#### Filter

A Filter Function is like a tool for tweaking data before it gets sent to the AI or after it comes back.

#### Action

An Action Function is used to add custom buttons to the chat interface.

### [Pipelines](https://docs.openwebui.com/pipelines/)

Pipelines is a modular framework designed to enhance workflows for any UI client compatible with OpenAI API specifications. It is
particularly useful for handling computationally intensive tasks, such as running large models or implementing complex logic, by
offloading them from the main OpenWebUI instance. This improves performance and scalability.

#### Filters

Filters are used to perform actions against incoming user messages and outgoing assistant (LLM) messages.

#### Pipes

Pipes are standalone functions that process inputs and generate responses, possibly by invoking one or more LLMs or external
services before returning results to the user.

### [Development](https://docs.openwebui.com/getting-started/advanced-topics/development)

By forking the WebUI repo, we can modify other aspects of the frontend and possibly contribute back to the open-source project.

The downside to this is that we must manage merging future updates ourselves for anything not contributed back to open-source.

Uses [SVELTE](https://svelte.dev/).

### [Restrictions](https://docs.openwebui.com/license#open-webui-license-explained)

You may NOT alter, remove, or obscure any “Open WebUI” branding (name, logo, UI marks, etc.) in any deployment or distribution,
except in the circumstances below.
Branding must remain clearly visible, unless:

- You have 50 or fewer users in a 30-day period;
- You are a contributor, and have gotten written permission from us;
- You’ve secured an enterprise license from us which explicitly allows branding changes.

## Feature Parity

### SDM

SDM is a foreign concept to WebUI and not natively supported.

#### Basic understanding of how it works today.

- Users must be authorized to use SDM via localized permissions within CaaS.
- Only certain models support SDM, and when enabled, only those models are viewable within the UI.
- Authorized users can enable SDM for newly created thread conversations or newly created agents (which will apply SDM to all
  conversations with that agent), and cannot be enabled/disabled after initial creation.
- A special parameter is sent to the Agents/CaaS API's specifying the agent/thread as having SDM enabled/disabled for the current
  conversation.
  - If the agent/thread conversation indicates SDM is enabled, the Agent/CaaS API will isolate conversation messages in S3 storage
    apart from normal conversation messages.

#### How "could" it be supported within WebUI?

- Apart from local development by forking the open-source repo and adding custom UI, I see no way to add additional options to
  enable/disable SDM mode globally.
  - If we could somehow enable/disable SDM globally:
    - I see no way of limiting the selectable models if SDM was enabled globally.
    - I am unsure if the setting could be passed to functions/pipelines for special treatment and storage considerations.
    - I am also unsure of a way we can modify where conversation history is stored and if we could ensure isolation of SDM
      conversations from normal conversations.
- We could define new models that would be considered "always SDM".
  - Such that if a user selects Model-A they would be interacting with the normal Model-A. And if they selected Model-A-SDM, they
    would be interacting with a specialized copy of Model-A that considers SDM to always be enabled.
  - We could restrict the "*-SDM" models to only be accessible from authorized users.
  - However, we would still have issues with storage isolation of conversation history.
- We could deploy a separate instance of WebUI specific to SDM.
  - For example, unauthorized users or those that do not wish to use SDM could visit: https://caas.open-webui.dev-godaddy.com
    - Authorized users who wish to use SDM would need to visit: https://caas.open-webui-sdm.dev-godaddy.com
  - This would solve the issue of conversation isolation, but would mean we have to maintain an additional site.

### Cost Aggregation

Today, cost aggregation is handled within CaaS-API and since the CaaS-UI uses this as it's backend, it's handled automatically for
us.

We could implement the same logic within GoCode if all LLM communications are guaranteed to go through that service; or if not, we
could instead add a Function/Pipeline to store costs within DDB like we do today.

### Sharing

#### Conversations

- Conversations can be shared with other users by creating a dynamic link and sending it manually to the person you wish to share
  with.
  - This link creates a read-only snapshot of the conversation up to the point where the link is created.
  - The link can be updated or revoked at any time by the original author.
- There doesn't appear to be any way to change the share link to something other than read-only.
- There doesn't appear to be any way to share directly with a person, or group within the UI.

#### Tools

Allows share settings Public/Private and Group

#### Prompts (aka Example)

Allows share settings Public/Private and Group

#### Knowledge (aka Library)

Allows share settings Public/Private and Group

#### Models (aka Companion)

Allows share settings Public/Private and Group

## Open Questions

- Are we sticking to opensource or will we be acquiring a license to allow full customization?
- Are we directing all LLM connections through GoCode or will we be connecting to some LLM's directly if they support the OpenAI
  API spec?
- We need to identify what models will be supported for initial release, and how they will be connected.
- How/Where are conversation histories stored? Are they accessible outside of WebUI?
- Sensitive Content/Moderation
  - I don't see support for this, or at least no support the way it's being done in CaaS-Web




































