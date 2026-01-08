---
title: "Arena Allocators and OLTP: Every Microsecond Matters"
date: 2026-01-07T20:50:00-05:00
draft: false
team: ["avi"]
---

In an OLTP database processing 1 million transactions per second, every microsecond matters. Each transaction allocates dozens of temporary objects: query ASTs, execution plans, intermediate results, lock metadata. Traditional allocators we have seen in C like `malloc` spend precious cycles walking free lists and coalescing blocks. But here's the insight: transaction lifetimes are perfectly scoped - once a transaction commits or aborts, all its allocations become garbage simultaneously. This is exactly the workload pattern arena allocators were designed to dominate.

In my efforts to understand how allocators work in Zig and how it helps programmers create a better mental model for implementations of memory management strategies, I've decided to write allocators from scratch. This is the first blog in a series of implementing custom allocators, understanding this in view of OLTP systems.

## Why Arena Allocators for OLTP?

OLTP systems handle many short, frequent transactions: database inserts, queries, updates, etc. Each transaction might need to allocate and free memory for query parsing, network buffering, caching, and temporary results. If latency is slow on these fronts, it can affect businesses exponentially.

```zig
const ArenaAllocator = struct {
    buffer: []u8,        // Pre-allocated to avoid syscalls
    offset: usize,       // Fast: just an integer add per allocation
    used: usize,
    parent_allocator: std.mem.Allocator,
    
    // offset only moves forward until reset
    // This makes allocation O(1) with no branching
};
```

The allocation path is:
- **Branch-free** on the "happy path"
- **No locks needed** (regarding the allocator's internal state)
- **No syscalls**
- **Predictable latency**

## The Bump-Pointer

Arenas use **bump-pointer allocation**: simply incrementing the pointer forward by the size of the requested data. Instead of freeing each data point individually, arenas free all data within its state all at once via a `reset` or `destroy` at the end of a transaction.

```zig
// Determine the alignment of T then subtract 1 to determine the mask
const alignment_mask = @alignOf(T) - 1;
// This determines the starting point bump-pointer for the data
const alignment_offset = (self.offset + alignment_mask) & ~@as(usize, alignment_mask);
// This determines the placement of the bump-pointer after the allocation
const allocation_end = (alignment_offset + size_in_bytes);

// Bound checking
if (allocation_end > self.buffer.len) return error.OutOfMemory;
// Reassign the offset to the end of the allocation
self.offset = allocation_end;
self.used += size_in_bytes;

// u8_ptr points to the start of the first byte
const u8_ptr = &self.buffer[alignment_offset];
```

Transactions follow a lifecycle: `Begin -> Allocate -> Commit/Abort -> Cleanup`. This creates a pattern of **bulk allocation -> bulk deallocation**.

## The Malloc Bottleneck in OLTP

General-purpose allocators (`malloc`/`free`) are for heterogeneous, long-lived allocations. In high-concurrency OLTP, threads processing transactions simultaneously may lead to **lock contention** in `malloc`, causing spikes in latency.

An arena implementation makes cleanup trivial:

```zig
pub fn deinit(self: *Self) void {
    // Single call to free the entire large block
    self.parent_allocator.free(self.buffer);
}

pub fn reset(self: *Self) !void {
    self.offset = 0;
    self.used = 0;
}
```

## Cache Locality: The Real Secret

Arena allocators ensure **Perfect Cache Locality**. By laying out objects sequentially in memory:
`[Trans. State][Lock A][Lock B][Audit Log][Result] → Contiguous Block`

When the CPU requests the transaction state, it likely loads the entire block into the L1 cache (typically 64 bytes). Subsequent objects (Lock A, Lock B) are already resident in cache, allowing access with near-zero latency.

In contrast, `malloc` often scatters memory:
`[Lock A]...Gap...[Audit Log]...Gap...[Lock B]...Gap...[Trans. State]`

This forced the CPU to fetch a new cache line for virtually every step, leading to constant cache misses (~30-50ns), which is catastrophic for P99 latency.

## Closing Thoughts

Choosing not to use a bump-pointer arena in OLTP is often "engineering process laziness"—sacrificing predictable, low latency for the sake of a general-purpose tool. Zig's O(1) arena allocator's speed and guaranteed cache locality are specialized precision tools for mission-critical services.

By using Zig to implement this custom memory strategy, you gain:
1. **Elimination of Lock Contention**: Thread-local arenas bypass global locks.
2. **Deterministic Performance**: Bulk deallocation is just a pointer reset.

Thank you for reading my first blog piece! Shout-out to Tiger Beetle 🪲 :3.
