# Identity Server (IDS)

The Identity Server is a "special microservice" that provides identity management for users (authentication) and for the microservices' client applications (authorization).

It also includes a Web API for managing its resources.

## Specification

| Metadata | Value |
| - | - |
| Namespace | Primavera.Lithium.IdentityServer |
| Version | 4.0 |
| API Version | 4.0 |
| Client Library | Primavera.IdentityServer.dll |
| Web UI | Yes |
| Depends on | SS, NS |

## Scopes

| Scope | Description |
| - | - |
| `identityserver4` | Access to the endpoints in the API that allow reading resources |
| `identityserver4-write` | Access to the endpoints in the API that allows writing resources (not privileged) |
| `identityserver4-privileged` | Access to the endpoints in the API that allows writing resources (privileged) |

## Environments

| Environment | Available |
| - | - |
| Production | Yes |
| Preview (WE) | Yes |
| Preview (NE) | Yes |
| Staging (WE) | Yes |
| Staging (NE) | --- |
| Development | --- |

## Reference Documentation

| Documentation | Link |
| - | - |
| Client Library | Not available yet |
| Web API | Not available yet |
