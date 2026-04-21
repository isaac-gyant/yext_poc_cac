# Yext POC — Configuration as Code

Yext CaC configuration for syncing healthcare providers and locations from the Fabric Health NestJS API.

## Structure

```
yext_cac/
├── km/
│   ├── entity-type/
│   │   ├── healthcareProvider.json          # Provider entity type
│   │   └── healthcareLocation.json          # Location entity type
│   ├── field/
│   │   ├── c_acceptingNewPatients.json      # Bool: accepting new patients
│   │   ├── c_accessibilityFeatures.json     # List: ADA features
│   │   ├── c_credentials.json               # String: MD, FACC, etc.
│   │   ├── c_displayName.json               # String: Dr. Jane Smith
│   │   ├── c_insurancesAccepted.json        # List: insurance plans
│   │   ├── c_languages.json                 # List: spoken languages
│   │   ├── c_locationIds.json               # List: linked location IDs
│   │   ├── c_parkingType.json               # String: parking type
│   │   ├── c_ratingsAverage.json            # Decimal: avg rating
│   │   ├── c_ratingsCount.json              # Integer: total ratings
│   │   ├── c_services.json                  # List: medical services
│   │   ├── c_specialty.json                 # String: specialty
│   │   ├── c_subspecialty.json              # String: subspecialty
│   │   ├── c_telehealth.json                # Bool: telehealth available
│   │   └── c_urgentCare.json                # Bool: urgent care available
│   └── connector/
│       ├── fabricHealthProviders.json        # HTTP connector: providers
│       └── fabricHealthLocations.json        # HTTP connector: locations
└── README.md
```

## How It Works

### Authentication

Both connectors reference `${{fabricHealthApiKey}}` in the `x-api-key` header. When the secret doesn't exist yet, Yext prompts for its value during `yext resources apply`. Each client (Yext app installation) enters the unique API key we provide them, and Yext stores it as the `fabricHealthApiKey` secret for that account.

### Pagination

The connectors use **token-based pagination**:
- `pageToken` query parameter carries the cursor
- Response `$.pagination.pageToken` provides the next cursor
- Response `$.pagination.hasMore` signals whether more pages exist
- Default page size: 20

### Sync Schedule

| Connector | Cron | Description |
|-----------|------|-------------|
| Providers | `0 2 * * *` | Daily at 2:00 AM |
| Locations | `0 3 * * *` | Daily at 3:00 AM |

## Prerequisites

1. **Yext CLI** installed: `brew install yext/tap/yext`
2. Authenticate: `yext init`
3. Update `ci.json` with your actual `businessId`

## Deployment

```bash
# Preview changes
yext resources apply --dry-run

# Apply to Yext
yext resources apply
```

## Secrets

During app installation, the client creates the `fabricHealthApiKey` secret with the API key provided to them:

```bash
yext secrets create fabricHealthApiKey --value "<API_KEY_PROVIDED_TO_CLIENT>"
```

Or via the Yext Platform UI under **Account Settings > Secrets**.

Each client gets a unique key (e.g. `yext-poc-secret-key-2024` for the Isaac App, `demo-api-key-2024` for the Demo App). The backend identifies which client is calling based on the key.

## Connector Base URL

Connectors currently point to `https://fabric.isaacnewton.dev`. For local development, update `baseUrl` in both connector JSON files to `http://localhost:30000`.
