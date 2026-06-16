# SD-WAN-GraphQL
make sure you have requirements installed on your host
```BASH
pip install -r requirements.txt
```

usage:
```BASH
join-sdwan-profile.py --gateway <gateway name> --profile "<profile name>"
```

example: 
```BASH
join-sdwan-profile.py --gateway hq-exl-1 --profile "SD-WAN Gateways"
```

options:
```BASH
join-sdwan-profile.py --gateway hq-exl --profile "SD-WAN Gateways" [--debug <reuest | response | both> [--env /path/to/.env]
```
# SD-WAN Automation

Assigns a gateway to an SD-WAN profile via the Check Point Infinity Portal.

All SD-WAN GraphQL calls use the same endpoint. Only the OAuth authentication call uses a different endpoint.

## Usage

```bash
./join-sdwan-profile.py --gateway "<gateway name>" --profile "SD-WAN Gateways" [--debug request|response|both] [--env /path/to/.env]
```

Example:

```bash
./join-sdwan-profile.py --gateway "branch-gw" --profile "SD-WAN Gateways" --debug both --env .env
```

> **Note**
>
> To run this automation with SD-WAN, you need to create an **Account API Key** with **admin permission** for SD-WAN.

## Execution Flow

1. Get OAuth token

   * Extract `token`
   * Extract `tenant_id`
2. Run `getAssets`

   * Find and extract the gateway asset ID
3. Run `getSdWanProfiles`

   * Find and extract the SD-WAN profile ID
4. Run `updateSdWanProfile`

   * Assign the gateway to the profile
   * If the profile is locked:

     * Run `discardChanges`
     * Retry `updateSdWanProfile`
5. Run `publishChanges`
6. Run `enforcePolicy`

---

# API Flow Details

## Step 1 — OAuth: Get Bearer Token

### Endpoint

```http
POST https://cloudinfra-gw.portal.checkpoint.com/auth/external
```

### Headers

| Header         | Value              |
| -------------- | ------------------ |
| `Content-Type` | `application/json` |

### Request Body

```json
{
  "clientId": "<SD_WAN_Client_ID>",
  "accessKey": "<SD_WAN_Client_SecretKey>"
}
```

### Response Example

```json
{
  "data": {
    "token": "<jwt>"
  }
}
```

### Extracted Values

| Field       | Path                                             | Used As                                  |
| ----------- | ------------------------------------------------ | ---------------------------------------- |
| `token`     | `data.token`                                     | Bearer token for all subsequent requests |
| `tenant_id` | JWT claim `tid` or `tenantId` decoded from token | `x-tenant-id` header                     |

---

## Steps 2–6 — Shared GraphQL Configuration

### Shared Endpoint

```http
POST https://cloudinfra-gw.portal.checkpoint.com/app/sd-wan/graphql/v1
```

### Shared Headers

| Header          | Value                                                  |
| --------------- | ------------------------------------------------------ |
| `Authorization` | `Bearer <token>`                                       |
| `Content-Type`  | `application/json`                                     |
| `Accept`        | `application/json`                                     |
| `x-tenant-id`   | `<tenant_id>` — optional, included only when non-empty |

---

## Step 2 — `getAssets`: Find Gateway Asset ID

The script first tries to search for the gateway by name. If no match is found, it can fall back to retrieving all assets.

### Request Body — Primary Filtered Search

```json
{
  "query": "query GetAssets($matchSearch: String) { getAssets(matchSearch: $matchSearch) { assets { id name } } }",
  "variables": {
    "matchSearch": "<gateway-name>"
  }
}
```

### Request Body — Fallback Search

```json
{
  "query": "query { getAssets { assets { id name } } }",
  "variables": {}
}
```

### Response Example

```json
{
  "data": {
    "getAssets": {
      "assets": [
        {
          "id": "abc-123",
          "name": "hq-exl"
        },
        {
          "id": "def-456",
          "name": "branch-gw"
        }
      ]
    }
  }
}
```

### Extracted Values

| Field                        | Match Condition                                  | Used As                                           |
| ---------------------------- | ------------------------------------------------ | ------------------------------------------------- |
| `data.getAssets.assets[].id` | `name == gateway_name` or `gateway_name in name` | `gateway_asset_id` passed to `updateSdWanProfile` |

---

## Step 3 — `getSdWanProfiles`: Find Profile ID

### Request Body

```json
{
  "query": "query { getSdWanProfiles { id name } }",
  "variables": {}
}
```

### Response Example

```json
{
  "data": {
    "getSdWanProfiles": [
      {
        "id": "profile-111",
        "name": "SD-WAN Gateways"
      },
      {
        "id": "profile-222",
        "name": "Branch Profiles"
      }
    ]
  }
}
```

### Extracted Values

| Field                        | Match Condition                                  | Used As                                     |
| ---------------------------- | ------------------------------------------------ | ------------------------------------------- |
| `data.getSdWanProfiles[].id` | `name == profile_name` or `profile_name in name` | `profile_id` passed to `updateSdWanProfile` |

---

## Step 4 — `updateSdWanProfile`: Assign Gateway to Profile

### Request Body

```json
{
  "query": "mutation UpdateSdWanProfile($id: ID!, $input: SdWanProfileUpdateInput!) { updateSdWanProfile(id: $id input: $input) }",
  "variables": {
    "id": "<profile_id>",
    "input": {
      "addSdWanGateways": [
        "<gateway_asset_id>"
      ],
      "removeSdWanGateways": []
    }
  }
}
```

### Success Response

```json
{
  "data": {
    "updateSdWanProfile": true
  }
}
```

### Lock Error Response

```json
{
  "errors": [
    {
      "message": "forbidden-status: profile is locked"
    }
  ]
}
```

### Result Handling

| Field                     | Condition                             | Action                                                      |
| ------------------------- | ------------------------------------- | ----------------------------------------------------------- |
| `data.updateSdWanProfile` | `true`                                | Success. Continue to `publishChanges`.                      |
| `errors[0].message`       | Contains `lock` or `forbidden-status` | Run `discardChanges`, then retry `updateSdWanProfile` once. |
| `errors[0].message`       | Any other error                       | Exit with error.                                            |

---

## Step 4a — `discardChanges`

This step is only used when `updateSdWanProfile` fails because the profile is locked.

### Request Body

```json
{
  "query": "mutation { discardChanges }",
  "variables": {}
}
```

The response is ignored.

After this call, `updateSdWanProfile` is retried once using the same request body from Step 4.

---

## Step 5 — `publishChanges`

### Request Body

```json
{
  "query": "mutation { publishChanges { isValid } }",
  "variables": {}
}
```

### Response Example

```json
{
  "data": {
    "publishChanges": {
      "isValid": true
    }
  }
}
```

### Result Handling

| Field                         | Condition | Action                                                                 |
| ----------------------------- | --------- | ---------------------------------------------------------------------- |
| `data.publishChanges.isValid` | `true`    | Changes are valid. Continue to `enforcePolicy`.                        |
| `data.publishChanges.isValid` | `false`   | Print a warning. This is treated as non-fatal and execution continues. |

---

## Step 6 — `enforcePolicy`

### Request Body

```json
{
  "query": "mutation { enforcePolicy { id status } }",
  "variables": {}
}
```

### Response Example

```json
{
  "data": {
    "enforcePolicy": {
      "id": "task-789",
      "status": "pending"
    }
  }
}
```

### Extracted Values

| Field                       | Used As                            |
| --------------------------- | ---------------------------------- |
| `data.enforcePolicy.id`     | Printed as task tracking reference |
| `data.enforcePolicy.status` | Printed as informational status    |

> **Important**
>
> This flow does not poll the enforce task after it has been triggered.
>
> In some environments, it may be useful to add task polling after `enforcePolicy` to verify the final result.

---

# Environment File Example

Create a `.env` file and store the SD-WAN API credentials there.

```env
SD_WAN_CLIENT_ID="<your-client-id>"
SD_WAN_CLIENT_SECRET_KEY="<your-secret-key>"
```

Make sure the `.env` file is not committed to Git.

Add it to `.gitignore`:

```gitignore
.env
```

---

# Debug Options

The script supports optional debug output.

```bash
--debug request
--debug response
--debug both
```

| Debug Option | Description                             |
| ------------ | --------------------------------------- |
| `request`    | Print outgoing request details          |
| `response`   | Print API response details              |
| `both`       | Print both request and response details |

---

# Summary

This automation performs the following actions:

1. Authenticates against the Infinity Portal.
2. Finds the target gateway asset.
3. Finds the target SD-WAN profile.
4. Adds the gateway to the SD-WAN profile.
5. Handles profile lock errors by discarding pending changes and retrying.
6. Publishes the changes.
7. Enforces the SD-WAN policy.

