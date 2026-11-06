# Cisco ISE MCP Server — Agent Instructions

## Goal

SSH to the Ubuntu MCP server, clone and configure Cisco's official ISE MCP server, install its required dependencies, force the ISE API connection to port `443`, test the MCP service, and return a concise report.

Change nothing else.

## Credential source

The only credential source is the AI agent's local file:

```text
~/ise-mcp.log
```

The file uses dotenv-style `KEY=VALUE` entries and contains both SSH and ISE values.

Example:

```dotenv
# Ubuntu MCP server SSH access
SSH_HOST=ubuntu-mcp.example
SSH_PORT=22
SSH_USERNAME=operator
SSH_KEY_PATH=/home/agent/.ssh/ubuntu_mcp
# SSH_PASSWORD=REPLACE_WITH_LOCAL_VALUE

# Remote setup directory
MCP_REMOTE_WORKDIR=/home/operator/cisco-ise-mcp-official

# Cisco ISE API access
ISE_IP=ise-pan.example
API_PORT=443
API_USERNAME=ise_mcp_readonly
API_PWD=REPLACE_WITH_ISE_API_PASSWORD

# TLS verification
ISE_VERIFY_SERVER_CERT=true
ISE_VERIFY_HOSTNAME=true

# MCP listener
HOST=0.0.0.0
PORT=5000
```

Protect the file:

```bash
chmod 600 ~/ise-mcp.log
```

Read only the required named values. Do not use `source`, `.`, or `eval` on the file.

Never print, echo, log, upload, commit, or report:

- passwords
- private keys
- tokens
- authorization headers
- complete `.env` contents
- contents of `~/ise-mcp.log`

SSH credentials are used only for SSH. ISE credentials are written only to the remote MCP server `.env`.

## Hard requirements

- Clone only:
  `https://github.com/CiscoDevNet/cisco-ise-mcp-official.git`
- ISE API port: **443**
- Never use ISE port `9060`.
- MCP listener port: **5000**
- Do not modify source code.
- Do not modify Cisco ISE configuration.
- Do not modify firewall, DNS, routing, or TLS settings.
- Do not install a systemd service, cron job, Docker service, or supervisor.
- Do not overwrite an existing checkout.
- Do not invoke mutating ISE or MCP tools.
- Do not expose credentials.
- Do not make unrelated changes.

## 1. Read and validate local configuration

Confirm that this file exists and is readable:

```text
~/ise-mcp.log
```

Required values:

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

Stop if required values are missing or if:

```text
API_PORT is not exactly 443
PORT is not exactly 5000
TLS verification is disabled
```

Do not silently correct invalid values.

## 2. SSH to the Ubuntu MCP server

Use the SSH values from `~/ise-mcp.log`.

Keep SSH host-key verification enabled. Never use:

```text
StrictHostKeyChecking=no
```

If password authentication is required:

- use `sshpass -e` with `SSHPASS` supplied through the environment,
- never use `sshpass -p`,
- never place the password on a command line.

Use the remote work directory from `MCP_REMOTE_WORKDIR`.

If the directory already exists, stop. Do not delete, overwrite, reset, or reuse it.

## 3. Install only required Ubuntu dependencies

After connecting, check:

```bash
command -v git
command -v curl
command -v python3
command -v uv
python3 --version
```

Install only missing required packages. If `sudo` is available:

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl git python3 python3-venv
```

The repository requires Python 3.12-compatible Python. Verify:

```bash
python3 --version
```

If Python 3.12 compatibility is not available, stop. Do not install an unrelated Python version or change system alternatives.

If `uv` is missing, install it with the official installer:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Load the installed binary path for the current session and verify:

```bash
uv --version
```

Do not install Docker, system services, firewall packages, unrelated Python packages, alternate Python versions, or `sshpass` on the remote server.

## 4. Clone the official repository

```bash
git clone -- \
  https://github.com/CiscoDevNet/cisco-ise-mcp-official.git \
  "$MCP_REMOTE_WORKDIR"

cd "$MCP_REMOTE_WORKDIR"
git remote get-url origin
git rev-parse HEAD
```

Verify that the origin is exactly:

```text
https://github.com/CiscoDevNet/cisco-ise-mcp-official.git
```

Record the actual commit ID. Do not modify repository files.

## 5. Create the remote `.env`

Create:

```text
$MCP_REMOTE_WORKDIR/.env
```

Populate it from the ISE values in `~/ise-mcp.log`:

```dotenv
ISE_IP=<value from ise-mcp.log>
API_PORT=443
API_USERNAME=<value from ise-mcp.log>
API_PWD=<value from ise-mcp.log>

ISE_VERIFY_SERVER_CERT=true
ISE_VERIFY_HOSTNAME=true

HOST=0.0.0.0
PORT=5000
```

Transfer the file through SSH stdin. Do not place credentials in command arguments.

Set restrictive permissions:

```bash
chmod 600 "$MCP_REMOTE_WORKDIR/.env"
```

Validate only pass/fail:

```text
ISE_IP present
API_USERNAME present
API_PWD present
API_PORT=443
PORT=5000
TLS verification enabled
```

Do not print values. Confirm `.env` is not tracked by Git.

## 6. Install application dependencies

```bash
cd "$MCP_REMOTE_WORKDIR"
uv sync --frozen
```

Verify:

```bash
test -x .venv/bin/python
.venv/bin/python --version
```

Do not run:

```text
uv add
uv remove
pip install
```

Do not modify `pyproject.toml`, `uv.lock`, or application source files.

If `uv sync --frozen` fails because the lock file is missing or inconsistent, stop and report the exact error. Do not regenerate or modify the lock file.

## 7. Start the MCP server temporarily

Confirm TCP port `5000` is free. Do not kill or replace an existing process.

Start the server:

```bash
cd "$MCP_REMOTE_WORKDIR"

nohup uv run server.py \
  > server.log 2>&1 < /dev/null &

echo $! > server.pid
```

Do not install a persistent service.

## 8. Validate through the MCP link

Use:

```text
http://<MCP_SERVER_ADDRESS>:5000/mcp/
```

Perform:

1. MCP `initialize`
2. `notifications/initialized`
3. `tools/list`
4. One safe, read-only ISE-backed MCP query

Expected transport results:

```text
initialize: HTTP 200
tools/list: HTTP 200
```

Do not invoke mutating tools.

Distinguish clearly between:

```text
MCP transport succeeded
```

and:

```text
ISE-backed data retrieval succeeded
```

Do not claim ISE connectivity based only on MCP initialization.

## 9. Stop the temporary process

When testing is complete:

- stop only the PID recorded in `server.pid`,
- verify that it stopped,
- do not use `pkill`,
- do not delete the checkout automatically.

## Final report

Return only this concise, sanitized report:

```markdown
# Cisco ISE MCP Server Setup Report

- Status: PASS / PARTIAL / BLOCKED / FAILED
- Timestamp UTC: <observed timestamp>
- Remote host: <non-secret identifier>
- Work directory: <remote path>

## SSH

- SSH connection: PASS / FAILED
- Host-key verification: PASS / FAILED
- Credentials exposed: NO

## Repository

- Official repository cloned: YES / NO
- Origin verified: YES / NO
- Commit: <observed commit>
- Existing checkout found: YES / NO
- Source modified: NO

## Dependencies

- Required Ubuntu dependencies: PASS / FAILED
- Python 3.12 compatibility: PASS / FAILED
- uv available: YES / NO
- uv sync --frozen: PASS / FAILED

## Configuration

- Local credential file: ~/ise-mcp.log
- API_PORT=443: CONFIRMED / NOT CONFIRMED
- PORT=5000: CONFIRMED / NOT CONFIRMED
- TLS verification enabled: YES / NO
- Credentials reported or committed: NO

## MCP validation

- MCP endpoint: <non-secret endpoint>
- initialize: PASS / FAILED
- tools/list: PASS / FAILED
- Read-only ISE query: PASS / FAILED / NOT RUN
- Actual ISE data returned: YES / NO

## Cleanup

- Temporary process stopped: YES / NO
- Persistent service installed: NO
- ISE configuration changed: NO
- Unrequested changes made: NO

## Issues

- <Observed issues only>
```

Report `PASS` only if SSH, cloning, dependency installation, `API_PORT=443`, `PORT=5000`, MCP initialization, tool discovery, and a read-only ISE-backed query all succeed. Otherwise report `PARTIAL`, `BLOCKED`, or `FAILED` with the exact observed reason.
