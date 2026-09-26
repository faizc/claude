# Migrating between hosting options

To move an existing deployment from one hosting option to the other:

1. Create a new deployment of the model's other hosting version: **Hosted on
   Azure** or **Hosted on Anthropic**. The deployment can use the same Foundry
   resource or a new resource.
2. Update the application to pass the new deployment name in the `model`
   parameter.
3. Delete the old deployment after application traffic has moved successfully.

When the new deployment is in the same Foundry resource, the endpoint URL and
authentication configuration remain unchanged. When using a new resource,
update the application's endpoint and credentials to target that resource.
