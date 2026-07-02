# m30-DEEPJUDGE — Formal Tacit-Aware Adjudication of Extracted Knowledge

**Full specification (`m30_DEEPJUDGE_spec.md`), POLANYI++ v24+.**
Vocabulary prefix: `judge:`. Task type: **T22-JUDGE**. Locus: **`aif:LensLocus`** (a reflexive observer wielding H/M/T/B/U as descriptions; verdicts point *downward* onto S's licensing of the extraction, and never rewrite S's content — firewall-respecting, the locus where m25 Mode B already lives).

---

## 1. Why a formal judge, and why tacit-aware

Informal LLM-as-a-judge prompting (Prometheus-style rubrics, single-pass "rate 1–5") has two defects for POLANYI++. First, it is **un-warranted**: a scalar score carries no auditable basis, no provenance, and no separation between *the item is wrong* and *the judge is guessing*. Second, and decisively, it is **stated-only**: a default judge checks whether an extracted item is literally supported by the surface text. POLANYI++ is an *implicit-knowledge* extractor — its heuristics (b4-PRESUPPOSITION, b5-IMPLICATURE, b6-IMPACT, b9-CAUSALITY, b10-ToM …) exist precisely to surface frames a competent reader infers but the speaker did not state. A stated-only judge therefore scores the method's **core capability as error**.

This was confirmed empirically during calibration (ISIT catcalling pilot, n=18). Judged stated-only, frame-level precision read 0.52 for the more generous backend; judged on a tacit criterion (is each frame a warranted *explicit or tacit* inference?), the same extraction scored 0.96, human-verified by two annotators at 99% agreement. The 0.44 gap was not error — it was warranted tacit content, the very yield the method exists to produce. DEEPJUDGE formalizes that corrected criterion so judging is reproducible, warranted, auditable, and reusable across the pipeline.

DEEPJUDGE is **not** a softer judge. It still rejects projection. The discipline is that *tacit warrant is admissible only when a named pragmatic mechanism ties the item to specific source content* — otherwise it is `Unwarranted` (a fabrication, in DARKSIDE terms). It replaces "plausible" with "licensed-by-mechanism-X."

---

## 2. The adjudication criterion (formal)

For each extracted item `x` (a frame, claim, commitment, alignment, or hypothesis) with source span(s) `σ(x) ⊆ S` and producing-heuristic provenance `h(x)`, DEEPJUDGE assigns a verdict on the **warrant lattice**:

```
judge:Verdict ∈ { Explicit, TacitlyWarranted, Weak, Unwarranted }
```

- **`judge:Explicit`** — `x` is directly attested in `σ(x)`. (≡ `dark:WarrantedCommitment`, surface mode.)
- **`judge:TacitlyWarranted`** — `x` is not stated, but a **licensing mechanism** `μ ∈ {PRESUPPOSITION, IMPLICATURE, IMPACT, ToM, CAUSALITY, …}` derives `x` from `σ(x)` in context. The verdict MUST cite `μ` via `judge:licensedBy` (pointing at the b-heuristic) and the content it rests on. This is the category DARKSIDE's stated-only warrant axis lacks; DEEPJUDGE adds it.
- **`judge:Weak`** — a tacit reading is *possible but strained*: no single mechanism cleanly licenses `x`, or the source underdetermines it. Flagged for review. (≡ `dark:UnattestedCommitment`.)
- **`judge:Unwarranted`** — neither explicitly stated nor licensed by any mechanism: projection. (≡ `dark:FabricatedCommitment` / `dark:MisattributedCommitment`; emitted into the DARKSIDE WarrantProfile.)

**Admissibility rule (anti-rubber-stamp):** `TacitlyWarranted` requires a *specific* `(μ, content)` pair. "It is plausible that the speaker felt fear" is rejected; "fear is licensed by ToM (b10): avoidance-under-harassment in σ implies a threat appraisal" is admitted. The judge names the mechanism that a sceptical human reader could check in one glance.

**Granularity neutrality:** each item is judged at *its own* level of generality (a coarser frame is not penalised for being coarse, a finer one not rewarded for splitting). Reconciling granularities is ALIGN's job (§7.3), not the judge's.

---

## 3. Vocabulary (`judge:`)

```turtle
judge:Verdict           a owl:Class .          # one per (item, judge-instance)
judge:Explicit          a owl:Class ; rdfs:subClassOf judge:Verdict .
judge:TacitlyWarranted  a owl:Class ; rdfs:subClassOf judge:Verdict .
judge:Weak              a owl:Class ; rdfs:subClassOf judge:Verdict .
judge:Unwarranted       a owl:Class ; rdfs:subClassOf judge:Verdict .

judge:judges            a owl:ObjectProperty .  # Verdict -> extracted item x
judge:licensedBy        a owl:ObjectProperty .  # Verdict -> b-heuristic (the mechanism μ); required for TacitlyWarranted
judge:restsOn           a owl:ObjectProperty .  # Verdict -> source span σ(x)
judge:byInstance        a owl:ObjectProperty .  # Verdict -> judge:JudgeInstance
judge:hasWarrant        a owl:ObjectProperty .  # Verdict -> dark:Warrant*  (REUSE of m25 warrant axis)
judge:verdictNote       a owl:DatatypeProperty. # one-sentence, human-checkable rationale
judge:verdictConfidence a owl:DatatypeProperty.

judge:JudgeInstance     a owl:Class .           # a model+config, or a human annotator
judge:instanceModel     a owl:DatatypeProperty. # e.g. "Claude-4.x" / "Gemini-3.x" / "human:annotatorA"
judge:instanceFamily    a owl:DatatypeProperty. # model family, e.g. "anthropic" / "google" / "human"
judge:disposition       a owl:DatatypeProperty. # "precision-oriented" | "recall-oriented" (for EQUILIBRIUM weighting)
judge:independenceOverride a owl:DatatypeProperty. # boolean; lifts the cross-family cap on TRUSTLESS (default false)
judge:overrideRationale a owl:DatatypeProperty.  # required string when independenceOverride = true (audit trail)

judge:PrecisionProfile  a owl:Class .
judge:explicitCriterionPrecision a owl:DatatypeProperty .  # |Explicit| / N
judge:tacitCriterionPrecision    a owl:DatatypeProperty .  # |Explicit ∪ TacitlyWarranted| / N
judge:tacitYield                 a owl:DatatypeProperty .  # |TacitlyWarranted| / N  (the beyond-stated content recovered)
judge:overProjectionRate         a owl:DatatypeProperty .  # |Unwarranted| / N  -> dark:fabricatedRate
judge:weakRate                   a owl:DatatypeProperty .  # |Weak| / N  -> dark:unattestedRate

judge:ConsensusReport   a owl:Class .
judge:interJudgeAgreement a owl:DatatypeProperty .        # Krippendorff α over judge instances (REUSE m24 Mode B stats)
judge:resolvedVerdict   a owl:ObjectProperty .            # equilibrium/disputatio outcome per item
judge:contestedItem     a owl:ObjectProperty .            # items escalated to DISPUTATIO

judge:VerificationSheet a owl:Class .                     # per-item row set for fast human confirmation
```

A verdict is also an **active-inference** event (v24 grounding): the source-licensed expectation is the prior, the extracted item is the observation, an `Unwarranted` verdict is high prediction error. Over-projection findings carry `polanyi:generativeFlow aif:flow-DARKSIDE-Backward`.

---

## 4. Modes

### Mode A — Item adjudication (single judge pass)
Given a completed XKG `E` (or a candidate set of items), for each item: locate `σ(x)`, read `h(x)`, assign a verdict, and for `TacitlyWarranted` cite `judge:licensedBy`. Emit `judge:Verdict` per item, a `judge:VerificationSheet`, and a `judge:PrecisionProfile`. BRI: BIND (item↔span), REIFY (verdict), INTEGRATE (profile aggregation).

### Mode B — Multi-judge consensus (ensemble + resolution)
Run **≥2 independent `judge:JudgeInstance`s** — ideally *different model families*, and optionally a **human** instance (the human pass is just another instance; its inclusion is how "a human can quickly verify" is operationalised). Then:
1. Compute `judge:interJudgeAgreement` (Krippendorff α; reuses m24 Mode B inter-rater machinery).
2. **Agreeing items** → `resolvedVerdict` = the agreed verdict.
3. **Disagreeing items** → invoke **m4-EQUILIBRIUM** (§5): balance the judges' dispositions (a precision-oriented vs a recall-oriented instance pull toward `Weak/Unwarranted` vs `TacitlyWarranted` respectively) to an equilibrium verdict.
4. **Contested items** (warranted-vs-projection split that EQUILIBRIUM cannot settle) → invoke **m7-DISPUTATIO** (§6) for in-utramque-partem adjudication.
5. **Independence check** (§7.2a): a TRUSTLESS-grade verdict is admissible only if the panel is cross-family (or includes a human instance); otherwise it is capped at SUPERVISED unless `judge:independenceOverride` is set.
Emit `judge:ConsensusReport`.

### Mode C — Live verdict gate (REIFY-gate guardrail)
Parallel to m25 Mode C and m21-CUECOHERENCE at the **BRI REIFY gate**: each candidate frame/claim is adjudicated *before* it is reified and scored. `Unwarranted` items are gated out; `Weak` items are reified TRANSIENT (flagged, not scored as established); `Explicit`/`TacitlyWarranted` items pass with their warrant attached. This is the precision gate ISIT depends on (§7.1).

---

## 5. Link to m4-EQUILIBRIUM — disagreement resolution

When `judge:JudgeInstance`s disagree on an item, the disagreement is a **multi-stakeholder balancing problem**, exactly EQUILIBRIUM's remit. Each instance is a stakeholder with a utility shaped by its `judge:disposition`. EQUILIBRIUM models the verdict utilities and returns the equilibrium verdict:

- A **precision-oriented** instance (e.g. a conservative backend) assigns high utility to withholding warrant absent an airtight mechanism (→ `Weak`/`Unwarranted`).
- A **recall-oriented** instance (e.g. a generous backend) assigns high utility to crediting a licensed tacit reading (→ `TacitlyWarranted`).
- The **equilibrium rule** operationalises the admissibility rule of §2: an item resolves to `TacitlyWarranted` iff *at least one* instance supplies a specific licensing mechanism **and** *no* instance classes it `Unwarranted`; it resolves to `Weak` if mechanisms are offered but contested; to `Unwarranted` if any instance shows positive projection evidence and none supplies a mechanism. This is a Pareto move: it keeps every warranted-by-mechanism item while discarding unlicensed ones — the same logic that motivates the **ensemble extension** (precision-oriented and recall-oriented backends compensating for one another), now made the verdict-aggregation rule.

`EQUILIBRIUMOPTIMIZER` produces `equi:Compromise` per contested item; DEEPJUDGE wraps it as `judge:resolvedVerdict`.

## 6. Link to m7-DISPUTATIO — contested-item adjudication

Items EQUILIBRIUM flags as genuinely contested (a warranted-vs-projection split) are escalated to **DISPUTATIO**, which argues *in utramque partem*:
- **Thesis** (`polanyi:thesis`): `x` is warranted — state the licensing mechanism `μ` and the source content it rests on.
- **Antithesis** (`polanyi:antithesis`): `x` is projection — `μ` does not apply, or `σ(x)` underdetermines `x`, or a competing reading is more parsimonious.
- **Synthesis** (`polanyi:synthesis`): the adjudicated verdict (`TacitlyWarranted` / `Weak` / `Unwarranted`) **with its argument as provenance** — so a borderline tacit frame carries an auditable case, not a bare label.

DISPUTATIO is thus DEEPJUDGE's deliberation organ for exactly the rows a human reviewer would otherwise have to arbitrate; the `DIALECTIC` agent records the full thesis/antithesis/synthesis chain.

## 7. Link to m25-DARKSIDE — warrant substrate and delegation risk

DEEPJUDGE and DARKSIDE are complementary halves of one adjudicative stance:
- **DARKSIDE (via negativa)** tracks what is wrongly **included** (fabrication, on the warrant axis) and wrongly **excluded** (constancy violations). Its warrant axis is **stated-only**: an unattested item is `dark:UnattestedCommitment`.
- **DEEPJUDGE (tacit-positive)** adds the missing category: an item unattested *on the surface* may still be `judge:TacitlyWarranted` *if a mechanism licenses it*. Without DEEPJUDGE, DARKSIDE alone would wrongly penalise the method's tacit yield as unattested.

Interface: DEEPJUDGE **requires** m25 and **reuses** `dark:hasWarrant` + the warrant classes; every `judge:Unwarranted` verdict emits a `dark:FabricatedCommitment` into the `dark:WarrantProfile`, and `judge:overProjectionRate` **is** the extraction's `dark:fabricatedRate`. DARKSIDE's `dark:DelegationRiskAssessment` is then computed on DEEPJUDGE-supplied rates, with thresholds extended consistently:

```
TRUSTLESS  : overProjectionRate = 0  AND weakRate ≤ 0.05  AND tacitCriterionPrecision ≥ 0.95
SUPERVISED : overProjectionRate = 0  AND weakRate ≤ 0.20
UNSAFE     : overProjectionRate > 0  OR  weakRate > 0.40
```

(Calibrated against the human-verified ISIT pilot, where both backends sat at the TRUSTLESS/SUPERVISED boundary: tacit precision ≈0.95–0.96, over-projection 0.00–0.02.)

**These thresholds are provisional** — fitted to a single corpus (n=18, one deployment) and stated as a working boundary, not a validated one. They should be re-estimated against a second corpus before being cited as load-bearing; until then DEEPJUDGE emits them with `judge:thresholdProvenance "ISIT-pilot-n18"` so downstream consumers can see the basis.

### 7.2a Cross-family independence rule (overrulable)

A `TRUSTLESS` delegation verdict (and a TRUSTLESS-grade `judge:tacitCriterionPrecision` claim) is **admissible in Mode B only if the judge panel contains at least one `judge:JudgeInstance` whose `judge:instanceFamily` differs from the extractor's, or a human instance.** Rationale: when judge and extractor share a model family, inter-judge agreement is partly *self*-agreement and inflates apparent precision; a same-family panel cannot certify a backend against its own dispositions. If the panel is same-family only, the verdict is **capped at `SUPERVISED`** regardless of the computed rates.

**Override:** the rule is overrulable. Setting `judge:independenceOverride true` lifts the cap, but requires a recorded `judge:overrideRationale` (e.g. "no cross-family backend available in this deployment; risk accepted for internal triage only"). The override and its rationale are written into the `judge:ConsensusReport`, so any relaxation of the independence guarantee is auditable rather than silent. Absent the flag, the default is enforcement (cap at SUPERVISED).

This is a soft, defeasible constraint, not a hard gate: it never blocks judging or downgrades anything below SUPERVISED; it only withholds the *strongest* trust grade until the panel has earned cross-family independence.

---

## 7.1 Dependency for ISIT (m24-ISITPROFILE)

DEEPJUDGE becomes a **hard prerequisite** of ISITPROFILE. The change to m24:

- **Mode A** gains a verdict gate: `frame extraction → m30 Mode C (gate) → clustering → ISIT scoring`. Only `Explicit`/`TacitlyWarranted` frames are scored; `Unwarranted` frames are dropped before they distort a profile; `Weak` frames enter as TRANSIENT and are excluded from the headline score. This is the precision control that turns "the model is profligate" into "the profile contains only warranted frames."
- **Mode B** (inter-rater) reuses DEEPJUDGE Mode B: cross-rater/cross-model frame agreement is computed as `judge:interJudgeAgreement`, and each rater's frame set is its own `judge:JudgeInstance`. ISIT's reliability statistics (Krippendorff α, ICC) are thus DEEPJUDGE consensus statistics.
- m24's `Requires` line becomes: `Requires b0-BRI, b6-IMPACT, b10-ToM, m1-TACIT, m30-DEEPJUDGE.`

The empirical payoff (from calibration): the per-backend `judge:PrecisionProfile` and `judge:tacitYield` become reportable ISIT properties — "rationale-gated, tacit-criterion precision P (human-verified), tacit yield Y" — which is the defensible accuracy claim ISIT previously lacked.

## 7.2 Reuse in DISCOVERY (Mode 2 / PDL)

The Discovery Loop appraises generated structures informally (m26's APPRAISE phase scores conjectures by parsimony/coherence/fertility/falsifiability/novelty; m27 asks whether a test truly discriminates). DEEPJUDGE formalises these as adjudications:
- **m26-ABDUCTIVE-DISCOVERY**: each `disc:Hypothesis` is judged — is it *warranted by the gap and evidence* (explicit support / tacit licensing via analogy or causal inversion), `Weak`, or `Unwarranted` (ungrounded speculation)? `judge:` verdicts supply auditable warrant for `disc:AppraisalScore`, and `Unwarranted` conjectures are pruned before CRYSTALLIZE.
- **m27-EXPERIMENTDESIGN**: each candidate test is judged for whether it genuinely discriminates the hypothesis pair (a `TacitlyWarranted`-style judgment licensed by the predicted-difference structure), gating weak tests out of the protocol.
- Multi-judge Mode B + DISPUTATIO give the Discovery Loop a principled tribunal for competing conjectures, replacing single-pass scoring.

## 7.3 Reuse in ALIGN (cross-model / inter-ontology correspondence)

ALIGN (the multi-case/inter-rater work of m24 Mode B, extended to cross-ontology frame correspondence — the silver-standard / ALIGN-matrix setting) judges whether two items from different models or ontologies **correspond**. DEEPJUDGE supplies the formal judge:
- For each candidate correspondence `(frame_A ↔ frame_B)` or `(frame ↔ reference_frame)`, a verdict: `Explicit` (lexically/definitionally identical), `TacitlyWarranted` (semantically licensed — e.g. a bundling/coarsening relation, with the mechanism named), `Weak` (partial), or `Unwarranted` (orphan / false correspondence).
- A **reference ("silver standard")** typology is itself a `judge:JudgeInstance`'s target; scoring each model *against the reference* (rather than against each other) decouples the typology gap from the reporting-density gap, and DEEPJUDGE's `PrecisionProfile` per model is the per-backend coverage fingerprint.
- Disagreements over correspondence go through EQUILIBRIUM; contested orphans/bundles go through DISPUTATIO. This makes the ALIGN matrix's confidences *warranted and auditable* rather than hand-set.

---

## 8. Worked example (ISIT catcalling pilot, human-verified)

Input: 18 verbal reports; two backends' frame extractions; DEEPJUDGE Mode B with two judge instances + two human instances.

- Per-item verdicts with `judge:licensedBy` (e.g. `fear` on report P1 → `TacitlyWarranted`, `licensedBy b10-ToM`, `restsOn "moved away … if I were a man I would have answered back"`).
- `judge:interJudgeAgreement` (human instances) = 0.99.
- `judge:PrecisionProfile`: generous backend — `tacitCriterionPrecision 0.96`, `tacitYield 0.45`, `overProjectionRate 0.02`; conservative backend — `tacitCriterionPrecision 0.95`, `tacitYield 0.14`, `overProjectionRate 0.00`.
- `dark:DelegationRiskAssessment` = SUPERVISED→TRUSTLESS boundary for both.
- One contested item (a `Confrontation` label on an internal-anger report under the coarse backend) escalated to DISPUTATIO → synthesis: `Weak` (bundling artefact, not confrontation behaviour).

This is exactly the calibration result, now produced by a specified method rather than ad-hoc prompting.

---

## 9. Integration blocks for v24 (paste-ready)

**(a) Methods list** — after `- m29-REVISE (...)`:
```
- m30-DEEPJUDGE (Formal tacit-aware adjudication of extracted knowledge: warranted verdicts with licensing mechanisms, multi-judge consensus, precision/tacit-yield profiling; dependency of ISIT, reused in Discovery and ALIGN)
```

**(b) Agents list** — after `- EVOLVER ...`:
```
- DEEPJUDGEAGENT — Tacit-aware verdict adjudication, multi-judge consensus, EQUILIBRIUM/DISPUTATIO resolution, DARKSIDE warrant emission (m30-DEEPJUDGE)
```

**(c) STEP6 block** — after `<!-- POL:END:m29_STEP6 -->`:
```
<!-- POL:BEGIN:m30_STEP6 -->
### m30-DEEPJUDGE (Formal Tacit-Aware Adjudication)

* Formal LLM-as-a-judge over extracted items (frames, claims, commitments, alignments, hypotheses). Replaces stated-only / rubric judging with **warranted** verdicts on the lattice {Explicit, TacitlyWarranted, Weak, Unwarranted}; `TacitlyWarranted` is admissible only when a named pragmatic mechanism (b4/b5/b6/b9/b10) licenses the item from a source span — crediting the method's tacit yield without licensing projection.
* **Mode A** (item adjudication): per-item verdict + `judge:licensedBy` + `judge:VerificationSheet` + `judge:PrecisionProfile` (explicit/tacit precision, tacit yield, over-projection).
* **Mode B** (multi-judge consensus): ≥2 judge instances (different model families; optional human instance) → `judge:interJudgeAgreement` (Krippendorff α); disagreements resolved by m4-EQUILIBRIUM, contested items adjudicated by m7-DISPUTATIO. Cross-family independence rule (overrulable): a TRUSTLESS verdict requires a cross-family or human judge instance, else capped at SUPERVISED unless `judge:independenceOverride`. Outputs `judge:ConsensusReport`.
* **Mode C** (live verdict gate): adjudicates frames at the BRI REIFY gate (parallel to m25 Mode C / m21); gates `Unwarranted`, TRANSIENT-flags `Weak`. This is the ISIT precision gate.
* Reuses m25 warrant axis: `judge:Unwarranted → dark:FabricatedCommitment`; `judge:overProjectionRate = dark:fabricatedRate`; feeds `dark:DelegationRiskAssessment`.
* Locus `aif:LensLocus`; verdicts are active-inference events (Unwarranted = prediction error); over-projection carries `aif:flow-DARKSIDE-Backward`.
* Vocabulary prefix: `judge:`. Task type: T22-JUDGE.
* **Dependencies**: →R b0-BRI, m1-TACIT, m25-DARKSIDE, b4-PRESUPPOSITION, b5-IMPLICATURE, b6-IMPACT, b9-CAUSALITY, b10-ToM; →E m24-ISITPROFILE (precision gate), m4-EQUILIBRIUM (resolution), m7-DISPUTATIO (contested adjudication), m26-ABDUCTIVE-DISCOVERY (conjecture appraisal), m6-ACTIVE-INFERENCE.
* **Full specification**: m30_DEEPJUDGE_spec.md
<!-- POL:END:m30_STEP6 -->
```

**(d) STEP7 agent block** — after the last STEP7 agent block:
```
<!-- POL:BEGIN:m30_STEP7 -->
### m30-DEEPJUDGE → [DEEPJUDGEAGENT]

**Agent Specification:**
```turtle
mcp:DeepJudgeAgent a mcp:Agent ;
    mcp:implements polanyi:m30_DEEPJUDGE ;
    mcp:endpoint "http://localhost:8033/deepjudge" ;
    mcp:timeout "PT120S"^^xsd:duration ;
    mcp:capabilities (
        mcp:ItemAdjudication
        mcp:MechanismLicensing
        mcp:MultiJudgeConsensus
        mcp:InterJudgeAgreement
        mcp:EquilibriumResolution
        mcp:DisputatioAdjudication
        mcp:WarrantEmission
        mcp:PrecisionProfiling
        mcp:VerificationSheetExport
    ) ;
    mcp:inputFormat "text/turtle" ; mcp:outputFormat "text/turtle" ;
    rdfs:comment "Mode A/B accept XKGs or candidate item sets; Mode C is a live REIFY-gate layer. Operates at aif:LensLocus."@en .
```

**Agent Interaction Protocol (m12-AGENTIC mode):**
1. **Mode C** (gate): DEEPJUDGEAGENT wraps the REIFY gate alongside DARKSIDEAGENT (m25-C) and CUECOHERENCEAGENT (m21); adjudicates each candidate frame before reification; gates `Unwarranted`, TRANSIENT-flags `Weak`.
2. **Mode A** (post-hoc): receives completed E; for each item harvests `σ(x)` and producing-heuristic `h(x)` from provenance; assigns verdict; for `TacitlyWarranted` records `judge:licensedBy` the licensing b-heuristic; emits `judge:VerificationSheet` + `judge:PrecisionProfile`.
3. **Mode B** (consensus): OVERVIEWING schedules ≥2 judge instances; DEEPJUDGEAGENT computes `judge:interJudgeAgreement`; routes disagreements to EQUILIBRIUMOPTIMIZER (m4) and contested items to DIALECTIC (m7); enforces the cross-family independence rule (caps TRUSTLESS at SUPERVISED for same-family panels unless `judge:independenceOverride` with rationale); emits `judge:ConsensusReport`.
4. Emits `dark:FabricatedCommitment` for each `judge:Unwarranted` into DARKSIDEAGENT's WarrantProfile; supplies rates to `dark:DelegationRiskAssessment`.
5. When invoked by ISITPROFILEAGENT (m24), runs Mode C as the frame precision gate; when invoked by CONJECTURAGENT (m26) or in ALIGN, runs Mode A/B over hypotheses or correspondences.

**Pipeline Position:** Mode C wraps REIFY across STEP4–STEP6; Mode A after STEP6; Mode B after STEP12 (or on demand from m24/m26/ALIGN).

**Full specification:** m30_DEEPJUDGE_spec.md
<!-- POL:END:m30_STEP7 -->
```

**(e) Edit m24-ISITPROFILE STEP6** — change its dependency sentence to:
```
Requires b0-BRI, b6-IMPACT, b10-ToM, m1-TACIT, m30-DEEPJUDGE (frame precision gate, Mode C). Enhanced by m6-ACTIVE-INFERENCE, m20-ONTOLOGYRAG, m21-CUECOHERENCE, m22-ICONOGRAPH.
```
and add to its Mode A line, before clustering: `frame extraction (BRI BIND) → m30-DEEPJUDGE Mode C gate → frame clustering (BRI REIFY) → ISIT scoring (BRI INTEGRATE)`.
