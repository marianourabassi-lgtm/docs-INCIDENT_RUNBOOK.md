# Incident RCA & Runbook: Mitigating 502 Bad Gateway under 150k RPM

## Incident Context
During peak traffic, Nginx upstream socket pool was exhausted resulting in widespread `502 Bad Gateway` errors.

## Root Cause Analysis (RCA)
1. **PHP-FPM Memory Leak:** Unindexed metadata queries caused PHP workers to hold connections for >4s, spiking memory usage to system limits.
2. **Nginx Backlog Overflow:** `somaxconn` system kernel limit was reached, causing TCP drops.

## Resolution Steps Executed
1. **Database Triage:** Applied compound index `idx_meta_key_value` on database tables, dropping query latency from 4.2s to 12ms.
2. **Kernel Tuning:** Increased Linux TCP backlog limits (`sysctl -w net.core.somaxconn=1024`).
3. **Automated Shielding:** Deployed `EdgeShield-PHP-Health-Mesh` to dynamically recycle memory-leaking workers.

## Outcome
Restored full system stability, latency stabilized below 140ms, 0% drop rate.
