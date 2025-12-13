| Probe Type | Purpose | Action on Failure |
| :--- | :--- | :--- |
| **1. Liveness Probe** | Detects if the application is **alive** and running. | The container is **restarted**. |
| **2. Readiness Probe** | Detects if the application is **ready to serve traffic**. | The Pod is **removed from Service endpoints** (no traffic is routed to it). |
| **3. Startup Probe** | Prevents premature restarts for **slow-booting** applications. | The container is **restarted** (only after the initial failure period/threshold). |
