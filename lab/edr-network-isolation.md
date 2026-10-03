# LAB-004: Controlled Network Isolation

Perform this exercise on an owned lab VM with direct hypervisor-console access and an online EDR sensor. Use a controlled, available destination and a fixed IP/port for comparison. This case used `172.66.147.243:443`; its availability should not be assumed for future runs.

1. Record the VM hostname, permitted scope, baseline UTC time, and successful TCP connection to the fixed destination. Preserve output.
2. In the LimaCharlie sensor Overview, confirm the intended endpoint and invoke Isolate from Network. Record the resulting dashboard state and any available audit metadata.
3. Repeat the exact fixed-IP TCP test. Preserve its result and timestamp. Validate sensor connectivity separately if it is part of the test objective.
4. Release isolation on the same endpoint. Preserve the status and action record.
5. Repeat the exact connection test and confirm recovery. Investigate failed recovery before closing the exercise.
6. Document which connections were checked and which were not. A single failed probe does not establish all-traffic blocking.

Use [SOC-L1-004](../cases/SOC-L1-004-edr-network-isolation.md) as the observed case record. There is no LAB-004 detection rule or automated response configured by this exercise.

Vendor command reference: [LimaCharlie endpoint commands](https://docs.limacharlie.io/8-reference/endpoint-commands/).
