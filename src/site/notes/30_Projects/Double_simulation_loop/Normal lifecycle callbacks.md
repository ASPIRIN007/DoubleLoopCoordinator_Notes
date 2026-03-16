---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/normal-lifecycle-callbacks/","created":"2026-03-14T17:59:06.565-04:00","updated":"2026-03-16T13:42:45.739-04:00"}
---

Examples:
- `register_initialization_callback`
- `register_before_ruc_solve_callback`
- `register_after_ruc_generation_callback`
- `register_before_operations_solve_callback`
- `register_after_operations_callback`
- `register_finalization_callback`

These get stored directly in callback lists inside `PluginCallbackManager`.

That behavior comes from [`internal.py`](file:///Users/vardhans/Library/Python/3.11/lib/python/site-packages/prescient/plugins/internal.py#L34), where Prescient creates lists like:

- `_before_ruc_solve_callbacks`
- `_after_ruc_generation_callbacks`
- `_before_operations_solve_callbacks`
- etc.

So for these hooks, yes, different hook types are registered in different lists.

For example:

```python
context.register_before_ruc_solve_callback(self.bid_into_DAM)
```

ultimately appends your function into a list for `before_ruc_solve` callbacks.