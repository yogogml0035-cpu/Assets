# Deep Interview Context Snapshot: prompt-demand-classification

- Task statement: User wants to classify similar prompt or demand content from `D:\Assets\历史长需求记录\历史长需求记录.md` and `D:\Assets\常用提示词\提示词.md` into corresponding type-specific Markdown files.
- Desired outcome: Not yet fully specified. Likely an organized Markdown knowledge base where prompts/requirements are grouped by category.
- Stated solution: Split/classify content into corresponding category `.md` files.
- Probable intent hypothesis: Reduce clutter, make reusable prompts/long requirements easier to find, reuse, improve, and maintain.
- Known facts/evidence: Workspace root is `/mnt/d/Assets`; target files exist at `/mnt/d/Assets/历史长需求记录/历史长需求记录.md` and `/mnt/d/Assets/常用提示词/提示词.md`; combined size is about 47KB and 199 lines, so initial context is prompt-safe and no summary gate is needed.
- Constraints: Must not execute classification directly during deep-interview; must clarify intent, boundaries, non-goals, decision authority, and acceptance criteria first.
- Unknowns/open questions: Category taxonomy, granularity, whether original files should be preserved or rewritten, naming scheme, duplicate handling, metadata/frontmatter, sorting strategy, and how to treat mixed-content entries.
- Decision-boundary unknowns: Whether Codex may invent categories, merge/split near-duplicates, normalize wording, create directories, move content, preserve original ordering, or edit source files.
- Likely touchpoints: `/mnt/d/Assets/历史长需求记录/`, `/mnt/d/Assets/常用提示词/`, possible new category files/directories under `/mnt/d/Assets`.
- Prompt-safe initial-context summary status: not_needed.
