# Evidence-Preserving Sync Rules

Nine Constraints for Sync That Doesn't Lose Meaning (or User Trust)

These rules describe limit conditions for synchronization in systems that preserve evidence over time while remaining usable, debuggable, and trustworthy to users.

They are framed as design constraints, not absolutes: systems may approximate them to varying degrees, with corresponding tradeoffs in reliability, debuggability, and trust.

---

## Rule 0: The Log Is the Database

Everything else is a view.

The append-only operation log is the single source of truth. Local state, UI, and caches are all derived from replaying operations. This architectural commitment makes the other nine rules coherent.

---

## Rule 1: Distinguish Recorded Operations from Derived State

Synchronization SHOULD operate on recorded operations or claims, not on reconstructed snapshots of state.

State is a derived interpretation. Syncing derived state risks overwriting evidence and obscuring causality.

### Engineering implication

- Sync logs, events, or operations
- Rebuild state locally from those records
- Treat snapshots as caches, not authorities

### Why

You can replay operations. You can't interrogate a snapshot about why it looks the way it does.

---

## Rule 2: Preserve the Origin of Every Write

Every synchronized write SHOULD retain information about where it originated, when, and under what context.

Sync that strips origin collapses distinct perspectives into an undifferentiated stream.

### Engineering implication

- Tag operations with origin (device, user, system)
- Do not "launder" local writes into server-authored state
- Avoid patterns where all data appears to come from the server

### Why

Sync without origin destroys the ability to reason about conflicts, intent, or error.

---

## Rule 3: Allow Concurrent Contributions Without Forced Resolution

The sync layer SHOULD allow concurrent or conflicting operations to coexist until explicitly resolved.

Resolution is an interpretive act, not a transport requirement.

### Engineering implication

- Avoid last-write-wins as a default
- Avoid silent overwrites during merge
- Prefer structures that preserve concurrent intent

### Why

Forced resolution optimizes for neatness at the cost of truth.

---

## Rule 4: Make Sync Status Explicit and Inspectable

Users and developers SHOULD be able to see the status of synchronization at a meaningful granularity.

Hidden sync state erodes trust and makes failures impossible to diagnose.

### Engineering implication

- Distinguish: local-only, pending sync, synced, failed
- Expose queues, retries, and failures
- Avoid "eventual consistency" that is invisible to users

### Why

Trust depends less on perfection than on legibility.

---

## Rule 5: Local Intent Must Not Be Silently Overwritten

Locally recorded intent SHOULD remain visible and preserved until explicitly superseded or reconciled.

Sync should not erase what a user just did without acknowledgement.

### Engineering implication

- Never overwrite unsynced local operations with remote state
- If reconciliation is needed, surface it as such
- UI reflects local intent first, not remote consensus

### Why

A system that appears to forget user actions feels broken, even if it is "technically consistent."

---

## Rule 6: Sync Failures Must Be Explicit and Recoverable

Synchronization failures SHOULD surface clearly and provide a path to recovery.

Silent failure is worse than visible error.

### Engineering implication

- No fire-and-forget writes
- Failed syncs remain queued and inspectable
- Users can retry, resolve, or discard intentionally

### Why

Evidence that disappears silently is indistinguishable from corruption.

---

## Rule 7: Operations Should Be Safe to Retry

Synchronized operations SHOULD be idempotent or otherwise safe to replay.

Unreliable networks are the norm, not the exception.

### Engineering implication

- Deduplicate by operation ID or content hash
- Expect retries, reordering, and duplication
- Never rely on "exactly once" delivery

### Why

Sync robustness comes from tolerance, not precision.

---

## Rule 8: Support Offline Recording as a First-Class Case

Systems SHOULD allow operations to be recorded and accumulated without network connectivity, then synchronized later without loss.

Offline behavior is not an edge case; it is an environmental condition.

### Engineering implication

- Persistent local queues
- Deterministic merge behavior
- No requirement for real-time authority to accept writes

### Why

If evidence can only be recorded when connected, the system controls reality instead of recording it.

---

## Rule 9: Deletion Is a Recorded Act, Not an Erasure

Deletion SHOULD be represented as an explicit operation, not as the disappearance of prior records.

What "deleted" means may differ by frame, time, or authority.

### Engineering implication

- Use tombstones or equivalent markers
- Prevent deleted items from silently reappearing
- Preserve the fact that deletion occurred

### Why

Erasure destroys auditability; recorded deletion preserves intent without lying about history.

---

## Conformance Levels

| Level | Rules | What You Get |
|-------|-------|--------------|
| **Minimum viable** | 0, 1, 5, 6 | Data doesn't vanish silently |
| **Trustworthy** | + 2, 4, 9 | Users trust the save button |
| **Robust** | + 3, 7, 8 | Works offline, survives chaos |

---

*December 2025*
