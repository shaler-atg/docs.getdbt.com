---
title: "The devil is in the docs"
description: "The devil in the details and the details are the docs that we write."
slug: the-devil-in-the-docs
authors: [mirna_wong]
tags: [ai, docs]
hide_table_of_contents: false
date: 2026-04-30
is_featured: true
---

> *"By all means, move at a glacier's pace."*
> — Miranda Priestly, *The Devil Wears Prada*

There's another scene in *The Devil Wears Prada* that I think about more than that one. If you haven't seen the it yet, I'll try not to spoil it for you. Miranda (the character modeled after Anna Wintour) turns to Andy (her assistant) and explains, with _complete_ patience, that the [cerulean blue](https://en.wikipedia.org/wiki/Cerulean) in Andy's "lumpy sweater" didn't come from nowhere &mdash; it's traced back through fashion decisions made years earlier, by people who thought carefully about every choice. The whole chain, invisible to Andy, was built on deliberate _curation_.

That iconic scene reminds me of what documentation architecture feels like to me. Users (could be developers, analysts, data engineers, etc.) ask a question, get an answer, keep working. They don't need to see the decisions behind it &mdash; what to include, how to structure it, where the gaps are, what needs updating. But those decisions shape every answer and experience they get, especially the ones coming from AI.

This blog discusses how we as a docs team are trying to bring docs closer to users; and why that architecture matters more than ever in the AI era.

<!-- truncate -->

## The problem: docs exist, but AI can't reach them

Think of Andy in *The Devil Wears Prada* &mdash; constantly fielding requests for information she could theoretically find, but scrambling each time because it isn't at her fingertips. She's not uninformed. She's just not connected to the right source when it counts.

That was the [dbt MCP server's](https://docs.getdbt.com/docs/dbt-ai/about-mcp) relationship with the docs:

- The open source documentation at [docs.getdbt.com](https://docs.getdbt.com) is carefully maintained (jointly by the docs team _and_ our amazing dbt community), up-to-date, and formatted for humans, of course. And now machine consumption: there's an `llms.txt` index, a full-content flat file, and markdown output on every page. We have an `AGENTS.md` file that lists how to access the docs via web requests. The source was solid and accessible.
- The dbt MCP server &mdash; where dbt users interact with dbt through AI tools &mdash; couldn't reach any of it by default. It has eight toolset categories: CLI, Semantic Layer, Discovery, Admin API, SQL, Codegen, Fusion, Server Metadata. But none of them connected to the live docs by default. It didn't have our docs as its canonical source.

So when an agent was asked *"how do I configure incremental models?"*, it improvised or used training data, pattern matching, best guess. The docs kinda existed. The path to them didn't.

<Lightbox src="/img/blog/2026-04-30-devil-in-the-docs/dbt-mcp-server-toolsets.png" title="Miranda mad at AI for hallucinating about incremental models" />

## The research: what the data actually showed

Earlier this year we'd been discussing docs data and how AI tools are now fetching them to answer questions. The term 'canonical docs' was mentioned and I struggled to understand what it meant and how it was different from the docs we were already building. But then [Google announced their new docs API and MCP server](https://developers.googleblog.com/introducing-the-developer-knowledge-api-and-mcp-server/) &mdash; then it clicked! Docs are now canonical and more important than ever in the AI era 💃💃💃!

So we chatted about this and toyed with the idea of either:
- building a new docs MCP server, which meant starting from scratch and building all the infrastructure from the ground up
- adding docs tools to the existing dbt MCP server, which meant we would need to add the docs toolset category to the existing dbt MCP server (which already has users and set up)
- or leaving it alone since we have the docs in [dbt agent skills](https://github.com/dbt-labs/dbt-agent-skills) (shout out to the amazing work from the DX team!), which is way forward for users who install the dbt agent skills.

But that still left us with the question of how to get the docs closer to users in a way that is easy to use and maintain.

We ran some sql queries using the [dbt VS Code extension](https://docs.getdbt.com/docs/install-dbt-extension?version=2.0) and also used [Insights](https://docs.getdbt.com/docs/explore/dbt-insights?version=2.0) for exploratory analysis in our internal dbt platform account:
- We saw that the dbt MCP server had great adoption already and had a ton of the infrastructure in place
- Adding the docs tools would be a natural extension of that and would be a great way to get the docs closer to users 
- The dbt MCP server had eight toolset categories already but none of them could directly access docs.getdbt.com
- When an agent needed to answer *"how do I configure incremental models?"* &mdash; it relied on training data and best guesses

As mentioned above, we already had a solid foundation for AI-readable docs:

- `docs.getdbt.com/llms.txt` — a page index for AI systems
- `docs.getdbt.com/llms-full.txt` — full docs content in a single file
- `.md` suffix support on any docs page for clean markdown output
- `AGENTS.md` file that lists how to access the docs via web requests
- A published [`fetching-dbt-docs` skill](https://skills.sh/dbt-labs/dbt-agent-skills/fetching-dbt-docs) that teaches AI agents how to access our docs via web requests, installed across Claude Code, Copilot, Gemini CLI, Amp, and others

The skill approach works great! But it relied on the agent knowing the skill existed, installing it, and performing web fetches as a workaround. We were curious to see what would help bring docs closer to users.

## The solution: delivering The Book

In *The Devil Wears Prada*, there's a ritual: every evening, "The Book" &mdash; a very secret binder containing the mock up layout of the upcoming Runway issue, featuring photo proofs, article layouts, and ad placements &mdash; is delivered to Miranda's apartment for her to review. She doesn't go to the office to get it. It arrives, in her space, when she needs it.

That's the standard we were trying to meet. Developers shouldn't have to leave their workflow to find docs. The docs should arrive &mdash; in the tool they're already using, at the moment they need them.

The dbt MCP server could already handle lineage lookups, test runs, Semantic Layer queries, job debugging. But ask it *"what's the syntax for a snapshot strategy?"* and it might get it right or it might get it wrong or it might improvise. Training data. Best guess. The Book wasn't being "delivered".

✨ **The solution:** We added two tools to the dbt MCP server under a new **"Product Docs"** category — the ninth toolset✨:

- **[`search_product_docs`](https://docs.getdbt.com/docs/dbt-ai/mcp-available-tools?version=2.0#product-docs)** &mdash; searches docs.getdbt.com and returns titles, URLs, and relevance-ranked descriptions for pages matching your query.
- **[`get_product_doc_pages`](https://docs.getdbt.com/docs/dbt-ai/mcp-available-tools?version=2.0#product-docs)** &mdash; fetches the full markdown content of one or more docs pages by path or URL.

The workflow mirrors how a human would use the docs: search first to find what's relevant, then fetch the full content. The difference is that it happens inside whatever AI tool you're already using, without a context switching headache. The dbt MCP server is open source and free, which also meant every user would get docs access automatically.

Alongside this, the DX team's [dbt agent skills](https://skills.sh/dbt-labs/dbt-agent-skills) &mdash; including `fetching-dbt-docs` &mdash; remain the best path for agents not connected via MCP. The two complement each other: the skill proved the demand; the MCP tools are the native solution.

## What this changes day to day
- Before: write a macro, hit an unfamiliar function, alt-tab to a browser, search docs, read the page, return to your editor, try to remember where you were. Or worse, ask AI and get a response that is not grounded in actual current documentation.(hallucinate much?)
- After: ask Claude or your AI tool directly, get an answer grounded in actual current documentation, keep coding!

We also added the product docs toolset to dbt's [Developer agent](https://docs.getdbt.com/docs/dbt-ai/developer-agent?version=2.0) experience &mdash; bringing the docs closer to users in dbt platform and the Studio IDE. 🎉

The Book arrived. No context switch required.

For analysts exploring a shared project, it means understanding what a model does without navigating to a separate tab. For teams working across different dbt setups, it means consistent, authoritative answers regardless of where they're working or who's asking. For the docs team, it means the work we put into writing and maintaining docs.getdbt.com is doing more than it was before &mdash; reaching users where they actually are.

## What's next

This is the first page of any in this new chapter. But there are a few things on the radar for the next pages as we continue to try to improve the docs experience:

- **Version-aware docs fetching** &mdash; Right now these tools return current docs. A developer on dbt Core 1.10 asking about incremental strategies gets 2026 docs. Version-aware routing &mdash; returning the right page for the right dbt version &mdash; is the next meaningful improvement, and we're [working](https://github.com/dbt-labs/dbt-mcp/pull/638) through it now!
- **Smarter search ranking** &mdash; Relevance is good. Relevance tuned to dbt-specific concepts and query patterns would be better.
- **Coverage gaps as a signal** &mdash; MCP usage will surface which questions return weak results &mdash; pages that are missing, thin, or outdated. That's a direct queue for the docs team, and a feedback loop that didn't exist before.
- **A dedicated dbt docs MCP server** maybe? &mdash; It's not the right call right now &mdash; one coherent product with an active existing server is the right place to start &mdash; but in the future, maybe a standalone docs server could reach every MCP client regardless of whether they use the dbt MCP server at all?

## Give it a try!

- If you're new to the MCP server, the [quickstart guide](https://docs.getdbt.com/docs/dbt-ai/mcp-quickstart-oauth?version=2.0) walks through both local and remote configuration.

- If you're already connected to the dbt MCP server, the product docs tools are available now. Check out the [MCP available tools reference](https://docs.getdbt.com/docs/dbt-ai/mcp-available-tools?version=2.0#product-docs) for details.

We'd love to hear your feedback! Open an issue at [github.com/dbt-labs/docs.getdbt.com](https://github.com/dbt-labs/docs.getdbt.com) to flag any bugs, typos, wrong info, etc. &mdash; or find me in the [dbt community Slack](https://www.getdbt.com/community/join-the-community). 

## Conclusion

Miranda's point about the cerulean sweater wasn't really about fashion. It was about invisible chains &mdash; every deliberate decision ripples further than the person making it can see, and that the quality of the source shapes everything downstream.

That's where docs are now. A developer asks an AI a question and gets an answer without possibly ever opening a browser. They don't need to see the decisions behind it &mdash; the careful decisions, curation, structure, accuracy, the gaps that got filled. But those decisions are in the chain. They always were. The difference is that now, they travel faster and reach further than ever before.

We're no longer moving at a glacial pace. The current runs _fast_. And the devil? Well, it's in the docs.

*Mirna Wong is a technical writer at dbt Labs who loves em-dashes, pop culture references, and writing about dbt and AI. dbt's product docs are open source &mdash; contributions and issues are **always** welcome at [github.com/dbt-labs/docs.getdbt.com](https://github.com/dbt-labs/docs.getdbt.com).*
