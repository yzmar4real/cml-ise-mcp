# CML Cisco ISE MCP

This repository contains a Cisco Modeling Labs (CML) proof-of-value lab for a Cisco ISE Model Context Protocol (MCP) integration. It provides the lab definition, a safe operating guide, and a credential-file example without storing live credentials in Git.

The intended workflow is:

1. Import the CML lab definition.
2. Keep SSH and Cisco ISE values in a local, ignored `mcp.log` file.
3. Configure or validate the MCP service using `agent.md`.
4. Use the MCP endpoint for approved, read-only Cisco ISE information retrieval.

This repository does not contain the Cisco ISE MCP application source code. The lab definition supplies the infrastructure context in which that service can run.

## Lab topology

`Cisco_ISE_MCP_LAB.yaml` preserves a five-node, four-link topology:

- `ise-0`: Cisco ISE node.
- `ext-conn-0`: CML external connector.
- `unmanaged-switch-0`: shared lab switch.
- `ISE-MCP`: Ubuntu host for the MCP service.
- `AI_AGENT`: Ubuntu host for the agent-side workflow.

The external connector, Cisco ISE node, MCP host, and agent host connect through the unmanaged switch. The YAML is suitable for import into Cisco Modeling Labs; image definitions and local CML resource choices must be available in the target CML installation.

## Repository files

- `Cisco_ISE_MCP_LAB.yaml` — CML lab definition with credential fields replaced by obvious placeholders.
- `agent.md` — safe, read-only operating and validation workflow.
- `mcp.log.example` — dotenv-style configuration template containing placeholders only.
- `.gitignore` — excludes the local credential file and common local credential artifacts.

## Validation screenshots

The 16 credential-safe validation screenshots below are committed in `docs/screenshots/`. The images are sanitized for publication and do not include raw credential material.

1. [Validation image 01](docs/screenshots/ise-mcp-image01-sanitized.png)
2. [Validation image 02](docs/screenshots/ise-mcp-image02-sanitized.png)
3. [Validation image 03](docs/screenshots/ise-mcp-image03-sanitized.png)
4. [Validation image 04](docs/screenshots/ise-mcp-image04-sanitized.png)
5. [Validation image 05](docs/screenshots/ise-mcp-image05-sanitized.png)
6. [Validation image 06](docs/screenshots/ise-mcp-image06-sanitized.png)
7. [Validation image 07](docs/screenshots/ise-mcp-image07-sanitized.png)
8. [Validation image 08](docs/screenshots/ise-mcp-image08-sanitized.png)
9. [Validation image 09](docs/screenshots/ise-mcp-image09-sanitized.png)
10. [Validation image 10](docs/screenshots/ise-mcp-image10-sanitized.png)
11. [Validation image 11](docs/screenshots/ise-mcp-image11-sanitized.png)
12. [Validation image 12](docs/screenshots/ise-mcp-image12-sanitized.png)
13. [Validation image 13](docs/screenshots/ise-mcp-image13-sanitized.png)
14. [Validation image 14](docs/screenshots/ise-mcp-image14-sanitized.png)
15. [Validation image 15](docs/screenshots/ise-mcp-image15-sanitized.png)
16. [Validation image 16](docs/screenshots/ise-mcp-image16-sanitized.png)

## Future article

**Substack article:** Link will be added after publication.

## Prerequisites

- A Cisco Modeling Labs installation with the required Cisco ISE and Ubuntu images.
- A supported Cisco ISE environment reachable from the lab when an ISE-backed check is approved.
- An MCP service checkout and runtime prepared according to the service's own documentation.
- An operator-controlled local credential file created from `mcp.log.example`.

The MCP service convention used by this lab is HTTPS/API access to Cisco ISE on port `443` and an MCP listener on port `5000`. Confirm the actual service configuration before use; do not infer connectivity from transport initialization alone.

## Local credential handling

Create the local file only on the operator's machine:

```bash
cp mcp.log.example mcp.log
chmod 600 mcp.log
```

Replace the placeholders locally. Do not commit, upload, paste, print, or include the file in diagnostics. `.gitignore` excludes `mcp.log`, dotenv files, common private-key extensions, and common credential paths. Read only the named values required for an approved operation; do not use `source`, `.`, or `eval` on the file.

If the MCP service needs a remote `.env`, generate it through a protected channel, set restrictive permissions, and ensure it is ignored by the remote repository. Never put credentials in command-line arguments, service-unit text, logs, or MCP client messages.

## Safe operating workflow

Read `agent.md` before operating the lab. The workflow is deliberately read-only for Cisco ISE and MCP validation:

- Verify the target and current state before making any change.
- Keep SSH host-key verification enabled.
- Use only the approved Cisco ISE MCP service and its documented endpoint.
- Prefer `initialize`, `tools/list`, health checks, and explicitly read-only queries.
- Do not invoke mutating MCP tools or change Cisco ISE configuration.
- Do not alter firewall, DNS, routing, TLS, or CML lab lifecycle settings as part of validation.
- Stop and request approval before any service-impacting, credential-changing, or externally visible action.
- Return only a sanitized pass/fail report; never include secrets, private keys, private addresses, raw logs, or complete environment files.

## Validation

From a clean checkout, these checks validate the repository artifacts without using credentials:

```bash
git diff --check
python - <<'PY'
import yaml
from pathlib import Path
lab = yaml.safe_load(Path('Cisco_ISE_MCP_LAB.yaml').read_text())
assert len(lab['nodes']) == 5
assert len(lab['links']) == 4
print('YAML topology: 5 nodes, 4 links')
PY
```

For an approved runtime validation, distinguish these results in the report:

- MCP transport initialized successfully.
- MCP tools were listed successfully.
- A specifically approved, read-only Cisco ISE query returned data.

The first two results do not prove that Cisco ISE data retrieval succeeded.

## Security and history

This commit removes the tracked `mcp.log` artifact and replaces credential values in the CML definition with placeholders. Any credentials that were previously exposed must still be rotated by the credential owner. Removing a file in a new commit does not erase historical Git objects; history rewriting or secret-history purging is a separate, coordinated operation and is not performed by this repository change.

## Ownership boundary

This repository documents infrastructure context and safe host-side validation for the CML Cisco ISE MCP lab. CML application software, lab lifecycle, images, console validation, pyATS workflows, and network-device operations remain the responsibility of the network/lab owner. Infrastructure changes require explicit approval and must remain scoped to the hosting layer.
