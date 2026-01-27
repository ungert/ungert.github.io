---
layout: project
type: project
image: img/plumeriafarm.jpeg
title: "Molokai Plumerias"
date: 2025 - Present
published: true
labels:
  - Shopify
  - Java
summary: "In the process of a project implementing a logic system that will authenticate orders for Plumaria Farms."
---



Molokai Plumerias is a farm that provides a large amount of plumerias throughout the United States. I am in the process of writing code for a logic system that will place, store, and authenticate oders. The unique challenge of this program is accounting for a rotating and depreciating inventory, while also constrained by the order's originating location.
Here is some code that illustrates how we are verifiying orders:

```cpp
package shopifyApplication.service.revalidation;

import shopifyApplication.inventory.DailyInventory;
import shopifyApplication.inventory.Snapshot;
import shopifyApplication.model.Order;
import shopifyApplication.repo.OrderRepository;
import shopifyApplication.repo.SnapshotRepository;
import shopifyApplication.service.InventoryService;
import shopifyApplication.service.OrderVerification;

import java.time.LocalDate;
import java.util.List;
import java.util.Objects;

/**
 * Replays a date window using current projections & rules to see whether
 * previously accepted orders still fit. simulate(...) is read-only; commit(...)
 * should persist any required changes (snapshots / order statuses).
 *
 * NOTE: This is a scaffold that compiles and runs. You’ll fill in the actual
 * replay logic later (earliest-placed-first per day, respect default usage,
 * flag PARTIAL/FAIL, etc.).
 */
public class RevalidationService {

    private final OrderRepository orderRepo;
    private final SnapshotRepository snapshotRepo;
    private final InventoryService inventoryService;

    // Optional: use verification helpers (e.g., fit defaults) if you expose them
    private final OrderVerification verifier; // may be null

    public RevalidationService(OrderRepository orderRepo,
                               SnapshotRepository snapshotRepo,
                               InventoryService inventoryService,
                               OrderVerification verifier /* nullable */) {
        this.orderRepo = Objects.requireNonNull(orderRepo, "orderRepo");
        this.snapshotRepo = Objects.requireNonNull(snapshotRepo, "snapshotRepo");
        this.inventoryService = Objects.requireNonNull(inventoryService, "inventoryService");
        this.verifier = verifier;
    }

    /** Read-only pass that produces a structured report for [start..end] inclusive. */
    public RevalidationResult simulate(LocalDate start, LocalDate end) {
        if (start == null || end == null || end.isBefore(start)) {
            throw new IllegalArgumentException("Invalid start/end");
        }

        RevalidationResult result = new RevalidationResult();

        // Walk day-by-day; for each day, derive start-of-day inventory and dry-run orders
        LocalDate d = start;
        DailyInventory invStart = inventoryService.getInventoryFor(d, snapshotRepo);
        while (!d.isAfter(end)) {
            RevalidationDay day = new RevalidationDay(d);

            // Pull all previously confirmed orders with pick date == d
            List<Order> dayOrders = orderRepo.getAcceptedOrdersForPickDate(d);

            // --- Dry run: fit per default usage (no lookback rebalancing here) ---
            DailyInventory work = invStart.copy();
            for (Order o : dayOrders) {
                // For now treat all as OK (placeholder).
                // Later: call a shared "fit default" method and set ADJUSTED/PARTIAL/FAIL based on result.
                day.addOutcome(RevalidationOrderOutcome.fromOrder(o, OutcomeStatus.OK, "simulated-ok"));
                // TODO: actually consume from 'work' using your default allocation to carry into rotate
            }

            result.addDay(day);

            // Rotate to next day's start-of-day inventory
            LocalDate next = d.plusDays(1);
            invStart = inventoryService.rotate(work, next);
            d = next;
        }
        return result;
    }

    /**
     * Persist the changes that your simulate() decided (e.g., downgrade status, mark partial, or
     * store updated snapshots). In this scaffold, we only persist snapshots so you can see a flow.
     */
    public void commit(RevalidationResult result) {
        if (result == null) return;

        for (RevalidationDay day : result.getDays()) {
            LocalDate date = day.getDate();
            if (date == null) continue;

            // In a full implementation, you'd:
            // 1) Recompute the final start-of-day inventory for 'date' (or carry over from simulate()).
            // 2) Save a Snapshot(date, inv, frozen=false/true depending on policy).
            // 3) Update orders based on outcomes (OK/ADJUSTED/PARTIAL/FAIL) in OrderRepository.

            // Here we at least ensure a snapshot exists so downstream code has a checkpoint.
            DailyInventory inv = inventoryService.getInventoryFor(date, snapshotRepo);
            snapshotRepo.save(new Snapshot(date, inv, /*frozen*/ false));
        }
    }
}
```

