# TODO — `docs/Decision Tree.mmd`

Scope: `docs/Decision Tree.mmd`, `docs/Decision Tree.md`, `README.md`, and the five empty guide stubs in `docs/`.
Line numbers refer to `docs/Decision Tree.mmd` at `8e8c989` unless noted.

Severity: **P0** = tree is wrong/broken for readers · **P1** = a major branch is unreachable or misleading · **P2** = clarity/polish.

> **Baseline check before and after edits.** Two audits disagreed on whether the diagram parses. Spot checks against a Mermaid 11 renderer show that a dangling `-->` followed by a node on the next line **does** parse and binds (so lines 16–17, 48–49, 50–51 etc. are fragile style, not parse errors), while an edge ending in `--->` with no target is an error in isolation and may silently bind to the next line inside this file. Do not trust this note — render the file with `mmdc -i "docs/Decision Tree.mmd" -o /tmp/tree.svg` after edits and confirm 0 errors.

---

## 1. P0 — Things that make the chart wrong or unrenderable

- [ ] **T1. Fix the malformed `use` menu (lines 79–83).** `use -.- gemini`, `use --- other --->`, `use --- cloud` reference IDs that do not exist; each creates a _ghost node_ labelled `gemini` / `other` / `cloud`, and the real nodes `use.gemini` (86), `use.other` (87), `use.cloud` (88) are orphaned. The trailing `--->` on 82 also has no target and can swallow the next line.
      Replace with one consistent style: `use --> use.openai | use.anthropic | use.gemini | use.other | use.cloud`.
- [x] **T2. Fix the `hight` typo (lines 67–70) and the missing edge at 66.** `host.usage.hight.csp.selfhost` is a different node from `host.usage.high` (66), so the ">10 people / constant usage" branch dead-ends on "You will need quite a bit of infrastructure" and its `yes`/`no` answers never fire. Rename to `host.usage.high.csp.selfhost` and connect `host.usage.high --> …`.
- [x] **T3. Define or remove `host.usage.small` (line 72).** It is referenced but never declared, so the "not sure, very small jurisdiction" answer lands on a blank box with the raw ID as its label.
    - [x] Connect the href link
- [x] **T4. Give `host` (line 32) real edges or delete it.** It has zero inbound and zero outbound edges, so "You should run LLMs yourself" floats as an orphan and is never the conclusion of any path.
- [ ] **T5. Give `host.cloud` (line 42) real children.** No edge connects it to `host.cloud.ollama` (43), `host.cloud.vllm` (44), or `host.cloud.k8s` (45). `host.cloud.ollama` (43) is completely unreachable.
- [ ] **T6. Terminate `host.cloud.k8s` (line 45) at the guide.** Unlike 43/44 it has no edge to `k8s` (173); add `--> k8s`, or delete the redundant node (see T15).
- [ ] **T7. Give the Cloud Service Provider subgraph an entry and an exit (99–130).** `csp.it-access` (107) has no inbound edge, so the whole interview is decorative. The only cross-links are `-.-` at 53/62/68 into `csp.selfhost` — the one node that already means "you self-host". Route the three "Do you have your own servers? → no" answers into `csp.it-access`, and route `csp.azure/gcp/aws/selfhost/other` out to the matching `use.cloud.*` / `host.machine` node.
- [ ] **T8. Stop `csp.unknown --> csp.aws` (128–129).** The label says the provider is unclear and the recommendation may be inaccurate, then recommends AWS. Make it terminal with a next step, or send it back to `csp.it-access` / `csp.other`.
- [ ] **T9. Remove duplicate node definitions of the same question (lines 52, 61, 67).** Three copies of `"Do you have your own servers? TODO: make single name for this"`, each with its own downstream routing. Collapse to one `csp.selfhost-q` decision and branch on tier.
- [ ] **T10. Strip every user-visible `TODO`.** Lines 52, 61, 67 in question labels; lines 153 and 159 inside comment nodes. This is authoring residue rendered to readers.
- [ ] **T11. Resolve the `host.cloud.vllm` link contradiction (line 44).** Its text is cloud-specific but its edge lands on `vllm.server`, whose href is the _server_ guide; `host.cloud.ollama` (43) does the same with the cloud guide. Decide: separate cloud terminals, or retitle to "run it on a cloud server" and link the server guide.
- [ ] **T12. Decide whether links inside labels are viable.** Mermaid's default `securityLevel: strict` (GitHub, VS Code, default `mmdc`) escapes HTML labels, so the 14 `<a href>` tags can render as visible markup. If the diagram will be read on GitHub, move navigation into `docs/Decision Tree.md` as markdown links and keep labels plain.

## 2. P1 — Decision logic that misleads

- [ ] **T13. Make the security gate universal.** `security.data-type` (134) is only entered from the automation branches (18, 25, 29). The chatbot path (15 → 27 → 28 → `host.local`) never asks what data is involved, so the chat use case is the least screened. Put the data-sensitivity question immediately after the root question, before any hosting recommendation.
- [ ] **T14. Replace the liability-vs-harm question (139–142).** "Not being liable" and "minimizing harm" are not opposites; "not sure" and "minimizing harm" go to the identical destination, making one answer carry no information. Worse, "not being liable" sends HIPAA/PII data straight to a third-party provider with no BAA/DPA checkpoint. Rewrite as a capability question ("Can this data go to a third-party service?" / "Only with a signed contract" / "I don't know who approves that") and add a `security.ask` terminal that says to consult privacy/legal before starting.
- [ ] **T15. Collapse the Kubernetes chain (37–39).** Three consecutive nodes ("Host our custom deployment", "buy several GPU machines", "put them on one network") all link to the same guide and duplicate 34's "office network" point. Use one result node with a short bullet list, or move the steps into the guide.
- [ ] **T16. De-duplicate the Ollama/vLLM terminals (35/43, 36/44).** Two entry paths feed one guide node each, which is fine, but 43's guide conflicts with 44's destination (see T11) and the repeated hrefs are copy-paste hazards. Link the guide once on the terminal node.
- [ ] **T17. Rebuild the "not sure" joins (71–76).** `host.usage.medium` and `host.usage.high` are message nodes defined _inside_ the 10-person and 10+-person branches, but 73–76 reuse them as answers. Two readers in the same situation reach different nodes, and one of the three "not sure" answers dead-ends. Define `tier.low` / `tier.medium` / `tier.high` once, route both volume and "not sure" answers into them, and hang the sizing messages off as notes.
- [ ] **T18. Add an exit for "we can't host anything".** Nothing on the host side routes to `use` (78); `host` and `use` are never connected. Add "no IT capacity / no budget / won't run infrastructure" → `use`.
- [ ] **T19. Add a "both" answer to the root question (13–15).** The two options are mutually exclusive by construction, but most departments will want chat now and automation later; today they must guess which branch sizes their infrastructure.
- [ ] **T20. Add an Ollama-vs-vLLM decision or criteria.** Both are presented as equal siblings (35–41) with no question distinguishing them, yet lines 63–64 silently force vLLM for the medium tier. Add one criteria node (one/few users + simplicity → Ollama; many concurrent requests → vLLM; HA/multi-tenant/reproducible → Kubernetes).
- [ ] **T21. Make `host.machine`'s options consistent.** `---` links to Ollama/vLLM (40–41) but `-->` chains into Kubernetes (37), so Kubernetes reads as a follow-on step rather than an alternative. Use one glyph for all three.
- [ ] **T22. Add the missing hybrid/existing-vendor branch.** M365 Copilot, an existing Azure OpenAI agreement, an EHR-embedded model, or a state-wide contract are common public-health situations and currently have no path. `use.cloud.azure` hints at it but is orphaned.
- [ ] **T23. Review the "Public Data → use" shortcut (137).** Every data type reaches `use` in at most one hop, so the data question never actually excludes a managed provider. Confirm whether that is intended, and if not, make `security.trust` outcomes conditional on the data type.
- [ ] **T24. Add a BAA/PHI warning to the laptop path.** `host.local` → `ollama.laptop.download` (33) is reachable by a HIPAA reader with only "You are liable for your actions" as a caveat. No BAA, PHI, or "don't paste patient data" statement exists anywhere in the file.

## 3. P0/P1 — Links and guide stubs

- [ ] **T25. Fill or hide the five empty guides.** `docs/Running Ollama in the Cloud.md`, `Running Ollama on a Laptop.md`, `Running Ollama on a Server.md`, `Running the Kubernetes Stack.md`, `Running vLLM on a Server.md` are all **0 bytes**. Every one of the 14 `href`s lands on a blank page. Either add a short "in progress + what to read instead" stub, or strip the `<a>` wrappers until content exists.
- [ ] **T26. Add the missing guides the tree promises.** Cloud vLLM, cloud Kubernetes/template infrastructure, "identify your cloud provider", "other CSP instructions", per-provider pages (Azure/GCP/AWS/Oracle/DigitalOcean/Linode), OpenAI/Anthropic/Gemini setup and data-handling notes, Microsoft Foundry / Vertex AI / Bedrock, security & HIPAA guidance, and a local-vs-hosted trade-offs page. At minimum, make `use.*`, `csp.*`, and `security.*` terminals point somewhere.
- [ ] **T27. Fix the `ollama.com` link on line 161.** The download URL is plain text inside a node whose only anchor is the (empty) laptop guide, so the actual download link is not clickable and the visible instruction is circular ("see the guide" → same click). Split into a real download link plus the guide.
- [ ] **T28. Replace "Click here for more info" link text (35, 36, 43, 44).** In an SVG the node text _is_ the link text; "Click here" is ambiguous for screen readers and repeats per branch. Use descriptive text only.
- [ ] **T29. Choose one link convention.** All 14 URLs hard-code `https://github.com/TheResilientCollective/Guide-to-LLMs-in-Public-Health/blob/HEAD/docs/…`, which breaks on fork, rename, transfer, and in offline/SVG exports. Prefer relative `./Running%20…md` in the `.mmd` plus ordinary markdown links in the `.md`. If absolute URLs stay, use `blob/main` instead of `blob/HEAD`.
- [ ] **T30. Embed or link the diagram from `docs/Decision Tree.md`.** That file is 3 lines of prose and never mentions `Decision Tree.mmd`; GitHub does not preview `.mmd`, so a repo visitor cannot see the tree at all. Add a ` ```mermaid ` fence (or a committed `Decision Tree.svg`) plus a text list of the five outcomes.
- [ ] **T31. Fix `README.md` navigation.** Line 3: "The follow repository" → "This repository". Line 5: "take a loot at [[decision-tree]]" → a real link, e.g. `[decision tree](docs/Decision%20Tree.md)`. `[[decision-tree]]` is Obsidian-only, renders literally on GitHub, and does not match the filename `Decision Tree.md`.

## 4. P2 — Remove or collapse unnecessary nodes and edges

- [ ] **T32. Remove the `host` heading node (32).** Unreachable, unlabelled as a result, and duplicates the branch's meaning. (Same action as T4; listed here because deletion is the cheaper option.)
- [ ] **T33. Remove the three `-.-` "self-host" links (53, 62, 68).** They draw long edges from the sizing questions into a _different_ subgraph's answer node and add no information once T9/T7 are done.
- [ ] **T34. Delete the duplicate CSP taxonomy.** `use.cloud.azure/gcp/aws/other` (89–96) and `csp.azure/gcp/aws/oracle/digital-ocean/linode` (112–125) describe the same vendors at different granularity with no cross-links. Pick one list or link them.
- [ ] **T35. Remove or finish the three comment nodes at 147–160.** `note.liability` ("You are liable for your actions."), `note.security` (truncated, "TODO"), and `note.tooling` ("TODO") are unreachable, so they never appear next to the laptop recommendation they qualify. Either connect them to `ollama.laptop.download` and write them properly, or delete them.
- [ ] **T36. Delete the stale comment on line 14.** `%% Are you only using LLMs…?"}` has copy-paste residue and duplicate/missing words. Its intent is better than the shipped question (see T37) — fold it in, then remove it.
- [ ] **T37. Rename `is-local` (13) to something that matches the question** (e.g. `usage.mode`), and `workflow.is-local` (23) accordingly, so IDs are not confused.
- [ ] **T38. Remove the trailing space on line 45** and normalise the 8 double-blank-line gaps (after 9, 31, 98, 132, 145, 166, 171, 176).

## 5. P2 — Copy, questions, terminology

- [ ] **T39. Rewrite the root question and its answers (13–15).** The question asks "how", the stored answer is a 100-character description of a scene ("People talk to a chatbot, get information, ask questions about files, generate reports"), and four verbs are bundled into one option. Use short selectable labels, e.g. "Chat with a model (ask questions, read files, draft text)" / "Automate a process (data processing, scheduled jobs)" / "Both".
- [ ] **T40. Shorten the automation question (17).** 210 characters, with a ~150-character CDC example inside the decision diamond. Ask "Should these workflows run on a schedule, or should a person start them?" and move the example to a comment node.
- [ ] **T41. Fix the data-type question (134).** Missing `?`, two clauses with no connector, and "likely to be used by others" is undefined. Make it "What is the most sensitive data this will touch?" with answers Public / Internal / Personal or health (PII, HIPAA) / I'm not sure.
- [ ] **T42. Fix the experimentation question (56–58).** "I suspect we will use this long term" does not answer "Are you looking to experiment first?". Ask about expected duration and make the answers parallel.
- [ ] **T43. Make "powerful computers" measurable (27).** Replace "powerful"/"beefy" with a checkable threshold, e.g. "work laptops with at least 16 GB of RAM".
- [ ] **T44. Define jargon before use:** vLLM (36), CSP (110 — avoid the acronym on canvas), PII (135), VPN (102), Kubernetes (37 vs 173), EC2/S3 (120 — justify as a recognition probe), "jurisdiction" (71–75 — replace with team-size ranges), "designated computer" (34), "our custom deployment" vs "our template infrastructure" (37 vs 45 — pick one name).
- [ ] **T45. Fix typos and brand casing:** `hight` (67–70), "setup" → "set up" (163, 168, and `docs/Decision Tree.md:3`), "Sharepoint" → SharePoint (112), "Digital Ocean" → DigitalOcean (96, 122), "gmail" → Gmail (114), "self hosted"/"Self Hosting" → self-hosted/self-hosting (110, 118).
- [ ] **T46. Pick one voice convention and apply it.** Currently mixed: hedged advice ("You should…", "You will want to…"), bare imperatives ("Purchase…", "Use…"), fragments ("OpenAI"), and assertions about the reader's org ("You probably use Microsoft Azure"). Recommended: questions in second person ending in `?`; recommendations starting with a fixed stem ("Recommended: …"); notes in third person present tense.
- [ ] **T47. Distinguish node roles visually.** Results, questions, guide links, notes, and menu items are all rounded rectangles except the diamonds. Give terminals a stadium/flag shape, guide links a distinct shape, keep notes as comments, and add a legend.
- [ ] **T48. Add a title, start marker, and accessibility text.** Add a diagram title, rename the root to `START: …`, and add `accTitle:` / `accDescr:` after `flowchart TD` — Mermaid currently emits an SVG with no accessible name.
- [ ] **T49. Add the textual alternative and five-outcome list to `docs/Decision Tree.md`.** This doubles as the accessible version, the legend, and the "next steps" every terminal node currently lacks.
- [ ] **T50. Review the `redux-color` theme.** GitHub ignores it (and applies its own theme); nothing in the diagram explains whether colour encodes anything; dark-mode contrast is unverified. Either treat it as decoration and say so in the legend, or encode recommendation/option classes deliberately with `classDef`.
- [ ] **T51. Rename files without spaces when links are next touched:** `Decision Tree.mmd` → `decision-tree.mmd`, etc. Not worth a standalone commit, but removes all `%20` encoding.

## 6. Optional structural refactor (fixes several items at once)

- [ ] **T52. Replace the dotted-ID pseudo-hierarchy with real subgraphs.** `host`, `host.machine`, `host.cloud`, `use` are flat sibling IDs, not namespaces, so `host.machine.k8s` is not a child of `host.machine` and every branch renders as an island. Wrapping each cluster in a `subgraph` fixes T4, T5, T6, and T33 together and removes the "did the edge mean 'is a child of' or 'is an option'?" ambiguity.
- [ ] **T53. Replace dangling line-continuation edges with single-line statements.** Lines 16–17, 19–20, 22–23, 48–51, 59–61, 65–66, 71–76, 108–129 rely on a `-->` at end of line and the target on the next. It currently parses in Mermaid 11, but it is not the documented style and a formatter or older renderer will silently cut the tree in half.
- [ ] **T54. Add labelled notes instead of untyped `-.-`.** If dotted links are kept, label them (`-. "see note" .->`) so the semantics are visible.

---

## Open questions for the content owner

1. Is the intended flow **data sensitivity first** (safety gate), or **usage first** (current)? T13 and T23 depend on this.
2. Should the tree have any path that says "do not use LLMs for this yet", or is every branch an endorsement?
3. Do you want the five guides written now, or should the links be hidden until they exist? (T25)
4. Is the target reader GitHub, Obsidian, or an exported SVG/PDF? The link strategy in T12/T29/T30 depends on it.
5. Should "use a cloud provider" (Bedrock/Vertex/Foundry) be one outcome or three? T34 needs this before the CSP subgraph is wired.

---

## Suggested order of work

1. **Mechanical P0s:** T1, T2, T3, T10, T38 — restore a coherent graph; verify with `mmdc`.
2. **Topology P0s:** T4, T5, T6, T7, T8, T9, T33 — reachability of the host, cloud, CSP, and high-usage branches.
3. **Link & navigation P0s:** T25, T30, T31, T27 — stop readers landing on blank pages or a broken README.
4. **Logic/safety decisions:** T13, T14, T24, T17, T18, T19 (needs owner answers above).
5. **Content pass:** T39–T46, T20–T22.
6. **Polish & structure:** T47–T54.

## Verification checklist

- [ ] `mmdc -i "docs/Decision Tree.mmd" -o /tmp/tree.svg` exits 0.
- [ ] Every node ID is defined exactly once; no label is a raw ID.
- [ ] BFS from the root reaches every recommendation node (no orphans).
- [ ] Every terminal recommendation has either a guide link or an explicit next step.
- [ ] Every link target exists and is non-empty.
- [ ] `grep -n "TODO\|hight\|click here" "docs/Decision Tree.mmd"` returns nothing.
