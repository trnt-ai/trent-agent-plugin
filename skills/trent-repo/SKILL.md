---
name: trent-repo
description: Work out which Trent project covers the git repository the user is sitting in, then answer their security question about it. Use this whenever the user asks about threats, vulnerabilities, findings, security posture or scan status for "this repo", "the current repo", "my repo", "this codebase" or "the project I'm working on" — any time they mean the repository in front of them and have not named a Trent project. Once the project is resolved, trent-threats carries the scanning and remediation workflow.
---

# Trent — the Current Repository

Resolve the repository the user is sitting in to the Trent project that covers
it: match the wrong one and you show the user unrelated findings, with no
sign that anything went astray. Once you have the project, the workflow around
it — scanning, reading results, remediation — is the `trent-threats` skill.

## Step 1: Detect the repository

Read the git remote:

```bash
git remote -v | grep origin | head -1 | awk '{print $2}'
```

Both remote forms give the same identifier:

- `git@github.com:acme/payment-service.git` → `acme/payment-service`
- `https://github.com/acme/payment-service.git` → `acme/payment-service`

If there is no remote, or the command fails, ask the user which repository they
mean. Do not fall back to the directory name.

## Step 2: Match it to a Trent project

Call `list_projects` and find the project carrying that repository. Prefer an
exact `owner/repo` match over a partial one, and confirm a match made on a bare
name the user typed ("payment-service") before you act on it.

- **Nothing matches** — say that no Trent project covers this repository, and
  offer to set one up with `create_project`. From there, `trent-threats` has
  the scan loop.
- **Several match** — ask the user which one rather than picking.

## Step 3: Answer the question

Use the matched project and route on what the user asked. The common cases:

- threats or vulnerabilities → `get_security_posture`
- controls, remediation, "what should I fix?" → `review_plan`
- scan progress or status → `get_scan_status`
- start a scan → `trigger_analysis`

`trent-threats` has the full intent-to-tool map, and the rules for presenting
each kind of result. Approving a plan or a paused scan phase is the user's
call: present, ask, end your turn, and call the approval tool only once they
have replied.

## Step 4: Lead with what you resolved

Put the resolution above the answer:

```
Repository: acme/payment-service
Trent project: payment-service (last scanned at commit 4f2a91c)
```

Then the answer itself — findings grouped by severity, CRITICAL and HIGH first;
controls as a status breakdown.

## Example

```
User: What are the threats in my current repo?

Claude:
1. [git remote -v → acme/payment-service]
2. [list_projects → project "payment-service" carries that repo]
3. [get_security_posture with that project]

Repository: acme/payment-service
Trent project: payment-service (last scanned at commit 4f2a91c)

7 open findings — grade C, grouped by severity.

Want the remediation plan?
```

## Tips

- Resolve the repo once and reuse it for the rest of the conversation.
- The findings describe the commit Trent last analysed, not the working tree.
  If the user has moved on since, say so, and offer `trigger_analysis` once
  their work is committed.
