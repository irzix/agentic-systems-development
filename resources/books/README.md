# Books

Selected references for understanding and building agentic systems. Read the sections relevant to your project; these are companions to the course.

## Main references

| Book | Focus | Suggested reading | Access |
| --- | --- | --- | --- |
| **AI Engineering** - Chip Huyen | Engineering LLM applications: evaluation, RAG, agents, inference and architecture | Chapters 3-4, 6, 9-10 | [Publisher](https://www.oreilly.com/library/view/ai-engineering/9781098166298/) - paid |
| **Algorithms for Decision Making** - Kochenderfer, Wheeler & Wray | Mathematical foundations: sequential decisions, online planning, belief states and multi-agent decisions | Chapters 6-9, 19-23; 24-27 for multi-agent systems | [PDF](pdfs/algorithms-for-decision-making-2022.pdf) / [Official site](https://algorithmsbook.com/decisionmaking/) - free |
| **Multiagent Systems: Algorithmic, Game-Theoretic, and Logical Foundations** - Shoham & Leyton-Brown | Coordination, distributed optimization, communication and incentives | Chapters 1-2, 7-8, 13-14 | [Official download page](https://www.masfoundations.org/download.html) - free |
| **Build a Large Language Model (From Scratch)** - Sebastian Raschka | Implementing attention, a GPT-style model, pretraining and fine-tuning with PyTorch | Work through the implementation if model internals are unfamiliar | [Author's site and code](https://www.sebastianraschka.com/llms-from-scratch/) - book paid, code free |

Start with **AI Engineering** for system design. Use **Algorithms for Decision Making** for deeper planning and uncertainty; it assumes probability, linear algebra and calculus. Add **Multiagent Systems** for coordination theory. Raschka covers the model inside an agent.

## Further reading

| Book | Use it for | Access |
| --- | --- | --- |
| **Artificial Intelligence: Foundations of Computational Agents**, 3rd ed. - Poole & Mackworth | Agent architectures and foundational reasoning; chapters 1-3, 6, 9, 12-14 | [Complete HTML](https://artint.info/3e/html/ArtInt3e.html) - free |
| **Planning Algorithms** - Steven M. LaValle | Discrete planning (chapter 2) and decisions under uncertainty (chapters 9-12) | [Official PDFs](https://lavalle.pl/planning/) - free |
| **Multi-Agent Reinforcement Learning: Foundations and Modern Approaches** - Albrecht, Christianos & Sch?fer | Learning policies through interaction; useful when the project involves MARL | [PDF](pdfs/multi-agent-reinforcement-learning-2024.pdf) / [Official site](https://marl-book.com/) - free |
| **Speech and Language Processing**, 3rd ed. draft - Jurafsky & Martin | Language models, transformers, retrieval and RAG | [Official manuscript](https://web.stanford.edu/~jurafsky/slp3/) - free, unfinished |
| **AI Agents: The Definitive Guide** - Nicole Koenigstein | Agent execution, sandboxing, evaluation, memory and operations; chapters 5-12 | [Publisher](https://www.oreilly.com/library/view/ai-agents-the/0642572247775/) - paid |

The classical planning and multi-agent books develop formal models; applying them to LLM agents requires adaptation. Koenigstein is a provisional recommendation based on the contents and available preview. Recommendations reflect topic coverage, not a complete technical review of every book.

## PDFs and licenses

The two included PDFs are unchanged official downloads under CC BY-NC-ND, for noncommercial use. Attribution, restrictions and checksums are in [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md). Other books link to official sources; Shoham and Leyton-Brown explicitly request no PDF redistribution.
