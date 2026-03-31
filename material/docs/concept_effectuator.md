---
description: "godon Effectuator concept — applies configurations to target systems via SSH, HTTP, and Kubernetes API channels. Idempotent operations, timing, and rollback support."
---

## Effectuator

An **effectuator** applies configurations to target systems — it's the bridge between abstract parameters and real-world changes. Effectuators take a parameter set and make it happen.

### Role in the Optimization Loop

```
┌─────────────────────────────────────────────────────────────┐
│                    Optimization Loop                         │
│                                                              │
│    Algorithm ──▶ Effectuator ──▶ Target System              │
│    (suggest)      (apply)         (changed)                 │
│                                                              │
│                                         │                    │
│                                         ▼                    │
│                                   Reconnaissance            │
│                                     (observe)                │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

The effectuator is responsible for making the algorithm's suggestions real.

---

### Effectuator Interface

All effectuators implement the same contract:

```
┌─────────────────────────────────────────────────────────────┐
│                     Effectuator                              │
│                                                              │
│  Input:  { param_a: 0.5, param_b: "foo", ... }             │
│                                                              │
│  Action: Apply configuration to target                      │
│                                                              │
│  Output: Success | Error                                     │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

| Phase | Responsibility |
|-------|----------------|
| **Validate** | Check parameters are applicable |
| **Apply** | Make changes to target system |
| **Verify** | Confirm changes took effect |
| **Report** | Return success or error details |

---

### Built-in Effectuators

| Effectuator | Target | Protocol |
|-------------|--------|----------|
| **SSH** | Remote servers, VMs | SSH exec |
| **HTTP** | APIs, services | REST/HTTP |
| **Kubernetes** | Pods, deployments, configmaps | K8s API |

#### SSH Effectuator

```yaml
effectuator:
  type: ssh
  host: 10.0.0.50
  user: admin
  ssh_key: /secrets/ssh_key
  
  apply:
    command: |
      sed -i 's/pool_size=.*/pool_size={{ pool_size }}/' /etc/app/config.ini
      systemctl restart app
```

Use for: Traditional servers, VMs, bare metal

#### HTTP Effectuator

```yaml
effectuator:
  type: http
  endpoint: https://api.example.com/config
  method: PATCH
  headers:
    Authorization: Bearer {{ api_token }}
  
  payload:
    pool_size: "{{ pool_size }}"
    timeout_ms: "{{ timeout_ms }}"
```

Use for: Services with config APIs, service mesh, control planes

#### Kubernetes Effectuator

```yaml
effectuator:
  type: kubernetes
  namespace: production
  
  target:
    kind: Deployment
    name: app-backend
    
  patches:
    - path: /spec/template/spec/containers/0/resources/requests/memory
      value: "{{ memory_mb }}Mi"
    - path: /spec/replicas
      value: "{{ replicas }}"
```

Use for: Container orchestration, cloud-native workloads

---

### Idempotency

Effectuators should be **idempotent** — applying the same configuration twice has the same result as applying it once.

```
┌─────────────────────────────────────────────────────────────┐
│                    Idempotent Apply                          │
│                                                              │
│  Apply {pool_size: 10} ──▶ Success                          │
│  Apply {pool_size: 10} ──▶ Success (no-op)                  │
│  Apply {pool_size: 10} ──▶ Success (no-op)                  │
│                                                              │
│  Same result regardless of how many times applied           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Why it matters:**
- Retries don't cause corruption
- Rollbacks are predictable
- Parallel workers don't conflict

---

### Effectuation Timing

```
┌─────────────────────────────────────────────────────────────────┐
│                    Effectuation Timeline                        │
│                                                                  │
│  Start ──▶ Connect ──▶ Apply ──▶ Propagate ──▶ Steady ──▶ End  │
│    │          │          │           │           │              │
│    0s        1s         3s         10s         30s            │
│                                                                  │
│  Propagation delay: changes take time to reach full effect     │
└─────────────────────────────────────────────────────────────────┘
```

| Phase | What happens |
|-------|--------------|
| **Connect** | Establish connection to target |
| **Apply** | Execute configuration change |
| **Propagate** | Change spreads through system |
| **Steady** | System reaches new equilibrium |

**Critical:** Reconnaissance should wait until steady state, or metrics will be misleading.

---

### Rollback Support

When guardrails fail, effectuators may need to revert:

```yaml
effectuator:
  type: kubernetes
  
  apply:
    # ... apply config ...
    
  rollback:
    strategy: previous
    # or explicit rollback action
    command: kubectl rollout undo deployment/app
```

| Rollback Strategy | What it does |
|-------------------|--------------|
| `previous` | Restore last known-good config |
| `baseline` | Restore original/default config |
| `explicit` | Run custom rollback command |

---

### Error Handling

Effectuation can fail for many reasons:

| Error Type | Cause | Recovery |
|------------|-------|----------|
| **Connection** | Network, auth | Retry with backoff |
| **Validation** | Invalid params | Mark trial failed |
| **Permission** | Insufficient rights | Alert, require manual fix |
| **Timeout** | Slow target | Increase timeout or fail |

```yaml
effectuator:
  retry:
    max_attempts: 3
    backoff: exponential
    base_delay: 1s
    
  timeout: 60s
  on_failure: mark_trial_failed
```

---

### Safety Considerations

Effectuators modify real systems. Safety matters:

| Practice | Why |
|----------|-----|
| **Dry-run mode** | Validate changes without applying |
| **Gradual rollout** | Apply to subset before full deploy |
| **Circuit breakers** | Stop if error rate spikes |
| **Audit logging** | Track what changed, when, by whom |

```yaml
effectuator:
  safety:
    dry_run: false        # Set true for testing
    max_concurrent: 1     # Only one change at a time
    require_confirmation: false  # For production
```

---

### Custom Effectuators

Implement your own for specialized targets:

```
┌─────────────────────────────────────────────────────────────┐
│                   Custom Effectuator                         │
│                                                              │
│  Implement:                                                  │
│    - validate(params) → bool                                │
│    - apply(params) → result                                 │
│    - rollback(params) → result                              │
│                                                              │
│  Register with breeder via plugin interface                  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

Use cases:
- Proprietary systems
- Database config changes
- Hardware tuning
- Feature flag services

---

### Summary

| Aspect | What it means |
|--------|---------------|
| **Role** | Apply parameters to target systems |
| **Types** | SSH, HTTP, Kubernetes, custom |
| **Contract** | Validate → Apply → Verify → Report |
| **Idempotency** | Same result on repeated applies |
| **Timing** | Account for propagation delay |
| **Safety** | Dry-run, gradual rollout, circuit breakers |

---

### See Also

- [Breeder](concept_breeder.md) — Orchestrates effectuators
- [Reconnaissance](concept_reconnaissance.md) — Observes what effectuators change
- [Guardrails](concept_guardrails.md) — May trigger rollback
