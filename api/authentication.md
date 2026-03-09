# Authentication

All V1 API endpoints require authentication. The API supports two methods:

## API Key Authentication (Recommended for Integrations)

Pass your API key in the `X-Public-API-Key` header:

```bash
curl -H "X-Public-API-Key: gm_live_abc123..." \
     https://agent.personize.ai/api/v1/me
```

### Verifying Your Key

```bash
curl -H "X-Public-API-Key: gm_live_abc123..." \
     https://agent.personize.ai/api/v1/me
```

Response:

```json
{
  "userId": "user_abc123",
  "organizationId": "org_xyz789",
  "organizationName": "Acme Corp",
  "plan": {
    "limits": {
      "maxApiCallsPerMinute": 60,
      "maxApiCallsPerMonth": 50000
    }
  }
}
```

The `organizationId` is automatically injected into all subsequent requests. You do not need to pass it explicitly. Check `plan.limits.maxApiCallsPerMinute` before running batch experiments — each record uses approximately 4–6 API calls, so divide by 6 for estimated records/min throughput.

## Bearer Token Authentication (For UI/Frontend)

Pass a JWT token in the Authorization header:

```bash
curl -H "Authorization: Bearer eyJhbGciOi..." \
     https://agent.personize.ai/api/v1/me
```

## API Key Scopes

API keys are scoped to a single organization. All operations performed with a key are isolated to that organization's data partition. A key cannot access memories, variables, schemas, or evaluation logs belonging to other organizations.
