# Claude in Microsoft Foundry

Access Claude models through Microsoft Foundry with Azure-native endpoints,
identity, networking, billing, and monitoring.

Claude usage is billed through Azure Marketplace in Claude Consumption Units
(CCUs). Azure-hosted Claude deployments support Global Standard and US Data
Zone Standard deployment types.

## Hosting options

Claude models in Microsoft Foundry are offered with two hosting options. Select
the hosting option while configuring the deployment.

| Area | Hosted on Azure | Hosted on Anthropic |
| --- | --- | --- |
| Where inference runs | Anthropic-operated service on Azure infrastructure | Anthropic-operated service on Anthropic infrastructure |
| Model availability | Latest supported models in the Opus, Sonnet, and Haiku families | All Claude models offered through Microsoft Foundry |
| Deployment types | Global Standard and US Data Zone Standard | Global Standard |
| Recommended for | Most workloads | [Models or features that are not yet available with Azure hosting](supported-features.md#additional-features-not-supported-when-hosted-on-azure) |

!!! note

    Anthropic acts as an independent processor for Microsoft. Customers using
    Claude through Microsoft Foundry are subject to Anthropic's data use terms.
    For deployments hosted on Azure, prompts and completions remain within
    Azure. Only usage metadata and content flagged by Anthropic's safety systems
    egress to Anthropic. Anthropic continues to provide its safety and data
    commitments.

## Prerequisites

Before deploying Claude, ensure that you have:

- An active, eligible Azure subscription
- Access to the [Microsoft Foundry portal](https://ai.azure.com/)
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) for
  command-line authentication and deployment validation
- An Azure RBAC role that permits model inference, such as **Foundry User** or
  **Cognitive Services User**

This starter kit also requires Azure Developer CLI (`azd`) and uses Microsoft
Entra ID for passwordless access.

## Install an SDK

Anthropic client SDKs support Microsoft Foundry through a platform-specific
package or client class.

!!! note

    Foundry is supported natively by the C#, Java, PHP, Python, and TypeScript
    SDKs. The Go and Ruby SDKs do not currently provide native Foundry clients;
    use their standard clients with the Foundry base URL as a workaround.

=== "Python"

    ```powershell
    pip install --upgrade anthropic

    # Also required for Microsoft Entra ID authentication
    pip install azure-identity
    ```

=== "TypeScript"

    ```powershell
    npm install @anthropic-ai/foundry-sdk

    # Also required for Microsoft Entra ID authentication
    npm install @azure/identity
    ```

=== "C#"

    ```powershell
    dotnet add package Anthropic.Foundry
    ```

=== "Go"

    ```powershell
    # The Go SDK does not yet support Foundry natively.
    go get github.com/anthropics/anthropic-sdk-go
    ```

=== "Java"

    Gradle:

    ```kotlin
    implementation("com.anthropic:anthropic-java:2.65.0")
    implementation("com.anthropic:anthropic-java-foundry:2.65.0")

    // Also required for Microsoft Entra ID authentication
    implementation("com.azure:azure-identity:1.18.3")
    ```

    Maven:

    ```xml
    <dependency>
        <groupId>com.anthropic</groupId>
        <artifactId>anthropic-java</artifactId>
        <version>2.65.0</version>
    </dependency>
    <dependency>
        <groupId>com.anthropic</groupId>
        <artifactId>anthropic-java-foundry</artifactId>
        <version>2.65.0</version>
    </dependency>
    <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-identity</artifactId>
        <version>1.18.3</version>
    </dependency>
    ```

=== "PHP"

    ```powershell
    composer require "anthropic-ai/sdk" "guzzlehttp/guzzle:^7"
    ```

=== "Ruby"

    ```ruby
    # The Ruby SDK does not yet support Foundry natively.
    # Add this dependency to your Gemfile.
    gem "anthropic"
    ```

## Provisioning

Microsoft Foundry uses two levels:

- A **Foundry resource** contains security, networking, and billing
  configuration.
- A **model deployment** is the named Claude model instance called by an
  application.

Create the Foundry resource first, then add one or more Claude deployments.

### Provision a Foundry resource

1. Open the [Microsoft Foundry portal](https://ai.azure.com/).
2. Create a Foundry resource or select an existing resource.
3. Configure Microsoft Entra ID and Azure RBAC for access control.
4. Optionally connect the resource to an Azure virtual network to restrict
   network access.
5. Record the resource name. It forms part of the API endpoint:

   ```text
   https://{resource}.services.ai.azure.com/anthropic/v1/*
   ```

### Deploy Claude models

Follow the current
[Microsoft Learn deployment guidance](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/how-to/use-foundry-models-claude?tabs=python#deploy-claude-models):

1. Sign in to [Microsoft Foundry](https://ai.azure.com/?cid=learnDocs) and make
   sure the **New Foundry** toggle is on.
2. From the portal home page, select **Discover** in the upper-right
   navigation, then select **Models** in the left pane.
3. Select a Claude model and review its model card. When both hosting versions
   are available, Foundry opens **Hosted on Azure** (version 2) by default.
   Confirm the selected version in the card's **Quick facts** pane.
4. Select **Deploy** > **Custom settings**.
5. Review the Azure Marketplace terms, select the appropriate industry, and
   select **Agree and Proceed** to subscribe to the offer.
6. Select the **Model version**:
    - <span style="color: #d32f2f;"><strong>2: Hosted on Azure</strong></span>
      is selected by default when both versions are available. Inference runs
      on Azure infrastructure, and supported models can use Global Standard or
      US Data Zone Standard deployments.
    - <span style="color: #d32f2f;"><strong>1: Hosted on Anthropic infrastructure</strong></span>
      can be selected when that hosting option is required. Inference runs on
      Anthropic infrastructure and uses the Global Standard deployment type.
7. Configure the remaining deployment settings:
    - **Deployment name:** Keep the model name or enter a custom deployment
      name. Use this deployment name in the API `model` parameter.
    - **Region scope:** Select **Global**, or select **Data Zone** when it is
      available for the selected model and version.
8. Select **Deploy**.
9. After deployment, use the Foundry playground to test the model
   interactively.
10. Open the **Details** tab, verify the deployment configuration, and confirm
    that its status is **Succeeded**.

Selecting **Deploy** > **Default settings** also creates version
**2: Hosted on Azure** when the selected model supports both hosting versions.
The Foundry project and resource must be in a region supported by the selected
model and deployment type.

!!! important

    The deployment name becomes the value supplied in the API `model`
    parameter. Multiple deployments of the same model can use different names
    to separate versions, configurations, or rate limits.

## Authentication

Azure-hosted Claude endpoints support Azure-issued API keys and Microsoft Entra
ID tokens. This starter kit uses Microsoft Entra ID so credentials do not need
to be stored or rotated as API keys.

All requests use an endpoint in this format:

```text
https://{resource}.services.ai.azure.com/anthropic/v1/*
```

### Microsoft Entra ID authentication

Microsoft Entra ID integrates model access with Azure RBAC and organizational
identity controls.

1. Enable Microsoft Entra ID authentication for the Foundry resource.
2. Assign the calling identity **Foundry User**, **Cognitive Services User**, or
   another suitable least-privilege role.
3. Use `DefaultAzureCredential` to obtain tokens for the
   `https://ai.azure.com/.default` scope.

=== "cURL"

    ```powershell
    $token = az account get-access-token `
      --resource https://ai.azure.com `
      --query accessToken `
      --output tsv

    curl.exe https://example-resource.services.ai.azure.com/anthropic/v1/messages `
      -H "content-type: application/json" `
      -H "Authorization: Bearer $token" `
      -H "anthropic-version: 2023-06-01" `
      -d '{\"model\":\"claude-opus-5-5\",\"max_tokens\":1024,\"messages\":[{\"role\":\"user\",\"content\":\"Hello!\"}]}'
    ```

=== "CLI"

    ```text
    # The ant CLI can send a bearer token with --auth-token, but a set
    # ANTHROPIC_API_KEY environment variable takes precedence over it (the CLI
    # prints only a console notice), so your request could authenticate with
    # the wrong credential. For the Entra ID flow, use the cURL example or one
    # of the SDK examples instead.
    ```

=== "Python"

    ```python
    from anthropic import AnthropicFoundry
    from azure.identity import DefaultAzureCredential, get_bearer_token_provider

    token_provider = get_bearer_token_provider(
        DefaultAzureCredential(),
        "https://ai.azure.com/.default",
    )

    client = AnthropicFoundry(
        resource="example-resource",
        azure_ad_token_provider=token_provider,
    )

    message = client.messages.create(
        model="claude-opus-5-5",
        max_tokens=1024,
        messages=[{"role": "user", "content": "Hello!"}],
    )

    print(message.content)
    ```

=== "TypeScript"

    ```typescript
    import AnthropicFoundry from "@anthropic-ai/foundry-sdk";
    import {
      DefaultAzureCredential,
      getBearerTokenProvider
    } from "@azure/identity";

    const credential = new DefaultAzureCredential();
    const tokenProvider = getBearerTokenProvider(
      credential,
      "https://ai.azure.com/.default"
    );

    const client = new AnthropicFoundry({
      resource: "example-resource",
      azureADTokenProvider: tokenProvider
    });

    const message = await client.messages.create({
      model: "claude-opus-5-5",
      max_tokens: 1024,
      messages: [{ role: "user", content: "Hello!" }]
    });

    console.log(message.content);
    ```

=== "C#"

    ```csharp
    using Anthropic.Foundry;
    using Anthropic.Models.Messages;
    using Azure.Identity;

    var client = new AnthropicFoundryClient(
        new AnthropicFoundryIdentityTokenCredentials(
            new DefaultAzureCredential(),
            "example-resource"
        )
    );

    var response = await client.Messages.Create(new MessageCreateParams
    {
        Model = "claude-opus-5-5",
        MaxTokens = 1024,
        Messages = [new() { Role = Role.User, Content = "Hello!" }],
    });

    Console.WriteLine(string.Join(
        "",
        response.Content
            .Select(block => block.Value)
            .OfType<TextBlock>()
            .Select(block => block.Text)
    ));
    ```

=== "Go"

    ```go
    package main

    import (
        "context"
        "fmt"
        "os"

        "github.com/anthropics/anthropic-sdk-go"
        "github.com/anthropics/anthropic-sdk-go/option"
    )

    func main() {
        client := anthropic.NewClient(
            option.WithoutEnvironmentDefaults(),
            option.WithBaseURL(
                "https://example-resource.services.ai.azure.com/anthropic",
            ),
            option.WithAuthToken(os.Getenv("AZURE_ACCESS_TOKEN")),
        )

        message, err := client.Messages.New(
            context.Background(),
            anthropic.MessageNewParams{
                Model:     "claude-opus-5-5",
                MaxTokens: 1024,
                Messages: []anthropic.MessageParam{
                    anthropic.NewUserMessage(
                        anthropic.NewTextBlock("Hello!"),
                    ),
                },
            },
        )
        if err != nil {
            panic(err)
        }

        fmt.Println(message.Content)
    }
    ```

=== "Java"

    ```java
    import com.anthropic.client.AnthropicClient;
    import com.anthropic.client.okhttp.AnthropicOkHttpClient;
    import com.anthropic.foundry.backends.FoundryBackend;
    import com.anthropic.models.messages.MessageCreateParams;
    import com.azure.identity.AuthenticationUtil;
    import com.azure.identity.DefaultAzureCredentialBuilder;
    import java.util.function.Supplier;

    Supplier<String> tokenSupplier =
        AuthenticationUtil.getBearerTokenSupplier(
            new DefaultAzureCredentialBuilder().build(),
            "https://ai.azure.com/.default"
        );

    AnthropicClient client = AnthropicOkHttpClient.builder()
        .backend(FoundryBackend.builder()
            .bearerTokenSupplier(tokenSupplier)
            .resource("example-resource")
            .build())
        .build();

    MessageCreateParams params = MessageCreateParams.builder()
        .model("claude-opus-5-5")
        .maxTokens(1024)
        .addUserMessage("Hello!")
        .build();

    client.messages().create(params);
    ```

=== "PHP"

    ```php
    <?php

    use Anthropic\Foundry;

    $client = Foundry\Client::withCredentials(
        authToken: getenv('AZURE_ACCESS_TOKEN'),
        baseUrl: 'https://example-resource.services.ai.azure.com/anthropic',
    );

    $message = $client->messages->create(
        model: 'claude-opus-5-5',
        maxTokens: 1024,
        messages: [['role' => 'user', 'content' => 'Hello!']],
    );
    ```

=== "Ruby"

    ```ruby
    require "anthropic"

    client = Anthropic::Client.new(
      base_url: "https://example-resource.services.ai.azure.com/anthropic",
      auth_token: ENV.fetch("AZURE_ACCESS_TOKEN")
    )

    message = client.messages.create(
      model: "claude-opus-5-5",
      max_tokens: 1024,
      messages: [{ role: "user", content: "Hello!" }]
    )

    puts message.content.find { it.type == :text }.text
    ```

`DefaultAzureCredential` supports local Azure CLI credentials and managed
identity in Azure-hosted applications. The token provider refreshes tokens for
long-running Python, TypeScript, C#, and Java processes. The Go, PHP, and Ruby
examples use a static `AZURE_ACCESS_TOKEN`; those applications must refresh it
before it expires.

### API key authentication

API key authentication is available in Foundry, but Microsoft Entra ID is
recommended for this starter kit. If an existing application requires an API
key:

1. In the Foundry portal, open **Build** > **Models**.
2. Select the Claude deployment and open **Details**.
3. Copy the key and target URI.
4. Store the key in a secret store or untracked environment variable.
5. Send the key in the `api-key` or `x-api-key` request header.

The Foundry SDK recognizes:

- `ANTHROPIC_FOUNDRY_API_KEY` for the Azure-issued API key
- `ANTHROPIC_FOUNDRY_RESOURCE` for the Foundry resource name
- `ANTHROPIC_FOUNDRY_BASE_URL` for a complete Foundry base URL

The resource name and base URL are alternatives; configure only one.

=== "cURL"

    ```powershell
    curl.exe https://example-resource.services.ai.azure.com/anthropic/v1/messages `
      -H "content-type: application/json" `
      -H "api-key: $env:ANTHROPIC_FOUNDRY_API_KEY" `
      -H "anthropic-version: 2023-06-01" `
      -d '{\"model\":\"claude-opus-5-5\",\"max_tokens\":1024,\"messages\":[{\"role\":\"user\",\"content\":\"Hello!\"}]}'
    ```

=== "CLI"

    ```powershell
    $env:ANTHROPIC_API_KEY = $env:ANTHROPIC_FOUNDRY_API_KEY

    ant messages create `
      --base-url https://example-resource.services.ai.azure.com/anthropic `
      --model claude-opus-5-5 `
      --max-tokens 1024 `
      --message '{role: user, content: "Hello!"}' `
      --transform content
    ```

=== "Python"

    ```python
    import os

    from anthropic import AnthropicFoundry

    client = AnthropicFoundry(
        api_key=os.environ["ANTHROPIC_FOUNDRY_API_KEY"],
        resource="example-resource",
    )

    message = client.messages.create(
        model="claude-opus-5-5",
        max_tokens=1024,
        messages=[{"role": "user", "content": "Hello!"}],
    )

    print(message.content)
    ```

=== "TypeScript"

    ```typescript
    import AnthropicFoundry from "@anthropic-ai/foundry-sdk";

    const client = new AnthropicFoundry({
      apiKey: process.env.ANTHROPIC_FOUNDRY_API_KEY,
      resource: "example-resource"
    });

    const message = await client.messages.create({
      model: "claude-opus-5-5",
      max_tokens: 1024,
      messages: [{ role: "user", content: "Hello!" }]
    });

    console.log(message.content);
    ```

=== "C#"

    ```csharp
    using Anthropic.Foundry;
    using Anthropic.Models.Messages;

    var client = new AnthropicFoundryClient(
        new AnthropicFoundryApiKeyCredentials(
            Environment.GetEnvironmentVariable("ANTHROPIC_FOUNDRY_API_KEY")!,
            "example-resource"
        )
    );

    var response = await client.Messages.Create(new MessageCreateParams
    {
        Model = "claude-opus-5-5",
        MaxTokens = 1024,
        Messages = [new() { Role = Role.User, Content = "Hello!" }],
    });

    Console.WriteLine(string.Join(
        "",
        response.Content
            .Select(block => block.Value)
            .OfType<TextBlock>()
            .Select(block => block.Text)
    ));
    ```

=== "Go"

    ```go
    package main

    import (
        "context"
        "fmt"
        "os"

        "github.com/anthropics/anthropic-sdk-go"
        "github.com/anthropics/anthropic-sdk-go/option"
    )

    func main() {
        client := anthropic.NewClient(
            option.WithoutEnvironmentDefaults(),
            option.WithBaseURL(
                "https://example-resource.services.ai.azure.com/anthropic",
            ),
            option.WithAPIKey(os.Getenv("ANTHROPIC_FOUNDRY_API_KEY")),
        )

        message, err := client.Messages.New(
            context.Background(),
            anthropic.MessageNewParams{
                Model:     "claude-opus-5-5",
                MaxTokens: 1024,
                Messages: []anthropic.MessageParam{
                    anthropic.NewUserMessage(
                        anthropic.NewTextBlock("Hello!"),
                    ),
                },
            },
        )
        if err != nil {
            panic(err)
        }

        fmt.Println(message.Content)
    }
    ```

=== "Java"

    ```java
    import com.anthropic.client.AnthropicClient;
    import com.anthropic.client.okhttp.AnthropicOkHttpClient;
    import com.anthropic.foundry.backends.FoundryBackend;
    import com.anthropic.models.messages.MessageCreateParams;

    AnthropicClient client = AnthropicOkHttpClient.builder()
        .backend(FoundryBackend.fromEnv())
        .build();

    MessageCreateParams params = MessageCreateParams.builder()
        .model("claude-opus-5-5")
        .maxTokens(1024)
        .addUserMessage("Hello!")
        .build();

    client.messages().create(params);
    ```

    Set `ANTHROPIC_FOUNDRY_API_KEY` and `ANTHROPIC_FOUNDRY_RESOURCE` before
    running the example.

=== "PHP"

    ```php
    <?php

    use Anthropic\Foundry;

    $client = Foundry\Client::withCredentials(
        apiKey: getenv('ANTHROPIC_FOUNDRY_API_KEY'),
        baseUrl: 'https://example-resource.services.ai.azure.com/anthropic',
    );

    $message = $client->messages->create(
        model: 'claude-opus-5-5',
        maxTokens: 1024,
        messages: [['role' => 'user', 'content' => 'Hello!']],
    );
    ```

=== "Ruby"

    ```ruby
    require "anthropic"

    client = Anthropic::Client.new(
      base_url: "https://example-resource.services.ai.azure.com/anthropic",
      api_key: ENV.fetch("ANTHROPIC_FOUNDRY_API_KEY")
    )

    message = client.messages.create(
      model: "claude-opus-5-5",
      max_tokens: 1024,
      messages: [{ role: "user", content: "Hello!" }]
    )

    puts message.content.find { it.type == :text }.text
    ```

!!! warning

    Never commit an API key to source control. Prefer Microsoft Entra ID for
    production workloads and store any required key in Azure Key Vault or
    another approved secret store.

## Correlation request IDs

Foundry response headers include identifiers for tracing and support:

- `request-id`
- `apim-request-id`

Record both values when investigating a failed request or contacting support.
Together, they help trace the request through Anthropic and Azure API
Management.

## API model IDs and deployments

Microsoft Foundry follows the Claude API model lifecycle. Review
[model deprecations](https://platform.claude.com/docs/en/about-claude/model-deprecations)
before adopting or upgrading a model.

The official documentation listed the following models on September 26, 2026.

| Model | Default deployment name | Hosted on Azure | Hosted on Anthropic |
| --- | --- | :---: | :---: |
| Claude Fable 5.1 | `claude-fable-5-1` | — | ✓ |
| Claude Mythos 5.1 ([limited availability](https://anthropic.com/glasswing)) | `claude-mythos-5-1` | — | ✓ |
| Claude Fable 5 | `claude-fable-5` | — | ✓ |
| Claude Mythos 5 ([limited availability](https://anthropic.com/glasswing)) | `claude-mythos-5` | — | ✓ |
| Claude Opus 5.5 | `claude-opus-5-5` | ✓ | ✓ |
| Claude Opus 5 | `claude-opus-5` | ✓ | ✓ |
| Claude Opus 4.8 | `claude-opus-4-8` | ✓ | ✓ |
| Claude Opus 4.7 | `claude-opus-4-7` | — | ✓ |
| Claude Opus 4.6 | `claude-opus-4-6` | — | ✓ |
| Claude Opus 4.5 | `claude-opus-4-5` | — | ✓ |
| Claude Sonnet 5 | `claude-sonnet-5` | ✓ | ✓ |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | — | ✓ |
| Claude Sonnet 4.5 | `claude-sonnet-4-5` | — | ✓ |
| Claude Haiku 4.5 | `claude-haiku-4-5` | ✓ | ✓ |

The default deployment name normally matches the model ID. Custom deployment
names are supported, and applications must send the deployment name—not
necessarily the underlying model ID—in the `model` parameter.

The [Foundry model catalog](https://ai.azure.com/catalog/publishers/anthropic)
is the source of truth for current availability.

## Billing

Claude in Microsoft Foundry is billed through
[Azure Marketplace](https://azuremarketplace.microsoft.com/). Usage is measured
in Claude Consumption Units, metered hourly, and included on the monthly Azure
invoice. CCUs are usage units rather than prepaid credits, so there is no CCU
balance to maintain.

See
[Claude in Microsoft Foundry pricing](https://platform.claude.com/docs/en/about-claude/pricing#claude-in-microsoft-foundry-pricing)
for current conversion and model token rates.

## Troubleshooting

### `401 Unauthorized` or `Invalid API key`

- For Microsoft Entra ID, confirm the identity can obtain a token for
  `https://ai.azure.com/.default`.
- Confirm that the token has not expired. Entra access tokens commonly expire
  after about one hour; use a token provider for automatic refresh.
- For API key authentication, confirm the key belongs to the target Foundry
  resource and has not been rotated.

### `403 Forbidden`

- Confirm the calling identity has an appropriate Azure RBAC assignment, such
  as **Foundry User** or **Cognitive Services User**.
- Allow time for a new role assignment to propagate before retrying.

### `429 Too Many Requests`

- Apply exponential backoff with jitter and retry only retryable requests.
- Review usage and throttling in Azure Monitor.
- Request additional quota through Azure when sustained traffic exceeds the
  deployment limit.

Foundry responses do not include the standard Anthropic rate-limit headers.
Use Azure monitoring and quota tools to track capacity and throttling.

### `Model not found` or `Deployment not found`

- Pass the deployment name in the `model` parameter.
- Confirm the deployment exists in the target Foundry resource.
- Confirm the selected model and Azure-hosted version are available in the
  deployment region or scope.

### `Invalid model parameter`

- Verify that `model` contains the exact deployment name.
- If a custom deployment name was configured, do not substitute the underlying
  model ID.
