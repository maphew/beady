# Datastar as a Beady dashboard framework

**Date:** 2026-07-30  
**Status:** Future consideration; no implementation commitment

## Conclusion

Datastar appears to be a strong technical fit for **Beady**, but it is not by itself a reason to create another generic Beads dashboard or to rewrite Beady wholesale.

Beady's distinctive proposition is already close to Datastar's strengths: a Fossil-like local web interface delivered as one Go executable, with the server rendering HTML and Beads remaining the authoritative work graph. The useful question is therefore not “should there be another Beads UI?” but:

> Can Datastar make Beady a substantially better live operator window onto agents changing a Beads work graph, while preserving the single-binary, local-first design and reducing frontend machinery?

That question can be answered with a small vertical experiment.

## Why Datastar fits

Datastar combines HTMX-like backend interaction with Alpine-like frontend reactivity. It uses declarative `data-*` attributes, server-sent events (SSE) for backend-driven DOM and signal updates, and provides an official Go SDK. Its browser bundle can be self-hosted and embedded in the executable; it does not require an npm-based frontend build.

| Beady or Beads concern | Datastar fit |
|---|---|
| One executable | Embed `datastar.js`, templates, and CSS in the Go binary |
| No Node toolchain | No npm or frontend build is required |
| Local-first operation | The UI and server can run entirely on the user's machine |
| Beads changes beneath the viewer | SSE can push updated issue fragments immediately |
| Filtering and inline editing | Signals provide lightweight browser-side state |
| Beads remains authoritative | Mutations continue to go through `bd` or a stable Beads application API |
| Server-rendered simplicity | Go renders HTML; the browser need not maintain a duplicate domain model |

Datastar is most compelling for a **live operational view**, rather than ordinary CRUD. Examples include:

- ready work appearing when blockers close;
- claims and statuses changing as agents work;
- blocked and stale work becoming visible;
- dependency or molecule progress advancing;
- multiple browser views updating from one authoritative state change.

Beady's existing HTMX approach is already adequate for conventional forms, filtering, and partial-page navigation. A migration is only justified if server-driven live behaviour becomes materially simpler or more capable.

## The important architectural boundary

The frontend framework is not the main risk. The essential boundary is:

```text
Datastar UI → stable Beads application or CLI interface → Dolt
```

It must not become:

```text
Datastar UI → knowledge of Beads storage tables → Dolt
```

Several Beads UIs were coupled to SQLite, JSONL, or a particular schema and suffered as Beads storage changed. Beady already uses `bd` for write operations; that principle should apply to all behaviour that requires Beads validation or semantics. A UI should not recreate issue lifecycle rules or become a second implementation of the work graph.

Datastar cannot compensate for an unstable application boundary. A prototype may nevertheless help define the boundary Beads UIs need:

- stable query and view models;
- mutation endpoints or CLI contracts;
- change notifications or event streams;
- error and conflict semantics;
- explicit transport separation from Dolt storage.

This interface work could prove more valuable to the wider Beads ecosystem than the dashboard itself.

## Relationship to adjacent technologies

These tools occupy different layers and should not be combined merely because they are all interesting.

| Technology | Role | Relationship to this experiment |
|---|---|---|
| Datastar | Presentation and live interaction | Candidate replacement or complement for Beady's HTMX layer |
| Beads and Dolt | Authoritative versioned work graph | Remain the system of record |
| Automerge | CRDT-based local-first synchronization | Exclude; it would overlap with Dolt and introduce a second consistency model |
| Agentsview | Local indexing, search, and analysis of agent sessions | Potential future information source, not a Beady dependency |
| Entire | Commit-linked agent provenance and resumable checkpoints | Potential future information source, not a Beady dependency |

A future human operator console could present work state from Beads, session retrieval from Agentsview, and commit provenance from Entire. That is a much larger product. It should not be smuggled into a Datastar evaluation.

## Bounded experiment

Do not begin with a rewrite or a new repository. Use an experimental branch and implement one end-to-end slice:

1. Render `bd ready`.
2. Open one issue without full-page navigation.
3. Change its status through `bd` or an equivalent stable Beads interface.
4. Detect an external Beads change and push the resulting UI update through SSE.
5. Embed Datastar and all other browser assets into the executable.

Compare the result directly with Beady's existing HTMX implementation.

### Continue only if

- the application remains a single executable;
- it works offline without CDN dependencies;
- there is no Node-based build requirement;
- the implementation does not read or mutate Beads storage directly;
- handwritten browser JavaScript is absent or genuinely exceptional;
- live updates are clearly simpler and more reliable than the current approach;
- total state-management and transport code decreases or delivers substantial new capability.

### Stop if

- most of the work concerns replacing already adequate CRUD behaviour;
- Datastar introduces a parallel client-side domain model;
- the experiment expands into Automerge, Agentsview, Entire, or a general agent cockpit;
- direct Dolt or schema coupling is needed to make progress;
- the branch becomes a broad rewrite before demonstrating the single vertical slice.

## Product positioning

The Beads ecosystem already contains numerous list, dashboard, Kanban, desktop, and editor interfaces. Matching their feature lists would make Beady another participant in a crowded category.

A more distinctive thesis is:

> Beady is a zero-ceremony, local, single-binary human window into the live Beads work graph.

Datastar is worth evaluating insofar as it strengthens that thesis. It is not worth adopting simply because it is new or impressive.

Agentic development makes small software mutations cheap. It does not make coherent semantics, compatibility, distribution, or maintenance cheap. The right response is to permit experiments that are narrow enough to be selected—or killed—quickly.

## References

- [Datastar getting started](https://data-star.dev/guide/getting_started)
- [Datastar backend SDKs](https://data-star.dev/reference/sdks)
- [Datastar repository](https://github.com/starfederation/datastar)
- [Beady repository and current design](https://github.com/maphew/beady)
- [Beads UI ecosystem discussion](https://github.com/gastownhall/beads/discussions/276)
- [Beads dashboard discussion](https://github.com/gastownhall/beads/discussions/978)
- [Automerge overview](https://automerge.org/docs/hello/)
- [Agentsview](https://github.com/kenn-io/agentsview)
- [Entire checkpoint overview](https://docs.entire.io/guides/checkpoints/overview)
