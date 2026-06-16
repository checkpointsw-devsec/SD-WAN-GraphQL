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
<div style="max-width: 900px; margin: 0 auto;">
<h1 style="font-size: 22px; margin-bottom: 4px;">SD-WAN Automation</h1>


<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 0 0 5px;">How to use the command:</div>
<div style="font-family: monospace; font-size: 13px; background: #f0f4f8; border: 1px solid #cdd5df; border-radius: 4px; padding: 8px 12px;">join-sdwan-profile.py --gateway &lt;gateway name&gt; --profile "SD-WAN Gateways" [--debug [request|response|both] [--env /path/to/.env]</div>
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 14px 0 5px;"></div>
<p style="color: #555; margin-bottom: 28px; font-size: 13px;"><strong>Note: </strong>to run the automation with SD-WAN you need to create an Account-API Key that has admin&nbsp;<br />permission on SD-WAN!</p>

<p style="color: #555; margin-bottom: 28px; font-size: 13px;">Assigns a gateway to an SD-WAN profile via the Infinity Portal. All GraphQL calls share the same endpoint; only the OAuth call differs.</p>
<!-- Flow -->
<h2 style="font-size: 16px; margin: 32px 0 12px; border-bottom: 2px solid #d0d5dd; padding-bottom: 6px;">Execution Flow</h2>
<div style="background: #fff; border: 1px solid #d0d5dd; border-radius: 6px; padding: 14px 18px; font-family: monospace; font-size: 13px; line-height: 1.8;">
<div style="display: flex; gap: 10px;"><span style="color: #888; min-width: 16px;">1.</span><span>OAuth token &nbsp;<span style="color: #0075be;">&rarr;</span>&nbsp; extract: <strong>token</strong>, <strong>tenant_id</strong></span></div>
<div style="display: flex; gap: 10px;"><span style="color: #888; min-width: 16px;">2.</span><span>getAssets &nbsp;<span style="color: #0075be;">&rarr;</span>&nbsp; extract: <strong>gateway asset id</strong></span></div>
<div style="display: flex; gap: 10px;"><span style="color: #888; min-width: 16px;">3.</span><span>getSdWanProfiles &nbsp;<span style="color: #0075be;">&rarr;</span>&nbsp; extract: <strong>profile id</strong></span></div>
<div style="display: flex; gap: 10px;"><span style="color: #888; min-width: 16px;">4.</span><span>updateSdWanProfile &nbsp;<span style="color: #0075be;">(uses ids from steps 2 &amp; 3)</span></span></div>
<div style="padding-left: 26px; color: #c04a00;">&#9492;&#9472; on lock error: discardChanges &rarr; retry updateSdWanProfile</div>
<div style="display: flex; gap: 10px;"><span style="color: #888; min-width: 16px;">5.</span><span>publishChanges</span></div>
<div style="display: flex; gap: 10px;"><span style="color: #888; min-width: 16px;">6.</span><span>enforcePolicy</span></div>
</div>
<hr style="border: none; border-top: 1px solid #d0d5dd; margin: 28px 0;" /><!-- Step 1 -->
<h2 style="font-size: 16px; margin: 32px 0 12px; border-bottom: 2px solid #d0d5dd; padding-bottom: 6px;">Step 1 &mdash; OAuth: Get Bearer Token</h2>
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 0 0 5px;">Endpoint</div>
<div style="font-family: monospace; font-size: 13px; background: #f0f4f8; border: 1px solid #cdd5df; border-radius: 4px; padding: 8px 12px;"><span style="display: inline-block; background: #0075be; color: #fff; font-size: 11px; font-weight: bold; border-radius: 3px; padding: 2px 7px; margin-right: 8px; vertical-align: middle;">POST</span>https://cloudinfra-gw.portal.checkpoint.com/auth/external</div>
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 14px 0 5px;">Headers</div>
<table style="width: 100%; border-collapse: collapse; font-size: 13px;">
<tbody>
<tr>
<th style="background: #f0f4f8; text-align: left; padding: 7px 10px; border: 1px solid #d0d5dd; font-weight: 600;">Header</th>
<th style="background: #f0f4f8; text-align: left; padding: 7px 10px; border: 1px solid #d0d5dd; font-weight: 600;">Value</th>
</tr>
<tr>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">Content-Type</code></td>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">application/json</code></td>
</tr>
</tbody>
</table>
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 14px 0 5px;">Request Body</div>
<pre style="margin: 0; font-family: monospace; font-size: 12.5px; background: #1e2530; color: #cdd9e5; border-radius: 5px; padding: 12px 14px; overflow-x: auto; line-height: 1.6;">{
  <span style="color: #ffa657;">"clientId"</span>:  <span style="color: #a5d6ff;">"&lt;SD_WAN_Client_ID&gt;"</span>,
  <span style="color: #ffa657;">"accessKey"</span>: <span style="color: #a5d6ff;">"&lt;SD_WAN_Client_SecretKey&gt;"</span>
}</pre>
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 14px 0 5px;">Response (HTTP 200)</div>
<pre style="margin: 0; font-family: monospace; font-size: 12.5px; background: #1e2530; color: #cdd9e5; border-radius: 5px; padding: 12px 14px; overflow-x: auto; line-height: 1.6;">{
  <span style="color: #ffa657;">"data"</span>: {
    <span style="color: #ffa657;">"token"</span>: <span style="color: #a5d6ff;">"&lt;jwt&gt;"</span>
  }
}</pre>
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 14px 0 5px;">Extracted for Next Step</div>
<table style="width: 100%; border-collapse: collapse; font-size: 13px;">
<tbody>
<tr>
<th style="background: #f0f4f8; text-align: left; padding: 7px 10px; border: 1px solid #d0d5dd; font-weight: 600;">Field</th>
<th style="background: #f0f4f8; text-align: left; padding: 7px 10px; border: 1px solid #d0d5dd; font-weight: 600;">Path</th>
<th style="background: #f0f4f8; text-align: left; padding: 7px 10px; border: 1px solid #d0d5dd; font-weight: 600;">Used as</th>
</tr>
<tr>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">token</code></td>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">data.token</code></td>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;">Bearer token for all subsequent requests</td>
</tr>
<tr>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">tenant_id</code></td>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;">JWT claim <code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">tid</code> or <code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">tenantId</code> (decoded from token)</td>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">x-tenant-id</code> header</td>
</tr>
</tbody>
</table>
<hr style="border: none; border-top: 1px solid #d0d5dd; margin: 28px 0;" /><!-- Shared headers -->
<h2 style="font-size: 16px; margin: 32px 0 12px; border-bottom: 2px solid #d0d5dd; padding-bottom: 6px;">Steps 2&ndash;6 &mdash; GraphQL Calls</h2>
<div style="background: #fff; border: 1px solid #d0d5dd; border-radius: 6px; padding: 18px 20px; margin-bottom: 20px;">
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 0 0 5px;">Shared Endpoint</div>
<div style="font-family: monospace; font-size: 13px; background: #f0f4f8; border: 1px solid #cdd5df; border-radius: 4px; padding: 8px 12px;"><span style="display: inline-block; background: #0075be; color: #fff; font-size: 11px; font-weight: bold; border-radius: 3px; padding: 2px 7px; margin-right: 8px; vertical-align: middle;">POST</span>https://cloudinfra-gw.portal.checkpoint.com/app/sd-wan/graphql/v1</div>
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 14px 0 5px;">Shared Headers</div>
<table style="width: 100%; border-collapse: collapse; font-size: 13px;">
<tbody>
<tr>
<th style="background: #f0f4f8; text-align: left; padding: 7px 10px; border: 1px solid #d0d5dd; font-weight: 600;">Header</th>
<th style="background: #f0f4f8; text-align: left; padding: 7px 10px; border: 1px solid #d0d5dd; font-weight: 600;">Value</th>
</tr>
<tr>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">Authorization</code></td>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">Bearer &lt;token&gt;</code></td>
</tr>
<tr>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">Content-Type</code></td>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">application/json</code></td>
</tr>
<tr>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">Accept</code></td>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">application/json</code></td>
</tr>
<tr>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">x-tenant-id</code> <span style="font-size: 11px; color: #555; background: #eef1f5; border-radius: 3px; padding: 1px 5px; margin-left: 6px;">optional</span></td>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">&lt;tenant_id&gt;</code> &mdash; included only when non-empty</td>
</tr>
</tbody>
</table>
</div>
<!-- Step 2 -->
<h3 style="font-size: 14px; margin: 24px 0 10px; color: #003087;">Step 2 &mdash; getAssets: Find Gateway Asset ID</h3>
<div style="background: #fff; border: 1px solid #d0d5dd; border-radius: 6px; padding: 18px 20px; margin-bottom: 20px;">
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 0 0 5px;">Request Body &mdash; Primary (filtered search)</div>
<pre style="margin: 0; font-family: monospace; font-size: 12.5px; background: #1e2530; color: #cdd9e5; border-radius: 5px; padding: 12px 14px; overflow-x: auto; line-height: 1.6;">{
  <span style="color: #ffa657;">"query"</span>: <span style="color: #a5d6ff;">"query GetAssets($matchSearch: String) { getAssets(matchSearch: $matchSearch) { assets { id name } } }"</span>,
  <span style="color: #ffa657;">"variables"</span>: { <span style="color: #ffa657;">"matchSearch"</span>: <span style="color: #a5d6ff;">"&lt;gateway-name&gt;"</span> }
}</pre>
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 14px 0 5px;">Request Body &mdash; Fallback (all assets, if no match found above)</div>
<pre style="margin: 0; font-family: monospace; font-size: 12.5px; background: #1e2530; color: #cdd9e5; border-radius: 5px; padding: 12px 14px; overflow-x: auto; line-height: 1.6;">{
  <span style="color: #ffa657;">"query"</span>: <span style="color: #a5d6ff;">"query { getAssets { assets { id name } } }"</span>,
  <span style="color: #ffa657;">"variables"</span>: {}
}</pre>
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 14px 0 5px;">Response example:</div>
<pre style="margin: 0; font-family: monospace; font-size: 12.5px; background: #1e2530; color: #cdd9e5; border-radius: 5px; padding: 12px 14px; overflow-x: auto; line-height: 1.6;">{
  <span style="color: #ffa657;">"data"</span>: {
    <span style="color: #ffa657;">"getAssets"</span>: {
      <span style="color: #ffa657;">"assets"</span>: [
        { <span style="color: #ffa657;">"id"</span>: <span style="color: #a5d6ff;">"abc-123"</span>, <span style="color: #ffa657;">"name"</span>: <span style="color: #a5d6ff;">"hq-exl"</span> },
        { <span style="color: #ffa657;">"id"</span>: <span style="color: #a5d6ff;">"def-456"</span>, <span style="color: #ffa657;">"name"</span>: <span style="color: #a5d6ff;">"branch-gw"</span> }
      ]
    }
  }
}</pre>
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 14px 0 5px;">Extracted for Next Step</div>
<table style="width: 100%; border-collapse: collapse; font-size: 13px;">
<tbody>
<tr>
<th style="background: #f0f4f8; text-align: left; padding: 7px 10px; border: 1px solid #d0d5dd; font-weight: 600;">Field</th>
<th style="background: #f0f4f8; text-align: left; padding: 7px 10px; border: 1px solid #d0d5dd; font-weight: 600;">Match Condition</th>
<th style="background: #f0f4f8; text-align: left; padding: 7px 10px; border: 1px solid #d0d5dd; font-weight: 600;">Used as</th>
</tr>
<tr>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">data.getAssets.assets[].id</code></td>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">name == gateway_name</code> or <code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">gateway_name in name</code></td>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">gateway_asset_id</code> &mdash; passed to updateSdWanProfile</td>
</tr>
</tbody>
</table>
</div>
<!-- Step 3 -->
<h3 style="font-size: 14px; margin: 24px 0 10px; color: #003087;">Step 3 &mdash; getSdWanProfiles: Find Profile ID</h3>
<div style="background: #fff; border: 1px solid #d0d5dd; border-radius: 6px; padding: 18px 20px; margin-bottom: 20px;">
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 0 0 5px;">Request Body</div>
<pre style="margin: 0; font-family: monospace; font-size: 12.5px; background: #1e2530; color: #cdd9e5; border-radius: 5px; padding: 12px 14px; overflow-x: auto; line-height: 1.6;">{
  <span style="color: #ffa657;">"query"</span>: <span style="color: #a5d6ff;">"query { getSdWanProfiles { id name } }"</span>,
  <span style="color: #ffa657;">"variables"</span>: {}
}</pre>
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 14px 0 5px;">Response example:</div>
<pre style="margin: 0; font-family: monospace; font-size: 12.5px; background: #1e2530; color: #cdd9e5; border-radius: 5px; padding: 12px 14px; overflow-x: auto; line-height: 1.6;">{
  <span style="color: #ffa657;">"data"</span>: {
    <span style="color: #ffa657;">"getSdWanProfiles"</span>: [
      { <span style="color: #ffa657;">"id"</span>: <span style="color: #a5d6ff;">"profile-111"</span>, <span style="color: #ffa657;">"name"</span>: <span style="color: #a5d6ff;">"SD-WAN Gateways"</span> },
      { <span style="color: #ffa657;">"id"</span>: <span style="color: #a5d6ff;">"profile-222"</span>, <span style="color: #ffa657;">"name"</span>: <span style="color: #a5d6ff;">"Branch Profiles"</span> }
    ]
  }
}</pre>
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 14px 0 5px;">Extracted for Next Step</div>
<table style="width: 100%; border-collapse: collapse; font-size: 13px;">
<tbody>
<tr>
<th style="background: #f0f4f8; text-align: left; padding: 7px 10px; border: 1px solid #d0d5dd; font-weight: 600;">Field</th>
<th style="background: #f0f4f8; text-align: left; padding: 7px 10px; border: 1px solid #d0d5dd; font-weight: 600;">Match Condition</th>
<th style="background: #f0f4f8; text-align: left; padding: 7px 10px; border: 1px solid #d0d5dd; font-weight: 600;">Used as</th>
</tr>
<tr>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">data.getSdWanProfiles[].id</code></td>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">name == profile_name</code> or <code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">profile_name in name</code></td>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">profile_id</code> &mdash; passed to updateSdWanProfile</td>
</tr>
</tbody>
</table>
</div>
<!-- Step 4 -->
<h3 style="font-size: 14px; margin: 24px 0 10px; color: #003087;">Step 4 &mdash; updateSdWanProfile: Assign Gateway to Profile</h3>
<div style="background: #fff; border: 1px solid #d0d5dd; border-radius: 6px; padding: 18px 20px; margin-bottom: 20px;">
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 0 0 5px;">Request Body</div>
<pre style="margin: 0; font-family: monospace; font-size: 12.5px; background: #1e2530; color: #cdd9e5; border-radius: 5px; padding: 12px 14px; overflow-x: auto; line-height: 1.6;">{
  <span style="color: #ffa657;">"query"</span>: <span style="color: #a5d6ff;">"mutation UpdateSdWanProfile($id: ID!, $input: SdWanProfileUpdateInput!) { updateSdWanProfile(id: $id input: $input) }"</span>,
  <span style="color: #ffa657;">"variables"</span>: {
    <span style="color: #ffa657;">"id"</span>: <span style="color: #a5d6ff;">"&lt;profile_id&gt;"</span>,
    <span style="color: #ffa657;">"input"</span>: {
      <span style="color: #ffa657;">"addSdWanGateways"</span>:    [<span style="color: #a5d6ff;">"&lt;gateway_asset_id&gt;"</span>],
      <span style="color: #ffa657;">"removeSdWanGateways"</span>: []
    }
  }
}</pre>
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 14px 0 5px;">Response &mdash; Success</div>
<pre style="margin: 0; font-family: monospace; font-size: 12.5px; background: #1e2530; color: #cdd9e5; border-radius: 5px; padding: 12px 14px; overflow-x: auto; line-height: 1.6;">{
  <span style="color: #ffa657;">"data"</span>: { <span style="color: #ffa657;">"updateSdWanProfile"</span>: <span style="color: #79c0ff;">true</span> }
}</pre>
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 14px 0 5px;">Response &mdash; Lock Error</div>
<pre style="margin: 0; font-family: monospace; font-size: 12.5px; background: #1e2530; color: #cdd9e5; border-radius: 5px; padding: 12px 14px; overflow-x: auto; line-height: 1.6;">{
  <span style="color: #ffa657;">"errors"</span>: [
    { <span style="color: #ffa657;">"message"</span>: <span style="color: #a5d6ff;">"forbidden-status: profile is locked"</span> }
  ]
}</pre>
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 14px 0 5px;">Extracted / Used</div>
<table style="width: 100%; border-collapse: collapse; font-size: 13px;">
<tbody>
<tr>
<th style="background: #f0f4f8; text-align: left; padding: 7px 10px; border: 1px solid #d0d5dd; font-weight: 600;">Field</th>
<th style="background: #f0f4f8; text-align: left; padding: 7px 10px; border: 1px solid #d0d5dd; font-weight: 600;">Condition</th>
<th style="background: #f0f4f8; text-align: left; padding: 7px 10px; border: 1px solid #d0d5dd; font-weight: 600;">Action</th>
</tr>
<tr>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">data.updateSdWanProfile</code></td>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">true</code></td>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;">Success &mdash; proceed to publishChanges</td>
</tr>
<tr>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">errors[0].message</code></td>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;">contains <code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">"lock"</code> or <code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">"forbidden-status"</code></td>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;">Run discardChanges (step 4a), then retry</td>
</tr>
<tr>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">errors[0].message</code></td>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;">any other error</td>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;">Exit with error</td>
</tr>
</tbody>
</table>
</div>
<!-- Step 4a -->
<h3 style="font-size: 14px; margin: 24px 0 10px; color: #003087;">Step 4a &mdash; discardChanges <span style="font-size: 11px; color: #555; background: #eef1f5; border-radius: 3px; padding: 1px 5px; margin-left: 6px;">only on lock error</span></h3>
<div style="background: #fff; border: 1px solid #d0d5dd; border-radius: 6px; padding: 18px 20px; margin-bottom: 20px;">
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 0 0 5px;">Request Body</div>
<pre style="margin: 0; font-family: monospace; font-size: 12.5px; background: #1e2530; color: #cdd9e5; border-radius: 5px; padding: 12px 14px; overflow-x: auto; line-height: 1.6;">{
  <span style="color: #ffa657;">"query"</span>: <span style="color: #a5d6ff;">"mutation { discardChanges }"</span>,
  <span style="color: #ffa657;">"variables"</span>: {}
}</pre>
<p style="margin: 10px 0 0; font-size: 13px; color: #444;">Response is ignored. After this call, <code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">updateSdWanProfile</code> is retried once with the same request as step 4.</p>
</div>
<!-- Step 5 -->
<h3 style="font-size: 14px; margin: 24px 0 10px; color: #003087;">Step 5 &mdash; publishChanges</h3>
<div style="background: #fff; border: 1px solid #d0d5dd; border-radius: 6px; padding: 18px 20px; margin-bottom: 20px;">
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 0 0 5px;">Request Body</div>
<pre style="margin: 0; font-family: monospace; font-size: 12.5px; background: #1e2530; color: #cdd9e5; border-radius: 5px; padding: 12px 14px; overflow-x: auto; line-height: 1.6;">{
  <span style="color: #ffa657;">"query"</span>: <span style="color: #a5d6ff;">"mutation { publishChanges { isValid } }"</span>,
  <span style="color: #ffa657;">"variables"</span>: {}
}</pre>
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 14px 0 5px;">Response</div>
<pre style="margin: 0; font-family: monospace; font-size: 12.5px; background: #1e2530; color: #cdd9e5; border-radius: 5px; padding: 12px 14px; overflow-x: auto; line-height: 1.6;">{
  <span style="color: #ffa657;">"data"</span>: {
    <span style="color: #ffa657;">"publishChanges"</span>: { <span style="color: #ffa657;">"isValid"</span>: <span style="color: #79c0ff;">true</span> }
  }
}</pre>
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 14px 0 5px;">Extracted</div>
<table style="width: 100%; border-collapse: collapse; font-size: 13px;">
<tbody>
<tr>
<th style="background: #f0f4f8; text-align: left; padding: 7px 10px; border: 1px solid #d0d5dd; font-weight: 600;">Field</th>
<th style="background: #f0f4f8; text-align: left; padding: 7px 10px; border: 1px solid #d0d5dd; font-weight: 600;">Condition</th>
<th style="background: #f0f4f8; text-align: left; padding: 7px 10px; border: 1px solid #d0d5dd; font-weight: 600;">Action</th>
</tr>
<tr>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">data.publishChanges.isValid</code></td>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">false</code></td>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;">Print warning (non-fatal, execution continues)</td>
</tr>
</tbody>
</table>
</div>
<!-- Step 6 -->
<h3 style="font-size: 14px; margin: 24px 0 10px; color: #003087;">Step 6 &mdash; enforcePolicy</h3>
<div style="background: #fff; border: 1px solid #d0d5dd; border-radius: 6px; padding: 18px 20px; margin-bottom: 20px;">
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 0 0 5px;">Request Body</div>
<pre style="margin: 0; font-family: monospace; font-size: 12.5px; background: #1e2530; color: #cdd9e5; border-radius: 5px; padding: 12px 14px; overflow-x: auto; line-height: 1.6;">{
  <span style="color: #ffa657;">"query"</span>: <span style="color: #a5d6ff;">"mutation { enforcePolicy { id status } }"</span>,
  <span style="color: #ffa657;">"variables"</span>: {}
}</pre>
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 14px 0 5px;">Response example:</div>
<pre style="margin: 0; font-family: monospace; font-size: 12.5px; background: #1e2530; color: #cdd9e5; border-radius: 5px; padding: 12px 14px; overflow-x: auto; line-height: 1.6;">{
  <span style="color: #ffa657;">"data"</span>: {
    <span style="color: #ffa657;">"enforcePolicy"</span>: {
      <span style="color: #ffa657;">"id"</span>:     <span style="color: #a5d6ff;">"task-789"</span>,
      <span style="color: #ffa657;">"status"</span>: <span style="color: #a5d6ff;">"pending"</span>
    }
  }
}</pre>
<div style="font-size: 11px; font-weight: bold; text-transform: uppercase; letter-spacing: .06em; color: #555; margin: 14px 0 5px;">Extracted</div>
<table style="width: 100%; border-collapse: collapse; font-size: 13px;">
<tbody>
<tr>
<th style="background: #f0f4f8; text-align: left; padding: 7px 10px; border: 1px solid #d0d5dd; font-weight: 600;">Field</th>
<th style="background: #f0f4f8; text-align: left; padding: 7px 10px; border: 1px solid #d0d5dd; font-weight: 600;">Used as</th>
</tr>
<tr>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">data.enforcePolicy.id</code></td>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;">Printed as task tracking reference</td>
</tr>
<tr>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;"><code style="font-family: monospace; font-size: 12px; background: #eef1f5; padding: 1px 4px; border-radius: 3px;">data.enforcePolicy.status</code></td>
<td style="padding: 7px 10px; border: 1px solid #d0d5dd; vertical-align: top;">Printed (informational)</td>
</tr>
</tbody>
</table>
<div style="font-size: 12.5px; color: #555; background: #fff8e6; border-left: 3px solid #f0a500; padding: 8px 12px; border-radius: 0 4px 4px 0; margin-top: 12px;">Please Note: the flow described here does not poll the enforce task &mdash; it fires and continues. it might be in some cases useful to enforce and check the result.</div>
</div>
</div>
					</div>
					
