# Condition-based waiting patterns

> Когда нужно дождаться состояния — никогда `setTimeout`. Patterns для разных контекстов.

---

## Pattern 1 — Wait for state in test

```typescript
async function waitFor<T>(
  condition: () => T | Promise<T>,
  opts: { timeout?: number; interval?: number; message?: string } = {}
): Promise<T> {
  const timeout = opts.timeout ?? 5000;
  const interval = opts.interval ?? 50;
  const start = Date.now();
  let lastError: unknown;
  
  while (Date.now() - start < timeout) {
    try {
      const result = await condition();
      if (result) return result;
    } catch (err) {
      lastError = err;
    }
    await new Promise(resolve => setTimeout(resolve, interval));
  }
  
  throw new Error(
    `waitFor timed out after ${timeout}ms${opts.message ? ': ' + opts.message : ''}\n` +
    `Last error: ${lastError instanceof Error ? lastError.message : lastError}`
  );
}

// Use:
await waitFor(() => expect(state.completed).toBe(true), {
  timeout: 5000,
  message: 'Expected state.completed = true',
});
```

---

## Pattern 2 — Wait for DB row

```typescript
async function waitForRow<T>(
  query: () => Promise<T | undefined>,
  opts: { timeout?: number; interval?: number } = {}
): Promise<T> {
  return waitFor(async () => {
    const row = await query();
    if (!row) throw new Error('Row not found yet');
    return row;
  }, { timeout: opts.timeout ?? 5000, interval: opts.interval ?? 100 });
}

// Use:
const payment = await waitForRow(() =>
  db.select().from(payments).where(eq(payments.id, paymentId)).limit(1).then(r => r[0])
);
```

---

## Pattern 3 — Wait for HTTP response status

```typescript
async function waitForStatus(
  url: string,
  expectedStatus: number,
  opts: { timeout?: number; interval?: number } = {}
): Promise<void> {
  await waitFor(async () => {
    const res = await fetch(url);
    if (res.status !== expectedStatus) {
      throw new Error(`Got ${res.status}, expected ${expectedStatus}`);
    }
  }, { timeout: opts.timeout ?? 30000, interval: 500 });
}

// Use в integration tests где сервер стартует:
await waitForStatus('http://localhost:3000/api/health', 200);
```

---

## Pattern 4 — Wait for browser element (Playwright/E2E)

```typescript
// Playwright auto-waits для элементов уже:
await page.click('button[data-testid="submit"]');
await page.waitForSelector('[data-testid="success-message"]');

// НЕ:
// await page.click(...);
// await page.waitForTimeout(2000);  // ❌
```

Playwright встроенно — condition-based. Никогда `waitForTimeout` без явной причины.

---

## Pattern 5 — Polling job/queue completion

```typescript
async function waitForJob(
  jobId: string,
  opts: { timeout?: number; interval?: number } = {}
): Promise<JobResult> {
  return waitFor(async () => {
    const job = await getJobStatus(jobId);
    if (job.status === 'pending' || job.status === 'running') {
      throw new Error(`Job still ${job.status}`);
    }
    if (job.status === 'failed') {
      throw new Error(`Job failed: ${job.error}`);
    }
    return job;  // 'completed'
  }, { timeout: opts.timeout ?? 60_000, interval: 1000 });
}
```

---

## Pattern 6 — Event-based (преферрно)

Если можно — лучше events чем polling:

```typescript
// Вместо polling:
async function waitForJobEvent(jobId: string): Promise<JobResult> {
  return new Promise((resolve, reject) => {
    const timer = setTimeout(() => reject(new Error('Timeout')), 60_000);
    
    eventBus.once(`job:${jobId}:complete`, (result) => {
      clearTimeout(timer);
      resolve(result);
    });
    
    eventBus.once(`job:${jobId}:failed`, (error) => {
      clearTimeout(timer);
      reject(error);
    });
  });
}
```

Event-based — реагирует мгновенно когда state меняется. Polling — sample-based, имеет latency = interval.

Используй events когда система их поддерживает (WebSocket, SSE, EventEmitter, Redis pub/sub).

---

## Pattern 7 — Combine timeout + polling + abort

```typescript
async function waitForWithAbort<T>(
  condition: () => Promise<T>,
  opts: { timeout: number; signal?: AbortSignal }
): Promise<T> {
  const start = Date.now();
  while (Date.now() - start < opts.timeout) {
    if (opts.signal?.aborted) {
      throw new DOMException('Aborted', 'AbortError');
    }
    try {
      return await condition();
    } catch {
      await new Promise(resolve => setTimeout(resolve, 100));
    }
  }
  throw new Error(`Timeout after ${opts.timeout}ms`);
}
```

Поддерживает AbortController для cancellation — критично в long-running operations.

---

## Anti-patterns

### ❌ Arbitrary setTimeout

```typescript
await someAction();
await new Promise(r => setTimeout(r, 2000));  // why 2000?
expect(state).toBe(...);
```

Проблема: 2000 ms — guess. Slower env → fails. Faster env → wasted time.

### ❌ Recursive polling без timeout

```typescript
async function check() {
  if (await ready()) return;
  return check();  // infinite loop возможен
}
```

Без timeout = potential infinite hang.

### ❌ Polling с tight loop

```typescript
while (!await ready()) {
  // no delay → CPU hot loop
}
```

Это перегревает CPU. Always include `await sleep(N)` в polling.

### ❌ Race без cleanup

```typescript
const timer = setTimeout(reject, 5000);
const result = await someOp();
resolve(result);
// timer не cleared если success — память leaks (мелкий, но style-issue)
```

Always clear timers.

---

## Choosing pattern

| Контекст | Pattern |
|---|---|
| Test для async state change | Pattern 1 (waitFor) |
| Test для DB-зависимого | Pattern 2 (waitForRow) |
| Wait for server startup в integration test | Pattern 3 (waitForStatus) |
| E2E с UI elements | Pattern 4 (Playwright auto-wait) |
| Background job completion | Pattern 5 (polling) или 6 (event) |
| Production code event listener | Pattern 6 (event-based) |
| Cancellable wait | Pattern 7 (with AbortController) |

---

## Финальное правило

**Wait for condition, not for time.**

Время — переменная зависит от среды. Условие — invariant: «когда ready() == true, продолжай». 

Если кажется что нужно «подождать 2 секунды» — найди condition которое это «2 секунды» означает. Wait for that condition.
