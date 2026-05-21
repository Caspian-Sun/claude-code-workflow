# Upstream Fidelity

> **Core principle**: Upstream documents (product spec / design mockup / OpenAPI) are the **single source of truth**.
> Downstream artifacts (PRD / task list / code) **must strictly follow upstream intent — no free association, restructuring, or overriding**.
> When conflicts / ambiguity / gaps are found, you **must stop and present options for the user to decide** — AI is not allowed to make the call unilaterally.

---

## Why this rule exists

Lessons repeatedly show that PRD drafters (whether AI or human), when faced with upstream documents that are not fully explicit, tend to exhibit the following behaviors:

- **Overriding intent**: Changing upstream "jump to A" into "jump to B" because B "feels more reasonable"
- **Fabricating references**: Citing upstream sections that don't exist, to wrap the change in an authoritative shell
- **Faking reviews**: Writing "confirmed with product" / "review approved" without any review evidence
- **Cross-section refactoring**: Rewriting an upstream "internal sub-entry X" as a "main entry Y", changing the hierarchy
- **Treating industry practices as rules**: "MetaMask does this, so we do this" goes directly into the PRD as a rule

On the surface these look like "optimizations", but in essence they **silently lose product intent across the PRD → plan → code chain**. The end user ultimately receives a product that does not match the original product spec, and **because the PRD looks "complete + has references", the discrepancy is hard to detect**.

---

## Scope (3 categories of upstream documents)

| Upstream document | Downstream artifact | Applicable commands |
|------------------|---------------------|---------------------|
| **Product spec** (PM-written specs / Notion / Lark imported to `docs/prds/_imports/`) | PRD (`docs/prds/**/*.md`) | `/prd`, `/prd-check` |
| **Design mockup** (Figma / screenshots placed in `docs/designs/`) | PRD + tasks + code | `/prd`, `/plan`, `/code`, `/review` |
| **OpenAPI protocol** (`docs/apis/openapi/*.json`) | API md + Repository / DTO | `/code`, `/review` |

---

## Hard Rules

### R1. Upstream source must be traceable; references must be real

- When a downstream artifact references upstream, it **must give the source location** (file path + line number or anchor)
- The cited section / paragraph **must actually exist** — fabricated sections are not allowed
- Before citing, grep / check the actual file to confirm existence

❌ Bad example:
```
Tap wallet name → Switch Wallet page (Product Spec §"Wallet Switch Entry" requirement)
```
("§ Wallet Switch Entry" section does not exist in the original product spec)

✅ Good example:
```
Tap wallet name → Switch Wallet page (product spec wallet-spec.md:367)
```
(real line number cited, can be one-click navigated to original for verification)

### R2. No "free association / context refactoring"

Downstream is not allowed to:
- **Rewrite / rename** content from upstream section A and apply it to section B
- **Refactor an upstream "internal sub-entry X" into a "main entry Y"**
- Freely add fields / flows / navigations that upstream didn't mention

❌ Bad example: Product spec "**Personal Center > Settings >** Switch Wallet" gets refactored into "**Home top bar >** Switch Wallet Page" — turning an internal entry into a main entry.

### R3. "Confirmed with product" must have review evidence

Downstream artifacts **must not write the following without evidence**:
- "Confirmed with product"
- "Confirmed with design"
- "Product decided X"
- "Review approved"

Unless one of the following exists as evidence:
- The commit message contains a reviewer (e.g., `Reviewed-by: @pm-xx`)
- The PRD's "Change Log" table contains a row: date / reviewer / decision content
- The associated GitLab MR / GitHub PR contains product reviewer comment / approval

Without evidence, the wording should change to:
- "**PRD author proposes** X, pending product review"
- "**Conflicts with product spec**, pending user decision (see §Conflicts To Decide)"

### R4. Conflict between upstream and design mockup: must be explicitly flagged + stop and ask

If the upstream product spec ≠ design mockup (e.g., product spec says "jump to A" but design mockup implies "jump to B"), **AI is not allowed to pick a side on its own**. Must:
1. Add a `## Conflicts To Decide` section in the PRD, explicitly listing the conflict points
2. `/prd-check` sees this section and automatically blocks entry to `/plan`
3. Wait for the user to answer (add a row to the PRD change log)

### R5. "Industry practice" / "I think" / "this is better" is **not justification**

Downstream artifacts must not use reasons like "industry X does this", "this is more intuitive", "reduces steps" to override upstream intent. Such reasoning can appear as a **proposal** in a "Pending Review" section, but **cannot be a confirmed PRD rule**.

---

## Special handling when upstream conflicts with design mockup

Per `.claude/commands/prd.md` Step 1A "Design Mockup Element Inventory", when the inventory conflicts with the true intent of the design mockup:
- The element inventory ≠ true design intent — it's only AI's recognition of elements from the image
- If recognition conflicts with the product spec, trust the **original product spec text first**, add a "Design vs Product Spec Conflict Pending" section to the PRD
- Wait for user decision before proceeding

---

## Check List (run by `/prd-check` + `/plan-check`)

### A. Reference authenticity (P0)

- ☐ All "product spec §X" / "design mockup X" / "H5 X" references in the PRD — section / file **actually exists**
- ☐ References include line numbers or anchors — no vague "§ some section" wording

### B. Free-association interception (P0)

- ☐ grep "confirmed with product" / "confirmed with design" / "product decided" / "review approved" — when matched, **must** have corresponding review evidence (commit / change log / MR)
- ☐ grep "self-modified" / "refactored to" / "adjusted to" / "industry standard" / "reference X" — when matched, must be explicitly tagged "pending review"

### C. Conflicts made explicit (P1)

- ☐ If the PRD has a `## Conflicts To Decide` section, block `/plan`
- ☐ Compare PRD with the corresponding section of the product spec on key fields (navigation paths / APIs / field names); discrepancies must be explicitly flagged

---

## Violation handling

### When detected

| Stage | Who runs it | What's checked |
|-------|-------------|----------------|
| `/prd` writing PRD | AI self-check | All R1–R5 |
| `/prd-check` | Command logic | All A / B / C |
| `/plan` before task split | AI self-check | A (reference authenticity); refuse to split if it fails |
| `/review` | code-reviewer | All A / B / C; matches marked 🔴 Critical |

### Fix priority

R1–R3 violations: 🔴 Critical, must fix before continuing downstream
R4–R5 violations: 🟡 Warning, should fix but not blocking

---

## Self-prompt for AI

Before writing each downstream artifact (PRD / task / code), ask yourself:

> **"For this line of content, can the original text be found in the upstream document?"**
> - Yes → Provide a `file:line` reference
> - No → **Stop, mark as "pending review"** — do not free-associate
> - No but "industry standard does this" → Same as above, **stop, list both options, let the user choose** — do not decide unilaterally

**Every single sentence** of a downstream artifact should be reverse-traceable to some line in the upstream. **"Water without a source" is not allowed.**
