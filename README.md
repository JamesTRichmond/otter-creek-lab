# Otter Creek Homelab

A small self-hosted lab in Vermont where my AI agents, local models, and test sandboxes run. It's described here by **role**: no addresses, hostnames, products, or configs. That's on purpose.

## Why it exists

- Run open-weight models locally, so long agent loops cost electricity, not API credit.
- Give coding agents somewhere disposable to run untrusted code, away from my laptop and my accounts.
- Practice the operations side: virtualization, networking, backups, runbooks, incident cleanup.

## Architecture

```mermaid
flowchart LR
  L["Laptop<br/>review + merge"] -->|tasks| R
  subgraph NET["Segmented home network"]
    subgraph AH["Agent host · trusted"]
      R["Relay loop<br/>bounded, redacted results"]
      M["Local models"]
    end
    subgraph HV["Hypervisor"]
      subgraph SB["Isolated sandbox · untrusted"]
        S["Disposable sandbox VMs<br/>destroyed after each run"]
      end
    end
  end
  R --> M
  R -->|"code + tests, secrets excluded"| S
  S -->|results| R
  L -.->|"review + merge"| G["GitHub<br/>protected main, required review"]
  K["Scheduled health checks"] -->|failures open issues| G
  classDef trusted fill:#5B54C7,stroke:#403A96,color:#FFFFFF
  classDef untrusted fill:#C3512C,stroke:#93391C,color:#FFFFFF
  classDef ops fill:#15866B,stroke:#0C6450,color:#FFFFFF
  classDef neutral fill:#646C76,stroke:#4B535C,color:#FFFFFF
  class R,M trusted
  class S untrusted
  class K ops
  class L,G neutral
  style NET fill:transparent,stroke:#768390
  style HV fill:transparent,stroke:#768390
  style AH fill:transparent,stroke:#7A73D6
  style SB fill:transparent,stroke:#D46A42
  linkStyle default stroke:#768390,stroke-width:1.5px
```

| Role | What it does |
|---|---|
| Agent host | Serves open-weight models locally and runs the relay loop |
| Hypervisor | Holds disposable sandbox VMs that are thrown away and rebuilt |
| Sandbox VMs | Built from a hardened template with resource caps. Isolated from the rest of the lab and destroyed after each run. They never see my keys |
| Network | Segmented home network, lab separated from household devices |

## Featured: the relay loop

A tag-team coding loop that runs entirely on local models:

1. A **builder** model writes code and tests for a task.
2. The code goes to a fresh sandbox VM and the tests run there under a hard timeout.
3. Two **reviewer** models critique the result.
4. Repeat until the tests pass and both reviewers are clean, or the round budget runs out.

What makes it trustworthy:

- **Secrets never leave the host.** Credential files are excluded from every copy sent to a sandbox.
- **Sandbox output is treated as hostile.** Results are size-bounded and stripped of anything that looks like a credential before a reviewer model reads them.
- **Infrastructure failures aren't blamed on the model.** A dropped connection, a timeout, or the wrong worker answering is retried once, then the run stops with a distinct exit code. The builder only gets a failed round for its own code.
- **Every outcome is classified.** Pass, failing tests, timeout, no tests collected, missing dependency, nothing saved. Each lands in a per-round log.
- **Runaway tests get killed.** A deliberately hanging test is part of the examples, and it must be killed by the timeout.

## Operations

- **A living runbook.** The lab's runbook is a live page that a scheduled agent sweeps every night. It folds in what changed, restores anything missing, and trims stale steps. A static runbook goes stale in a week.
- **Checks that fail loud.** Scheduled health checks open a GitHub issue on any failure, so nothing breaks quietly.
- **Nightly backups** with a rolling retention window.
- **Guardrails on the repos agents touch.** Branch protection on main, required review, no force pushes, linear history. Agents open PRs; I merge.
- **Incident practice.** I've already worked real incidents end to end: a credential rotation and a runaway API-cost leak, each traced to root cause and closed.

## Next

Hardening the lab further and making each trust boundary physical where that is cheap to do, using reused hardware. Details are kept private.

— [James Richmond](https://github.com/JamesTRichmond)
