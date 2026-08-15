# Adversarial Fixtures

These test FlowBreaker's **judgement**, not its coverage. Each targets a specific
failure mode. Run them after any change to §1, §5 or §6.

| Fixture | FlowBreaker must | Failure mode it guards against |
|---|---|---|
| `01-contradiction.md` | Report the conflict as `contradictory` and ask which is correct | Silently picking one side and building on it |
| `02-no-problem.md` | Refuse to proceed past step 1 | Inventing a plausible user problem to keep moving |
| `03-permission-gap.md` | Raise a `critical` permission question | Treating an unwritten access rule as "presumably fine" |
| `04-complete.md` | **Find no critical questions** | Manufacturing findings to look thorough |

## `04-complete.md` is the important one

Recall is easy. Any sufficiently anxious reviewer finds five criticals in any
document. Precision is what makes the tool worth running twice.

A tool that returns five criticals on every input teaches its user to skim past all
five — including the one that mattered. If FlowBreaker invents criticals for
`04-complete.md`, that is a **bug of the same severity** as missing a real defect in
the other three, and it should be treated as one.

`04-complete.md` is deliberately not perfect — no real spec is. It has genuine
`medium` and `low` gaps. The correct output is a short review that reports those
honestly, finds no criticals, and says so plainly. "No critical gaps found in
permissions" is a valid and valuable result (R7).

## Running them

```
Follow FLOWBREAKER.md to review examples/adversarial/01-contradiction.md
```

Check the expected behaviour in the table above. There are no golden outputs
committed for these — the assertion is behavioural, and pinning exact output would
make them brittle for no benefit.
