# Transaction Dashboard Examples

Examples showing how to surface transaction state and submission progress in
frontend applications built on `sorokit-core`.

---

## Components

### `react-dashboard.example.tsx` — Status Dashboard

A React component that subscribes to a `TransactionStatusAggregator` and renders:

- Success rate and average fee metrics
- Status distribution chart data
- Recent transaction list with category labels

```tsx
import {
  TransactionStatusDashboard,
  startLiveUpdates,
} from "./react-dashboard.example";

// Poll your transaction source and push updates to all subscribers.
const stop = startLiveUpdates(async () => {
  // Return recently fetched TransactionResult[]
  return [];
});
```

---

### `transaction-progress.example.tsx` / `.vue` — Submission Progress Tracker (#133)

Maps the `simulate → sign → submit → confirm` lifecycle into a step-by-step
progress UI. When a fee-bump transaction is used, an extra **Apply Fee Bump**
step appears between Submit and Confirm.

#### React

```tsx
import {
  TransactionProgressTracker,
  TransactionProgressDemo,
  buildStepList,
  useTransactionProgress,
} from "./transaction-progress.example";

// Option A: use the hook and pass props manually
function MyProgress({ txId, queue }) {
  const { steps, currentStep, error } = useTransactionProgress(txId, queue);
  return (
    <TransactionProgressTracker
      steps={steps}
      currentStep={currentStep}
      error={error}
    />
  );
}

// Option B: use the all-in-one demo wrapper
function MyProgressDemo({ txId, queue }) {
  return <TransactionProgressDemo transactionId={txId} queue={queue} />;
}
```

#### Vue

```vue
<script setup lang="ts">
import TransactionProgress from "./transaction-progress.example.vue";
</script>

<template>
  <TransactionProgress
    :transaction-id="txId"
    :queue="txQueue"
    :includes-fee-bump="true"
  />
</template>
```

#### Steps rendered

| Step | Trigger |
|------|---------|
| Simulate | Before the transaction is signed |
| Sign | Waiting for wallet signature |
| Submit | Sent to Horizon, awaiting receipt |
| Apply Fee Bump | Present when `includesFeeBump` is true |
| Confirm | Transaction included in a ledger |

#### Failure display

When a step fails, the component:

- Marks that step with a `✗` icon
- Shows the decoded error message inline on the step
- Renders a summary `role="alert"` paragraph below the list

#### Testing with mockClient

```ts
import { createMockClient } from "../../src/testing/mockClient";
// See src/tests/transactionProgressComponent.test.ts for full examples
const client = createMockClient();
```

---

## Relevant source files

| File | Purpose |
|------|---------|
| `src/transaction/status.ts` | `getTransactionStatus` — lifecycle state polling |
| `src/transaction/transactionQueue.ts` | `TransactionQueue`, `QueueItemStatus` |
| `src/transaction/submitTransaction.ts` | Actual submission, error outcomes |
| `src/transaction/feeBumpTransaction.ts` | Fee-bump construction |
| `src/soroban/rpcErrorMapper.ts` | Error decoding for Soroban failures |
| `src/testing/mockClient.ts` | `createMockClient` for unit tests |
