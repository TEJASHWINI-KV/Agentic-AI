# Interview Preparation — Q&A

A structured set of interview answers covering day-to-day work, behavioural questions, and technical deep-dives based on my work at Kyndryl.

## Table of Contents

1. [Walk me through your day-to-day](#q1-walk-me-through-your-day-to-day)
2. [Why are you looking for a change?](#q2-why-are-you-looking-for-a-change)
3. [What's the difference between a role and a playbook?](#q3-whats-the-difference-between-a-role-and-a-playbook)
4. [How do you handle sensitive data in Ansible?](#q4-how-do-you-handle-sensitive-data-in-ansible)
5. [How do you handle Windows automation through Ansible?](#q5-how-do-you-handle-windows-automation-through-ansible)
6. [Where do you see yourself in 3 years?](#q6-where-do-you-see-yourself-in-3-years)
7. [How do you manage version control for Ansible code?](#q7-how-do-you-manage-version-control-for-ansible-code)
8. [Walk me through your GitHub Actions pipeline](#q8-walk-me-through-your-github-actions-pipeline)
9. [Explain the Server Provisioning asset](#q9-explain-the-server-provisioning-asset)
10. [Explain the SOCKS tunnel role](#q10-explain-the-socks-tunnel-role)
11. [Tell me about your Agentic AI work](#q11-tell-me-about-your-agentic-ai-work)
12. [Tell me about the GEMU migration](#q12-tell-me-about-the-gemu-migration)

---

## Q1: Walk me through your day-to-day

On a typical day, I start by reviewing my sprint tasks and checking for any open tickets or customer updates. I mainly work on automation use cases where I enhance and integrate customer requirements into existing playbooks. I also perform peer reviews to ensure code quality, linting checks, and compliance with security standards. In parallel, I handle incident management from P2 to P4, ensuring SLA adherence and system stability. I collaborate with cross-functional teams whenever needed and provide regular updates on task progress and issue resolution.

---

## Q2: Why are you looking for a change?

I've spent the last 2.5 years at Kyndryl, and honestly it's been a great learning ground — I started as a fresher and got the chance to work on real enterprise automation, build playbooks that run across hundreds of servers, and own things end-to-end from design to production.

But over time I've started feeling that my work has settled into a similar pattern — the same set of tools, the same kind of use cases. I want to step into an environment where I can apply what I've learned so far to **different industries, different problem statements, and a wider technology stack** — maybe more cloud-native work, more exposure to client-facing delivery, and the chance to learn from a new set of engineers.

So it's not about moving away from Kyndryl — it's more about taking the foundation I've built here and growing it in a place where the canvas is bigger. That's really the next step I'm looking for.

---

## Q3: What's the difference between a role and a playbook?

*(To be filled in.)*

---

## Q4: How do you handle sensitive data in Ansible?

Honestly, this is something we treat as non-negotiable at Kyndryl — sensitive data never sits in plaintext anywhere.

The way I handle it depends on where the secret lives:

- For anything that needs to ship with the playbook — like service account passwords or certificate paths — I use **Ansible Vault** to encrypt those variable files. So even if someone clones the repo, all they see is encrypted blobs.
- For cloud-resident secrets, I integrate with **Azure Key Vault** and pull them at runtime — that way credentials rotate centrally without me touching the playbook.
- One thing people often forget — even with encryption, secrets can leak through logs. So on any task that touches sensitive variables, I set `no_log: true` so it doesn't print to stdout or AAP job output.
- The vault password itself? Never in the repo. We inject it through **AAP credentials** at execution time, so the password lives in the platform, not in code.

---

## Q5: How do you handle Windows automation through Ansible?

Yes, Windows automation is genuinely different from Linux — and the three areas I always pay attention to are **connectivity, modules, and path handling**.

- **Connectivity** — Linux uses SSH, but Windows uses **WinRM**, so the setup is completely different.
- **Modules** — I can't use the same modules as Linux. Windows has its own native set: instead of `copy`, I use `win_copy`; instead of `service`, I use `win_service`; for scheduled jobs, I use `win_scheduled_task`. These modules understand Windows internals — registry, services, the task scheduler — natively.
- **Path handling** — Windows uses backslashes and drive letters, so paths have to be quoted carefully and module-specific path conventions followed.

---

## Q6: Where do you see yourself in 3 years?

Three years out, I see myself growing into a **Senior Engineer or Automation Architect** role — leading end-to-end automation strategy, mentoring junior engineers, and shaping how DevSecOps and AI-driven automation come together in modern delivery. I want to be the person who brings both technical depth and architectural thinking, not just delivery.

---

## Q7: How do you manage version control for Ansible code?

We follow **Git-flow** strictly at Kyndryl:

- `main` is treated as production-ready only — nothing lands there without going through the full pipeline.
- All feature work happens in short-lived **feature branches** off `develop`.
- Every merge goes through a **pull request** with mandatory peer review and passing CI checks — lint, syntax, and dry-run validation.
- Releases are then **tagged semantically** so we can trace exactly what's running where.

---

## Q8: Walk me through your GitHub Actions pipeline

At Kyndryl, we use a **centralised reusable workflow** approach — there's a shared repo called `ce-workflows` that hosts all the standard CI workflows, and every project just calls them.

In my Ansible projects, I plug in four checks that run on every push and pull request:

| Check | Purpose |
|-------|---------|
| **Ansible Lint** | Checks playbook quality against best practices |
| **Markdown Lint** | Checks README and documentation files |
| **Extra Linters** | Covers Python, PowerShell, and Shell scripts in the repo |
| **CodeQL** | Scans for security vulnerabilities, mainly in our Python modules |

All four run in **parallel**, and no PR can merge into `main` unless every check passes and a teammate has reviewed it.

**Impact** — 40% drop in pipeline failures and post-deployment defects. And the best part: when the platform team updates a workflow, every project picks it up automatically — no manual work for me.

---

## Q9: Explain the Server Provisioning asset

One of the main assets I worked on is called **GAT Server Provisioning Consumption**. It's a Kyndryl Ansible automation that takes care of VM provisioning on VMware vCenter from end to end.

The way it works:

1. A user raises a VM request in **ServiceNow**.
2. That request flows into **Ansible Automation Platform**, which kicks off a workflow.
3. The workflow downloads the right **golden image** to the bastion and imports it into vCenter.
4. It then creates the VM with the requested CPU, memory, disks, and network.
5. Joins it to **Active Directory** if it's Windows.
6. Installs standard tools like **Trendmicro and ITM**.
7. Formats the additional disks.
8. Finally updates **ServiceNow** with the status and **CMDB** entry.

So basically what used to be a 4–5 hour manual task for an admin now finishes in about **15 minutes**.

---

## Q10: Explain the SOCKS tunnel role

In our Ansible Tower setup, the automation control node and the target servers don't sit in the same network — there are usually **1 to 4 jumphosts** in between because of customer firewall zones. So before any playbook can reach a Linux or Windows endpoint, we need a network path through those jumphosts.

To solve that, our team has a shared Ansible role called **`ansible-role-event-socks-tunnel`**. What it does is — it runs on the Tower job pod, takes the jumphost details and SSH keys from Tower credentials, and uses `ssh -D` to set up a **SOCKS5 tunnel** that chains through each jumphost using `ProxyCommand`. After that, every SSH or PSRP connection from the playbook goes through that one tunnel.

I contributed to this role in a few ways:

- **Multi-hop `ProxyCommand` chaining** — made sure the same role works for 1-hop, 2-hop, 3-hop, and 4-hop setups, because different customer accounts have different network depths.
- **Passphrase-protected SSH keys** — added support using `ssh-agent` and `ssh-add` with `expect`, so accounts with stricter security policies could still use the role.
- **Port-based tunnel option** (`-D job_pod_port`) — an alternative to the default Unix-socket tunnel. This was important for large Linux inventories, because opening a fresh SSH connection per host was triggering SSH brute-force detection on the customer's firewall. With a single SOCKS port, all host connections multiplex through one tunnel.
- **Cross-platform variable handling** — `ansible_ssh_common_args` for Linux and `ansible_psrp_proxy` for Windows — so the same role serves both inventories.

Today this role is used as a dependency in pretty much every Kyndryl GAT automation asset that has to talk to customer endpoints — server provisioning, SSL cert scanning, inventory management, patching — all of them import it as the first task.

---

## Q11: Tell me about your Agentic AI work

Apart from my regular Ansible work, I've also been building a small **AI agent project** on the side. The idea came from a simple problem — whenever a P1 or P2 incident comes in, the on-call engineer spends almost half an hour just reading logs, checking past tickets, and figuring out what to do. I wanted to automate that first triage step.

So I built an agent using **LangGraph**, which is a framework for orchestrating LLM workflows. My agent has **six steps**:

`validate → extract → retrieve_logs → combine → generate_analysis → notify`

It validates the incident details, pulls the relevant logs, combines everything into a structured prompt, sends it to an LLM for analysis, and then posts the recommended fix to the on-call team on **Microsoft Teams**.

A few things I added to make it production-ready:

- A **provider layer** so the agent can run on OpenAI, Anthropic, or an on-prem model — and there's a mock mode for testing, so CI runs don't cost money.
- I forced the LLM to return **structured JSON output** instead of free text, which removed almost all hallucinations and made the Teams card parsing reliable.
- A **human-in-the-loop step** — the agent only suggests the fix; a human approves it before any Ansible playbook actually runs.
- **Fallback logic** — if the LLM times out or the log is too big for the context window, the agent still returns a safe recommendation from a static runbook, so the engineer isn't left empty-handed.

What I like about this project is that it isn't just a chatbot — it sits on top of the same Ansible automation I already build, so the agent's suggestions can turn into **real, executable actions** in production. It's basically connecting traditional automation with AI-driven decision-making, which is the direction I want to keep growing in.

---

## Q12: Tell me about the GEMU migration

One of the larger initiatives I led at Kyndryl was the migration of our team's repositories to **GitHub Enterprise Managed Users (GEMU)**.

The background — Kyndryl was moving away from the old GitHub Enterprise setup where developers used their personal-style GitHub accounts, to a **fully company-managed identity model** where every user is provisioned and de-provisioned through corporate SSO. The goal was tighter security, centralized access control, and clean audit trails — basically aligning Git access with the same identity governance as the rest of our enterprise tools.

For my team, that meant moving **50+ Ansible and Python automation repositories** from the legacy org into the new GEMU-backed org, without breaking anything that depended on them — and a lot of things depended on them, because these repos feed directly into AAP job templates running in production.

The way I approached it:

1. **Inventory and dependency map** — which repos were active, who owned them, which AAP projects pointed to them, and what CI workflows referenced them.
2. **Repo-by-repo cut-over** — cloning into the new org, preserving full Git history, branches, tags, and PR references, and re-applying branch protection rules and CODEOWNERS.
3. **Update AAP and workflows** — updated all AAP project URLs and SCM credentials to point to the new org, and re-pointed every GitHub Actions workflow that referenced the old org — especially the shared reusable workflows from our `ce-workflows` repo.
4. **Rotate credentials** — rotated deploy keys, webhooks, and service-account tokens because the old credentials no longer worked under GEMU identity.
5. **Documentation** — documented the whole process as a runbook so other teams could follow the same steps for their own migrations.

The migration was completed with **zero downtime** — automation kept running in production throughout, and we improved governance because now every commit is tied to a managed corporate identity. It also made onboarding and offboarding much cleaner — when someone leaves the team, access is revoked automatically through SSO instead of someone manually cleaning up Git permissions.
