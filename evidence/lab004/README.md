# LAB-004 Evidence Inventory

All three images are operator-saved screenshots located in the local lab folder and copied without editing. The assistant inspected the fixed-IP screenshot; the operator performed the actions and VM checks.

- `LAB-004-network-before-isolation.png`: initial hostname-based baseline.
- `LAB-004-network-during-isolation.png`: initial hostname-based isolated result.
- `LAB-004-fixed-ip-validation.png`: fixed destination 172.66.147.243:443, False followed by True. The second-cycle baseline True is reported in chat but not visible in this screenshot.

No separate after-release screenshot, full EDR action audit, isolation-status screenshot, or timestamped second-cycle baseline was located. The domain-cycle recovery and isolation-state transitions are supported by operator reports rather than retained files. No event-body response JSON was supplied for isolation.

[SHA256SUMS.txt](SHA256SUMS.txt) identifies the retained image bytes. The public addresses are the test destination, not a claim about the operator's public address. Hashes provide integrity comparisons, not independent proof of action or authority.
