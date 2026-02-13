# Mental Model: "Spec → Plan → Execute"

Here's a reusable 3-step framework for planning coding agent tasks from specs:

## **Step 1: Structure First (Foundation)**

Ask your agent to:

```markdown
"Read this spec and create a dependency-ordered implementation plan.
Start with foundational layers (database → core services → domain services → API).
Group related work into phases. Show what must be built before what."
```

## **Step 2: Slice Vertically (Features)**

Ask your agent to:

```markdown
"For each major feature/workflow, create a vertical slice showing:

- Database models needed
- Service methods required
- API endpoints/resolvers
- Business rules to enforce
- Tests to write
  Identify which slices can be parallelized."
```

## **Step 3: Prioritize by Risk (Validation)**

Ask your agent to:

```markdown
"Rank the slices by:

1. Critical path items (blocks other work)
2. High-risk/complex items (needs early validation)
3. Quick wins (build momentum)
4. Nice-to-haves (defer if needed)
   Then create sprint-sized task batches."
```

---

## **Quick Prompt Template**

```markdown
I have this backend spec. Help me plan the implementation:

1. STRUCTURE: Create dependency-ordered phases (database → services → API → tests)
2. SLICE: Break each feature into vertical slices (models → business logic → endpoints → tests)
3. PRIORITIZE: Rank by critical path, complexity, and quick wins

For each slice, specify:

- Files to create/modify
- Dependencies on other slices
- Estimated complexity (S/M/L)
- Integration test scenarios

Output as a markdown task board I can track.
```

This gives you a **structured, feature-focused, risk-aware plan** you can execute incrementally. Works for any spec of this type.
