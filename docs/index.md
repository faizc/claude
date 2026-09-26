# Anthropic on Azure onboarding guidance

Welcome to the documentation hub for customers planning to adopt and operate
Anthropic Claude on Microsoft Azure.

This site brings together the technical guidance, architecture decisions, and
deployment resources needed to move from initial evaluation to a secure,
production-ready Claude implementation in Microsoft Foundry.

## Available guidance

### Hosting Claude in Microsoft Foundry

Learn how to provision and call Claude models through Microsoft Foundry:

- [Hosting options](hosting/options.md) covers prerequisites, SDK installation,
  deployment, Microsoft Entra ID and API key authentication, model IDs,
  billing, and troubleshooting.
- [Supported features](hosting/supported-features.md) documents context-window
  support and capabilities that differ across Microsoft Foundry hosting
  options.
- [Migration](hosting/migration.md) explains how to move an application between
  Azure-hosted and Anthropic-hosted deployments.

### One-click infrastructure deployment

Deploy the required Foundry resources and Claude models with Azure Developer
CLI using either infrastructure-as-code implementation:

- [Bicep implementation](https://github.com/Azure-Samples/claude/tree/main/infra-bicep)
- [Terraform implementation](https://github.com/Azure-Samples/claude/tree/main/infra-terraform)

Both implementations use `azd up` as the deployment entry point and configure
Claude access through Microsoft Entra ID.

## Guidance in progress

The following documentation areas are planned and currently display a
work-in-progress page.

### Azure landing zone architecture

Plan the Azure foundation for Claude workloads, including subscription design,
resource organization, networking, security controls, governance, and
enterprise landing-zone alignment.

### AI Gateway

Design an Azure API Management-based AI gateway for centralized model access,
policy enforcement, token governance, observability, and controlled consumption
across applications and teams.

### Microsoft Foundry

Learn how Claude fits into Microsoft Foundry application and agent
architectures, including retrieval-augmented generation and agent design
patterns.

### Microsoft Entra ID

Use passwordless authentication, Azure RBAC, managed identities, and
least-privilege access to secure model inference without embedding API keys in
applications.

### Operations

Prepare monitoring, diagnostics, cost management, quota planning, deployment
verification, and operational runbooks for production workloads.

## Intended audience

This guidance is for:

- Cloud and solution architects designing Claude workloads on Azure
- Platform teams establishing secure, reusable AI foundations
- Developers integrating Claude through Microsoft Foundry endpoints
- Security and identity teams defining network and access controls
- Operations teams responsible for reliability, monitoring, quota, and cost

## Start here

Begin with [Hosting Claude in Microsoft Foundry](hosting/options.md) to
understand deployment choices, prerequisites, authentication, and available
models. Continue to [Supported features](hosting/supported-features.md) before
selecting a hosting option, and use the [Migration guide](hosting/migration.md)
when moving an existing deployment.
