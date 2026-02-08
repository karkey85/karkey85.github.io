# RDMA unleashed

RDMA is high performance networking technology used in data centers for low latency and high throughput.


## RDMA stack overview from control path vs data path perspective:

🔧 Control path (slow path)

Used for:

Device discovery

PD / QP / CQ creation

MR registration

QP state transitions (RESET→INIT→RTR→RTS)

Destroying resources

Flow:

Application
  ↓
libibverbs / rdma-core
  ↓
User-space provider (mlx5, rxe, etc.)
  ↓
Kernel RDMA core
  ↓
NIC firmware / hardware
  ↓
Wire (RoCE / IB / iWARP)





🚀 Data path (fast path)
Used for:

ibv_post_send()

ibv_post_recv()

Doorbells

CQ polling

DMA

Flow:

Application
  |
  v
libibverbs
  |
  v
Provider (userspace)
  |
  v
NIC (direct MMIO / DMA)
