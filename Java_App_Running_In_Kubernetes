When a **Java application runs inside Kubernetes**, debugging changes in **how you access the JVM**, not **what you debug**. The JVM internals (threads, heap, GC) are the same, but **containers, pods, and limits** introduce new failure modes.

Below is the **production-grade, Kubernetes-specific debugging model** used by SREs and senior backend engineers.

---

## 1. Correct Mental Model (Very Important)

**Kubernetes does NOT understand Java.**
Kubernetes only sees:

* CPU usage
* Memory usage
* Process exit codes
* Health probes

Java failures often look like **K8s failures**, but the root cause is still **JVM-level**.

---

## 2. First Checks (Before JVM Debugging)

### Pod-Level Signals

```bash
kubectl get pods
kubectl describe pod <pod>
```

Look for:

* `OOMKilled`
* `CrashLoopBackOff`
* Restarts count
* CPU throttling

### Logs

```bash
kubectl logs <pod>
kubectl logs <pod> --previous
```

**Rule:**
If pod restarts → **check K8s events first**, JVM later.

---

## 3. Thread Dumps in Kubernetes (Most Used)

### Option 1: Exec into Pod (Preferred)

```bash
kubectl exec -it <pod> -- /bin/sh
jstack <PID>
```

Find PID:

```bash
jps
```

### Option 2: Signal-Based Dump (No Tools Needed)

```bash
kubectl exec <pod> -- kill -3 1
```

Thread dump appears in logs:

```bash
kubectl logs <pod>
```

### Why `kill -3` is gold

* Works even in minimal images
* Zero JVM overhead
* Safe in production

---

## 4. Kubernetes-Specific Thread Problems

### 1️⃣ Thread Pool Exhaustion

Common cause:

* Blocking I/O inside request threads
* Too few threads vs traffic

Thread dump shows:

* Many threads `WAITING` on same call
* Request latency spikes

---

### 2️⃣ CPU Throttling (Very Common)

Kubernetes enforces CPU **limits**, JVM does not know it is throttled.

Symptoms:

* CPU < 100% inside container
* App feels slow
* No GC or code issue visible

Check:

```bash
kubectl describe pod <pod>
```

Look for:

```
CPU throttling
```

**Fix**

* Increase CPU limit
* Set:

```text
-XX:ActiveProcessorCount
```

---

## 5. Memory & OOM in Kubernetes

### Kubernetes OOM vs Java OOM

| Type               | Cause                     |
| ------------------ | ------------------------- |
| `OOMKilled`        | Container exceeded memory |
| `OutOfMemoryError` | JVM heap exhausted        |

### Why This Happens

* Heap + Metaspace + Thread stacks > container limit
* JVM default heap too large for container

### Mandatory JVM Flags in K8s

```text
-XX:+UseContainerSupport
-XX:MaxRAMPercentage=70
```

---

## 6. Heap Dumps in Kubernetes

### On OOM

```text
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/tmp
```

Then:

```bash
kubectl cp <pod>:/tmp/heap.hprof .
```

### Warning

Heap dumps are **large**. Ensure:

* Enough disk
* Pod does not die before dump completes

---

## 7. Java Flight Recorder (Best in K8s)

### Start JFR

```bash
kubectl exec <pod> -- jcmd 1 JFR.start
```

### Stop & Export

```bash
kubectl exec <pod> -- jcmd 1 JFR.stop filename=/tmp/app.jfr
kubectl cp <pod>:/tmp/app.jfr .
```

**Why JFR works well in K8s**

* Very low overhead
* No agent needed
* Safe in prod

---

## 8. Debugging Garbage Collection in Pods

Enable logs:

```text
-Xlog:gc*
```

Typical K8s GC issues:

* Frequent Full GC → memory pressure
* Long pauses → container memory too tight

Correlate:

* GC logs
* Pod memory metrics
* Restart timing

---

## 9. Observability is Mandatory in Kubernetes

Thread dumps show **one pod**
Kubernetes problems are **distributed**

You must have:

* Metrics (CPU, memory, restarts)
* Traces (request flow)
* Logs (structured)

Without tracing:

> You will never know *which pod* or *which request* caused the issue.

---

## 10. Debugging Playbook (Real Production Flow)

### Pod Restarting

1. `kubectl describe pod`
2. Check `OOMKilled`
3. Check heap vs limits
4. Adjust JVM memory flags

### App Slow but Pod Healthy

1. Check CPU throttling
2. Take thread dumps
3. Capture JFR
4. Check blocking calls

### Random Latency Spikes

1. Traces across pods
2. Identify slow pod
3. Thread dump that pod
4. Fix contention

---

## 11. Golden Rules for Java in Kubernetes

1. **Always size JVM for container**
2. **Always enable GC logs**
3. **Always use kill -3 for threads**
4. **Never rely on local debugging**
5. **Observability is not optional**

---

## 12. What We Can Do Next (Deep Dives)

Choose one:

* Kubernetes **OOMKilled vs Java OOM** (deep)
* CPU throttling **with real metrics**
* Thread dump **from a real pod example**
* JFR **flame graph interpretation**
* Debugging **Spring Boot + K8s**

Tell me which one.
