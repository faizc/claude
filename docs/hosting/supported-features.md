# Feature support

Claude in Microsoft Foundry supports most Claude features. See the
[Claude features overview](https://platform.claude.com/docs/en/build-with-claude/overview)
for the complete feature catalog.

## Context window

The following models support a
[1M-token context window](https://platform.claude.com/docs/en/build-with-claude/context-windows)
in Microsoft Foundry:

- Claude Fable 5.1
- Claude Fable 5
- Claude Opus 5.5
- Claude Opus 5
- Claude Opus 4.8
- Claude Opus 4.7
- Claude Opus 4.6
- Claude Sonnet 5
- Claude Sonnet 4.6

Other Claude models, including Claude Sonnet 4.5, support a 200k-token context
window.

## Claude features not supported in Microsoft Foundry

The following features are not currently supported for Claude deployments in
Microsoft Foundry:

- Admin API
- Advisor tool
- Claude Managed Agents
- Compliance API
- Models API
- Message Batches API
- Server-side fallback through the
  [`fallbacks` parameter](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback#server-side-fallback);
  use the
  [client-side fallback pattern](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback#client-side-fallback)
  instead
- [Computer use](https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool)
  and
  [browser use](https://platform.claude.com/docs/en/agents-and-tools/tool-use/browser-use-tool)
  toolsets `computer_toolset_20260801` and `browser_toolset_20260801`; beta
  computer-use tool versions remain available

!!! important

    **Managed Agents is first-party only.** It is not available on Amazon
    Bedrock, Google Vertex AI, or Microsoft Foundry. For agents on third-party
    providers, use
    [Claude API and tool use](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview).

## Additional features not supported when hosted on Azure

The following features are available for deployments hosted on Anthropic
infrastructure but are not currently supported for deployments hosted on
Azure:

- [Code execution](https://platform.claude.com/docs/en/agents-and-tools/tool-use/code-execution-tool)
- [Web search](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool)
  and
  [web fetch](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-fetch-tool)
  versions later than `web_search_20250305` and `web_fetch_20250910`.
  Azure-hosted deployments support these basic versions, but dynamic
  filtering, response inclusion, and cache bypass are unavailable.
- [Agent Skills](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [Programmatic tool calling](https://platform.claude.com/docs/en/agents-and-tools/tool-use/programmatic-tool-calling)
- [Files API](https://platform.claude.com/docs/en/build-with-claude/files)

Requests that use these features with an Azure-hosted deployment return
`400 Bad Request` by design. Claude Code detects Azure-hosted deployments and
automatically adjusts its feature set.
