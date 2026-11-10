# Topic of the day: should your SDK retry writes? (2026-11-10)

Today's digest has a small line about `databricks-sdk 0.163.0` that deserves a closer look. Per its release notes, POST requests no longer retry after a read timeout, because the server may already have handled the first attempt. It's a good reminder that retries are not free.

## The problem

A read timeout means your client stopped waiting. It does not mean the server stopped working. If the request was something like "create a job" or "send a message" and you quietly send it again, you can end up with two jobs or two messages.

Reads are different. Asking for the same data twice is harmless, so SDKs retry GET requests freely. Writes are where it gets risky.

## What to do in your own code

1. **Know which calls are safe to repeat.** Reading, listing and looking something up are fine. Creating, charging and sending are not, unless the API says otherwise.
2. **Use idempotency keys when the API offers them.** You send a unique key with the request, and the server uses it to spot a repeat and return the first result instead of doing the work again.
3. **After a timeout on a write, check before you retry.** Look up whether the thing you tried to create already exists, then decide.
4. **Log retries.** If a job ran twice, you want to be able to see why.

## Quick check for your stack

```python
from databricks.sdk import WorkspaceClient

w = WorkspaceClient()
# Read calls like this are safe to retry:
for job in w.jobs.list(limit=5):
    print(job.job_id)
```

Source for the SDK change: [databricks-sdk 0.163.0 release notes](https://github.com/databricks/databricks-sdk-py/releases/tag/v0.163.0).

## Try it today

Search your own code for places that wrap a create or send call in a retry loop. For each one, ask what happens if the first attempt actually worked.
