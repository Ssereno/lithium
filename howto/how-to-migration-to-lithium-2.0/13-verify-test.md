# How to Upgrade a Microservice from Lithium v1.0 to Lithium v2.0

## 13. Verify and Test the Service

### 13.1 - Verify the Service Routes

If the service generates a Web API it is very important that the API routes remain unchanged after the upgrade.

Compare the `Models\GeneratedCode\Routes.gen.cs` with the previous version (before the upgrade) and make sure that none of the routes have changed.

> NOTE: The `v{apiVersion:apiVersion}` part in the routes is expected to change to `v{version:apiVersion}`.

### 13.2 - Test the Service using Postman

You can use Postman to test the service Web API. As part of the upgrade a Postman collection is generated - see `Design\GeneratedCode\Postman.gen.postman_collection.json` - that can be used for that purpose.

### 13.3 - Test the Service using the Console Client

The console client is another way to test the behavior of the service.

### 13.4 - The following aspects of service behavior should be tested with extra attention:

- Authorization scopes and authorization policies.
- Background services and background workers.
- OIDC authentication.
- Custom user interface.
- REDIS cache.
- Table storage and blob storage.

### 13.5 - The Upgrade is DONE!