---
title: Query Tool Result File Generation

---

## How to generate query tool result files

Follow the steps below:

- Bring up the stack using `Docker compose pull && docker compose up -d`
    - Make sure to bring the stack up in non-aggregate mode with authentication disabled
- Send an empty query to `Local graph node 1` node
- Select the BIDs Synthetic dataset
- Download both human-readable labels and URIs files
- Make sure to manually add aggregate rows to both files
