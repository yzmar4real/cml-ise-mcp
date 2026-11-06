# Cisco ISE MCP — Safe Agent Workflow

## Purpose

This runbook describes a bounded, safe, read-only workflow for validating the Cisco ISE MCP lab. It is not authorization to change Cisco ISE, the CML lab, network devices, credentials, firewall rules, DNS, routing, TLS, or persistent services.

Use it only for an explicitly approved target and scope. Perform read-only discovery first. Stop and request approval before any destructive, service-impacting, externally visible, credential-changing, or power-related action.

## Local credentials

The only local credential source is the operator-controlled, untracked file:

```text
~/cml-ise-mcp/mcp.log
```

Use `mcp.log.example` as the template. Keep `mcp.log` local and mode `600`; never commit, upload, paste, print, echo, or report its contents. Read only the named values required for the approved check. Do not use `source`, `.`, or `eval` on the file, and do not put credentials in command arguments, URLs, service definitions, logs, or MCP messages.

Expected keys are:

```text
SSH_HOST
SSH_PORT
SSH_USERNAME
SSH_KEY_PATH or SSH_PASSWORD
MCP_REMOTE_WORKDIR
ISE_IP
API_PORT
API_USERNAME
API_PWD
ISE_VERIFY_SERVER_CERT
ISE_VERIFY_HOSTNAME
HOST
PORT
```

Treat all values as sensitive except where a value is explicitly a non-secret operational setting. Validate them only as pass/fail. The ISE API port must be exactly `443`; the MCP listener port must be exactly `5000` for this lab convention. Do not report the actual host, address, username, password, key path, token, or environment-file contents.

## Discovery before validation

Record only sanitized facts:

- target identity and approved scope;
- repository and checkout state;
- service/process state;
- listener and endpoint availability;
- whether credentials are present, without displaying values.

Keep SSH host-key verification enabled. Never disable host-key checks or use `StrictHostKeyChecking=no`. Do not reuse an existing remote checkout if the approved procedure requires a new one. Do not delete or overwrite data as part of discovery.

## Read-only MCP validation

Use only the approved MCP endpoint. A safe validation sequence is:

1. MCP `initialize`.
2. `notifications/initialized`.
3. `tools/list`.
4. One specifically approved, read-only Cisco ISE query, if the target and query are in scope.

Never invoke tools that create, update, delete, restart, disable, or otherwise mutate Cisco ISE or the MCP service. Do not treat successful MCP initialization or tool discovery as proof that Cisco ISE-backed retrieval succeeded. Report transport success and ISE data retrieval separately.

Do not change source code or Cisco ISE configuration. Do not install packages, create services, change firewall/DNS/routing/TLS settings, or modify CML lifecycle state during validation without separate explicit approval.

## Safe handling of remote configuration

If a separately approved host-side setup requires a remote `.env`, transfer it through a protected stdin/channel, set mode `600`, verify that it is untracked, and validate only presence/format checks. Never display the file. Never place credentials in a systemd unit, shell history, process arguments, or diagnostic output.

## Final report

Return a concise sanitized report containing:

- status: `PASS`, `PARTIAL`, `BLOCKED`, or `FAILED`;
- observed timestamp;
- non-secret target identifier;
- repository/check-out result;
- API port `443` and MCP port `5000` pass/fail results;
- MCP initialize and tools/list pass/fail results;
- read-only ISE query pass/fail/not-run result;
- credential exposure: `NO`;
- changes made: `NONE`, unless separately approved and explicitly verified;
- observed issues only.

Never include passwords, private keys, tokens, authorization headers, private addresses, raw `mcp.log`, complete `.env` contents, or unredacted command output.
