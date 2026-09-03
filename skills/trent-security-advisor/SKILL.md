---
name: trent-security-advisor
description: Get a principal security engineer's review of specific content — code, a diff, an implementation plan, a config, an architecture decision — from Trent's Security Advisor. Use this whenever the user asks "is this secure?", asks for a security review, or is about to write or present work touching authentication, authorization, secrets, sensitive data, network exposure, cryptography, IAM, or command execution, even when they never say the word "security". For the security of a whole repository or website rather than one piece of content, use trent-threats instead.
---

# Trent Security Advisor

The advisor answers from what you send — a diff, a plan, a config, a design
decision, a question — and needs no project set up first. Call it through the
`security_advisor` tool on the Trent MCP server; check the tool's description
before calling for what it returns, what its fields mean and when it refuses.

## Is it worth a consultation?

The advisor is a consultation you choose, not a gate. Weigh it on one
question: **can this review still change a decision?**

Worth it — a new auth path, a change to how a secret is handled, an endpoint
going public, a schema that starts holding personal data, an implementation
plan you are about to hand back to the user. Not worth it — a typo, a README
edit, a rename, a change whose security shape you already know is nil.

When the user tells you to skip it, skip it.

## Running the consultation

### Step 1: Gather the context the code cannot show

The advisor sees only what you send. Ask about what the content does not
reveal:

- What already protects this? (an API Gateway authorizer, a WAF, network
  isolation)
- Is anything public on purpose?
- How sensitive is the data involved?

Skip any question the code already answers, or the user already has.

If the context and the code disagree — the user says "no PII" and the code
handles SSNs — say so and settle it first.

### Step 2: Send the thing being decided, not the repository around it

Send only the content under review; a whole codebase is what a scan is for.

For a file too large to sit in the context window, upload it with
`prepare_file_upload` and pass the returned reference in `content_document`
rather than the bytes.

### Step 3: Call `security_advisor`

Fill what you have: the content to review, the `context` you gathered, and the
user's `question` if they asked one.

Pass `thread_id` to continue a conversation the advisor already has — when it
asked you something, or when you are iterating on content it has seen. Start a
fresh one for a new topic.

### Step 4: Do something with the answer

Attribute it to "Trent Security Advisor" and keep its severity labels
(CRITICAL / HIGH / MEDIUM / LOW).

Frame it as what it is: focused feedback on the content you sent. If the user
asked "is my whole app secure?", say this review answers a narrower question,
and offer a scan for the rest.

Then act: say which findings you will act on, which you dispute and why, and
which need a decision from the user.

## When it does not go straightforwardly

**The tool is not available.** Tell the user the security consultation could
not run, carry on with the task, and do not pass your own opinion off as
Trent's review.

**The reply carries `detected_repository` and an `onboarding_hint`.** No Trent
project covers this repository, so the review drew on no existing findings. Say
that, offer to set the repository up with `create_project`, and follow
`trent-threats` from there.

## Example

```
User: /trent-security-advisor [selects the auth middleware they just wrote]

Claude: Before I send this over — is anything in front of this middleware
        already doing auth, like an API Gateway authorizer?

User: Yes, API Gateway validates the JWT. This is authorization only.

Claude: [Calls security_advisor with the middleware as the content and
         "API Gateway validates JWTs upstream; this layer does
         authorization only" as the context]

        Trent Security Advisor, on your authorization middleware:

        HIGH — the role check runs after the resource is loaded, so an
        unauthorized caller can still infer the record exists.
        MEDIUM — no audit log on a denied decision.

        I'll move the role check ahead of the load. The audit log needs a
        decision from you: do we have a sink for it yet?
```
