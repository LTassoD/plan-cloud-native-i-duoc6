# Configuración Azure CLI

## Estado actual

| Campo | Valor |
|-------|-------|
| Azure CLI | v2.89.1 |
| Cuenta | Azure for Students |
| Tenant | Fundación Instituto Profesional Duoc UC (`duoc.cl`) |
| Usuario | `lu.tasso@duocuc.cl` |
| Estado | Active |

## Comandos útiles

```bash
# Verificar sesión activa
az account show

# Ver suscripción actual
az account show --query "subscriptionId" -o tsv

# Listar todos los recursos creados
az resource list --output table

# Verificar saldo de crédito (solo portal)
# → portal.azure.com → Subscriptions → tu suscripción → Charges
```

## Recursos clave en Azure

| Recurso | CLI para verificar |
|---------|-------------------|
| API Management | `az apim list --output table` |
| App Service | `az webapp list --output table` |
| Cosmos DB | `az cosmosdb list --output table` |
| Service Bus | `az servicebus namespace list --output table` |
| Event Hubs | `az eventhubs namespace list --output table` |
| Azure AD | `az ad app list --output table` |
