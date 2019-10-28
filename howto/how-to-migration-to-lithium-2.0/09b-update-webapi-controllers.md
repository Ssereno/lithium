# How to Upgrade a Microservice from Lithium v1.0 to Lithium v2.0

## 9b - Update Controllers

### 9b.1 - `ServiceProvider`

The generated API controllers no longer have a ServiceProvider property.

Code like this:

```csharp
private ISubscriptionsManager Manager
{
    get
    {
        return this.ServiceProvider.ResolveService<ISubscriptionsManager>(true);
    }
}
```

should be converted to:

```csharp
private ISubscriptionsManager Manager
{
    get
    {
        return this.HttpContext.RequestServices.GetRequiredService<ISubscriptionsManager>();
    }
}
```

> NOTE: This dependency in particular could also be injected in the controller constructor.

## Next

> [9c - Update Web UI Custom Code](./09c-update-webapi-webui.md)