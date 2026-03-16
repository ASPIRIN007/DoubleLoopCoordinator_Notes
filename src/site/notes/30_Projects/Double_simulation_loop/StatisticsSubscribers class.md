---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/statistics-subscribers-class/"}
---

`_StatisticsSubscribers` is just a small container that groups together the pending stats subscriber lists.

It has four fields:

- `operations`
- `hourly`
- `daily`
- `overall`

Each field is a list of callback functions.

So its job is to hold the stats-related subscribers temporarily before they are moved into `stats_manager`.

That is why `PluginCallbackManager.__init__()` sets:

```python
self._pending_stats_subscribers = _StatisticsSubscribers()
```

Meaning:
- create one object
- inside it keep four buckets
- each bucket stores callbacks of one stats type

So conceptually it is like:

```python
pending_stats_subscribers = {
    "operations": [...],
    "hourly": [...],
    "daily": [...],
    "overall": [...],
}
```

except Prescient chose to package that as a `NamedTuple` instead of a dict.

Why this class exists:
- to keep the pending stats subscribers organized
- to separate stats subscriptions from the normal lifecycle callback lists
- to make it easy to later move each category into `stats_manager`

So it is not doing any computation by itself.
It is mainly a structured holder for four lists of subscriber callbacks.
