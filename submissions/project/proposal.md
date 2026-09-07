# Video service and OS identification through network data
### A feature-leakage audit of two traffic-classification benchmarks

Winston Li and Chris Furlong

## Project summary

Reported accuracies on network traffic classification benchmarks sit close to ceiling, but it is not always clear what the model has learned. Packet-level representations such as nPrint expose every header bit, including fields that identify the *host* rather than its *behavior*: source and destination addresses, port numbers, and TCP sequence and acknowledgment numbers. A classifier that keys on these will score well on a held-out split drawn from the same capture and then fail on any other network. The nPrint OS detection benchmark already disallows exactly these fields, which is an acknowledgment that the risk is real. The size of the effect, however, has not been measured, and it is not known whether other tasks on the leaderboard share the problem. We propose to quantify it across two tasks using one shared pipeline.

## Data

Two pcapML-encoded datasets from the public benchmark suite, roughly 3 GB uncompressed in total. **nPrint OS detection** (<1 GB): 100-packet samples labeled with the sending host's operating system, 13 classes, derived from the CICIDS capture; the benchmark disallows IPv4 source and destination addresses, TCP ports, and TCP SEQ/ACK numbers. **nPrint streaming video services** (<2 GB): identify which of four video services generated the SYN packets in a session, with addresses CryptoPAn-anonymized. Both decode through `pcapML_FE`, which gives a common sample iterator and lets a single harness serve both tasks. Balanced accuracy is the specified metric for each.

## Machine learning

For each dataset we evaluate the same four-rung feature-restriction ladder:

- **R0, unrestricted.** Full nPrint including the disallowed identifier fields. Deliberately illegal; establishes the leakage ceiling.
- **R1, benchmark-legal.** Full nPrint minus the disallowed fields. Directly comparable to the published leaderboard entry.
- **R2, behavior only.** Header fields with all remaining identifier-like fields removed, including IP ID.
- **R3, cheap baseline.** Roughly six hand-picked features: TTL, TCP window size, flags, options length, packet length, inter-arrival time.

At every rung we fit a leaderboard-comparable model (AutoGluon, or random forest and gradient boosting if compute is tight) alongside a logistic regression, giving a 4 × 2 grid per dataset. Splits are grouped by source host wherever host identity is recoverable, so the same host cannot appear in both training and test. That grouping is itself a leakage control, and we report its effect.

## Evaluation

Balanced accuracy is the primary metric, matching both leaderboards, reported with macro-F1 and confusion matrices. The headline figure plots balanced accuracy against restriction rung, one line per dataset and model class, with the published leaderboard result drawn as a reference. We also record feature-extraction and training time at each rung so the result carries a cost axis as well as an accuracy axis, and we run permutation importance at R1 to name the fields carrying the surviving signal.

## Learning objective

Three things: how much reported accuracy survives removal of identifier-like fields; whether the pattern holds across two unrelated tasks or is an artifact of one capture; and whether a six-feature logistic model lands within a few points of a leaderboard model at a fraction of its cost. 

## Division of labor

Chris builds the shared harness (loader, restriction switch, model wrappers, metrics) and runs OS detection. Winston runs streaming video service identification and owns the Sphinx report build. Each reviews the other's notebook end to end before submission. 

