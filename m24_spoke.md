<!-- POL:BEGIN:m24_SPOKE -->

## m24-ISITPROFILE: Full Procedures and Empirical Grounding

## 3. Mode A: Single-Case Situation Profiling

### 3.1 When to Use

Mode A is invoked when S represents a single source (text, image, multimodal) that has been processed by the POLANYI++ pipeline (H + M), producing an XKG E. m24 Mode A then extracts, clusters, and scores the frames present in E to produce a `isit:SituationProfile`.

### 3.2 Procedure

```
PROCEDURE m24_Mode_A(E, FO_or_U, T)

INPUT:
  E      — extended knowledge graph (XKG) produced by POLANYI++ pipeline
  FO_or_U — EITHER an externally provided ontology prior U (single-case use)
             OR an induced FrameOntology FO from Mode B's Phase 0 (multi-case use)
  T      — task specification (may constrain which frame types to extract)

OUTPUT:
  SP — isit:SituationProfile with scored frames
  FO — isit:FrameOntology (induced from E if FO_or_U is a raw U;
       or the shared FO passed from Mode B if already induced)

// ─── PHASE 1: FRAME EXTRACTION (b0-BRI BIND) ───

// 1a. Harvest frames from heuristic outputs in E
FOR EACH triple pattern (entity, heuristic_property, value) IN E:

  IF provenance = "b6-IMPACT":
    EXTRACT emotional frames (emotion type + intensity)
    EXTRACT social impact frames (relationship type + friendliness)
    EXTRACT cognitive impact frames (cognitive state)
    EXTRACT physical impact frames (embodiment, spatial)

  IF provenance = "b10-ToM":
    EXTRACT motivational frames (motivation type + confidence)
    EXTRACT intentional frames (intention type + confidence)
    EXTRACT epistemic frames (belief states, knowledge attributions)

  IF provenance = "b12-PERSPECTIVE":
    EXTRACT perspectival frames (viewpoint, lens, conceptualiser)

  IF provenance = "b9-CAUSALITY":
    EXTRACT causal frames (cause-effect links, temporal sequences)

  IF provenance = "b7-COERCION":
    EXTRACT metaphorical frames (source-target mappings, symbolic meanings)

  IF provenance = "m21-CUECOHERENCE":
    EXTRACT coherence meta-frames:
      - CoherenceVerdict (Coherent/Partial/Incoherent) as binary frame
      - Each CrossChannelTension as a named tension frame
      - Overall coherence score as intensity

  IF provenance = "m22-ICONOGRAPH" OR "b25-VISUALSEMIOTICS":
    EXTRACT semiotic frames:
      - Denotative content frames
      - Connotative meaning frames
      - Compositional structure frames (Given/New, salience)
      - Iconographic symbol frames

// 1b. Harvest cue-source frames
FOR EACH visual rationale annotation in E:
  EXTRACT cue frame with channel type (FacialExpression, Posture, Context, etc.)

// 1c. Harvest free-text frames from rationale literals
FOR EACH rdfs:comment or rationale literal in E:
  APPLY free-text frame extraction (pattern matching for:
    VisualLimitation, Curiosity, Disengagement, Waiting,
    InformationSeeking, Observation, Vulnerability, Dominance,
    Affection, Fatigue, ActiveEngagement, PassingTime, etc.)

OUTPUT: Set of raw frames F = {f_1, ..., f_n}

// ─── PHASE 2: FRAME CLUSTERING (b0-BRI REIFY) ───

// 2a. If U provides an anchoring ontology (e.g., Ekman emotions, relationship types):
IF U contains structured label sets:
  CREATE anchor clusters from U:
    - EmotionCluster (with sub-frames per Ekman category)
    - RelationshipCluster (with sub-frames per relationship type)
    - MotivationCluster (with sub-frames per motivation category)
    - IntentionCluster (with sub-frames per intention category)
    - CueCluster (with sub-frames per rationale source)
  MAP each raw frame f_i to its nearest anchor cluster

// 2b. For frames that don't map to U (emergent frames):
FOR EACH unmapped frame f_j:
  COMPUTE semantic similarity to existing clusters
  IF similarity > THRESHOLD_CLUSTER_MEMBERSHIP:
    ADD f_j to nearest cluster as an extension frame
  ELSE:
    CREATE new emergent cluster containing f_j
    LABEL the cluster based on f_j's content

// 2c. Construct Frame Ontology
FO = isit:FrameOntology containing all clusters and their frame memberships
  WITH inter-cluster relationships extracted from E:
    - "triggers" (from b9-CAUSALITY provenance)
    - "enables" (from b9)
    - "contrasts" (from b12-PERSPECTIVE provenance)
    - "coherent_with" / "tension_with" (from m21 provenance)

OUTPUT: FO, clustered frames FC = {C_1: {f_1,...}, C_2: {f_k,...}, ...}

// ─── PHASE 3: ISIT SCORING (b0-BRI INTEGRATE) ───

FOR EACH frame f_i IN FC:

  // 3a. Direct Manifestation (S_M, weight ~0.4)
  // How explicitly is this frame evoked in S?
  S_M = assess_direct_manifestation(f_i, E):
    - Explicit mention in S → high score
    - Inferred by single heuristic → moderate score
    - Inferred by chain of heuristics → lower score
    - Weighted by the number of distinct triples supporting f_i in E

  // 3b. Affective Alignment (S_E, weight ~0.3)
  // How strongly do emotional indicators in E align with this frame?
  S_E = assess_affective_alignment(f_i, E):
    - IF f_i is an emotion frame: use intensity directly
    - IF f_i is non-emotional: compute alignment between
      f_i and emotion frames co-occurring in the same subgraph
    - Modulated by valence: positive emotions amplify positive frames

  // 3c. Contextual Influence (S_I, weight ~0.3)
  // How does external/contextual data modulate this frame?
  S_I = assess_contextual_influence(f_i, E):
    - IF U provides intensity/confidence scores: use them directly
    - IF m21 provides coherence data: frames in high-tension areas
      receive prediction-error-modulated scores (via m6)
    - IF m22 provides semiotic strata: connotative frames receive
      semiotic salience scores from compositional analysis

  // 3d. Composite raw score
  S_raw = (S_M × w_M) + (S_E × w_E) + (S_I × w_I)
  // Default weights: w_M=0.4, w_E=0.3, w_I=0.3
  // Adjustable via U or T

  // 3e. ISIT normalization to [-1, +1]
  S_norm = ((S_raw - S_min) / (S_max - S_min)) × 2 - 1

  CREATE isit:FrameScore for f_i with S_norm

// 3f. Semantic space projection (for interoperability)
FOR EACH anchor cluster WITH defined semantic space:
  PROJECT frame scores onto continuous coordinates:
    - Emotion frames → valence-arousal circumplex
    - Relationship frames → intimacy-formality space
    - Motivation frames → agency-directedness space
    - Intention frames → continuity-engagement space
  MODULATE coordinates by intensity amplitude:
    amplitude = (1 + S_norm) / 2
    projected_coords = base_coords × amplitude

// ─── PHASE 4: PROFILE ASSEMBLY ───

SP = CREATE isit:SituationProfile:
  - forSituation: the interpreted situation in E
  - hasFrameScore: all computed FrameScores
  - Attach FO as the frame ontology reference
  - Annotate with provenance "m24-ISITPROFILE"

RETURN SP, FO
```

### 3.3 Graph Output (Mode A)

```turtle
# Example: Situation profile for a social gathering scene (P1)

polanyi:profile_agent_X_P1 a isit:SituationProfile ;
    isit:forSituation polanyi:social_gathering_1 ;
    isit:hasFrameScore polanyi:score_happiness, polanyi:score_friends,
                       polanyi:score_keep_listening, polanyi:score_cue_facial ;
    isit:usesOntology polanyi:frame_ontology_1 ;
    rdfs:comment "ISIT situation profile for agent X on picture P1."@en .

polanyi:score_happiness a isit:FrameScore ;
    isit:forFrame isit:Happiness ;
    isit:normalizedScore "0.25"^^xsd:float ;
    isit:coordinate "{\"valence\": 0.20, \"arousal\": 0.125}"^^xsd:string ;
    isit:manifestationWeight "0.4"^^xsd:float ;
    isit:affectWeight "0.3"^^xsd:float ;
    isit:contextWeight "0.3"^^xsd:float .

polanyi:score_friends a isit:FrameScore ;
    isit:forFrame isit:Friends ;
    isit:normalizedScore "0.50"^^xsd:float ;
    isit:coordinate "{\"intimacy\": 0.25, \"formality\": -0.15}"^^xsd:string .

polanyi:score_cue_facial a isit:FrameScore ;
    isit:forFrame isit:CueFacialExpression ;
    isit:normalizedScore "0.33"^^xsd:float .

[ a owl:Axiom ;
  polanyi:provenance "m24-ISITPROFILE" ;
  owl:annotatedSource polanyi:profile_agent_X_P1 ;
  owl:annotatedProperty isit:hasFrameScore ;
  owl:annotatedTarget polanyi:score_happiness ] .
```


## 4. Mode B: Multi-Case Statistical Profiling

### 4.1 When to Use

Mode B is invoked when S contains **multiple cases** — either multiple XKGs from different agents interpreting the same situation, or multiple XKGs from the same agent interpreting different situations, or both. Mode B aggregates Mode A profiles, computes inter-rater statistics, and outputs group-level comparisons.

### 4.2 Input Specification and Frame Ontology Induction

Mode B accepts cases in S as:
- A set of XKG files `{E_1, ..., E_n}` with agent and situation metadata
- OR a structured dataset (e.g., tabular data with categorical assignments and Likert scores) alongside XKGs
- OR references to previously computed `isit:SituationProfile` instances

Each case must specify:
- `agent_id`: unique identifier for the interpreting agent
- `agent_group`: group membership (e.g., "patients", "healthy", "AI_cl_ico")
- `situation_id`: which situation was interpreted (e.g., "P1", "P2", "P3")
- `situation_type`: metadata (e.g., "realistic", "simulated")

#### 4.2.1 The Individual XKGs as Ontology Prior: U = ⋃{E_i}

In Mode B, the ontology prior U is not supplied externally as a fixed schema. Instead, **the collected XKGs themselves constitute U**. The Frame Ontology (FO) is *induced* from the union of all input XKGs, implementing the core grounded theory principle that theory should emerge from data rather than be imposed a priori.

This works as follows:

1. **Frame vocabulary induction**: Each E_i contributes its extracted frames (from b6-IMPACT, b10-ToM, m21-CUECOHERENCE, m22-ICONOGRAPH, free-text rationales, etc.) to a common pool. The union `⋃{frames(E_i)}` defines the maximal frame vocabulary.

2. **Anchor frame identification**: Frames that originate from a shared ontology prior used during extraction (e.g., Ekman emotion categories, relationship typologies provided in U during the original POLANYI++ runs) serve as *anchor frames* — stable reference points that all agents share. These provide the interoperability backbone.

3. **Emergent frame incorporation**: Frames that appear only in specific agents' XKGs are not discarded but incorporated as *emergent extensions* of the FO. Examples:
   - AI extended frames (e.g., `CulturalResonance`, `CognitiveOverload`, `TenseConfrontation`) — present only in AI XKGs
   - Human free-text frames (e.g., `VisualLimitation`, `Disengagement`, `InformationSeeking`) — present only in human data
   - Agent-specific semiotic frames (e.g., `SnakeNecklaceSymbolism` from AI_ge_ico's iconographic analysis) — present in a single XKG

4. **Sparse profile construction**: Each individual profile SP_i is scored against the *full* induced FO. Agents who lack a frame simply score 0.0 on that dimension. This produces sparse but fully comparable profile vectors — the sparsity pattern itself is informative (e.g., `VisualLimitation` frames appearing only in patient profiles reveals a group-level cognitive signature).

5. **Cluster induction**: Phase 2 clustering operates on the pooled frame vocabulary to identify natural groupings. Anchor frames provide initial cluster nuclei; emergent frames are assigned to existing clusters by semantic similarity or form new clusters if sufficiently distant. The resulting FO captures both the shared interpretive structure and the agent-specific extensions.

This architecture has a direct connection to the **eXtreme Design (XD)** methodology for ontology engineering: the induced FO is analogous to the situated scenarios and competency questions that XD uses as input for pattern matching and ontology construction. Each frame cluster defines an implicit competency question ("can agents reliably detect this situational dimension?"), each inter-agent comparison functions as a unit test, and the emergent frames point to potential new ontology design patterns. While the full XD pipeline is beyond m24's scope, the FO produced by m24 Mode B provides the empirically grounded starting material for subsequent ontology design efforts.

```turtle
# Example: Induced Frame Ontology with anchor and emergent frames

polanyi:induced_FO_study1 a isit:FrameOntology ;
    rdfs:label "Induced Frame Ontology from ISIT Interoperability Study"@en ;
    isit:hasCluster polanyi:cluster_emotion, polanyi:cluster_relationship,
                    polanyi:cluster_motivation, polanyi:cluster_intention,
                    polanyi:cluster_cue, polanyi:cluster_coherence,
                    polanyi:cluster_emergent_visual, polanyi:cluster_emergent_cognitive ;
    isit:inductionSource "Union of 6 AI XKGs + 74 human agent profiles"@en ;
    isit:anchorFrameCount "22"^^xsd:integer ;
    isit:emergentFrameCount "31"^^xsd:integer .

# Anchor cluster (from shared U)
polanyi:cluster_emotion a isit:FrameCluster ;
    isit:hasMemberFrame isit:Happiness, isit:Sadness, isit:Anger,
                        isit:Fear, isit:Surprise, isit:Disgust, isit:Neutral ;
    isit:projectedOnto isit:ValenceArousalCircumplex ;
    rdfs:comment "Anchor cluster: all agents share Ekman categories from U."@en .

# Emergent cluster (induced from AI XKGs)
polanyi:cluster_emergent_cognitive a isit:FrameCluster ;
    isit:hasMemberFrame polanyi:CognitiveOverload, polanyi:InformationSeeking,
                        polanyi:RestingBehavior, polanyi:DividedAttention ;
    rdfs:comment "Emergent cluster: cognitive load management frames, appearing primarily in AI XKGs for P3."@en ;
    isit:sourceAgentPattern "AI agents only"@en .

# Emergent cluster (induced from human free-text)
polanyi:cluster_emergent_perceptual a isit:FrameCluster ;
    isit:hasMemberFrame polanyi:VisualLimitation, polanyi:PerceptualUncertainty ;
    rdfs:comment "Emergent cluster: perceptual limitation frames, appearing exclusively in patient profiles."@en ;
    isit:sourceAgentPattern "patient agents only"@en .
```

### 4.3 Procedure

```
PROCEDURE m24_Mode_B(Cases, T)

INPUT:
  Cases = {(E_i, agent_id_i, group_i, situation_id_i, type_i)}
  T     — task (typically: "compare groups", "assess inter-rater agreement",
          "test co-pilot hypothesis")
  NOTE: U is NOT a separate input — it is induced from ⋃{E_i}

OUTPUT:
  FO  — isit:FrameOntology (induced from the union of all XKGs)
  IR  — isit:InterRaterReport containing all statistical results
  Graphs or Literals — depending on pragmatic feasibility (see §4.5)

// ─── PHASE 0: FRAME ONTOLOGY INDUCTION (U = ⋃{E_i}) ───

// 0a. Harvest all frames from all XKGs
ALL_FRAMES = {}
FOR EACH case (E_i, ...):
  frames_i = extract_frames(E_i)  // same extraction as Mode A Phase 1
  ALL_FRAMES = ALL_FRAMES ∪ frames_i
  TAG each frame with its source agent_id and group

// 0b. Identify anchor frames (shared across ≥ 2 groups)
ANCHOR_FRAMES = {f ∈ ALL_FRAMES : |unique_groups(f)| ≥ 2}

// 0c. Identify emergent frames (unique to one group or agent)
EMERGENT_FRAMES = ALL_FRAMES \ ANCHOR_FRAMES

// 0d. Cluster all frames
FO = cluster_frames(ANCHOR_FRAMES, EMERGENT_FRAMES):
  - Anchor frames form initial cluster nuclei
  - Emergent frames assigned by semantic similarity or form new clusters
  - Each cluster annotated with:
    - Source agent pattern (which groups contribute frames)
    - Semantic space projection (if applicable)
    - Inter-cluster relationships (triggers, enables, contrasts)

// 0e. Define the full frame dimension vector
DIMS = ordered list of all frame dimensions in FO
  // Anchor dimensions first (for interoperability), then emergent

// ─── PHASE 1: PROFILE COMPUTATION (against induced FO) ───

FOR EACH case (E_i, agent_id_i, ...):
  SP_i = m24_Mode_A(E_i, FO, T)
  // Mode A now uses the INDUCED FO as its frame ontology
  // Frames absent from E_i but present in FO score 0.0

// Collect all profiles
ALL_PROFILES = {SP_1, ..., SP_n}
// All profiles are now vectors of the same dimensionality (|DIMS|)
// with sparse entries where agent-specific frames are absent

// ─── PHASE 2: GROUP AGGREGATION ───

FOR EACH unique group g:
  profiles_g = {SP_i : group_i == g}
  GP_g = CREATE isit:GroupProfile:
    - Mean vector: avg(SP_i.scores) for each frame dimension
    - SD vector: std(SP_i.scores)
    - N: count(profiles_g)
    - Median, IQR for each dimension

// ─── PHASE 3: DESCRIPTIVE STATISTICS ───

FOR EACH (group, situation) combination:
  COMPUTE:
    - Categorical distributions (mode, entropy)
      for emotion, relationship, motivation, intention
    - Continuous distributions (M, SD, Mdn, range)
      for intensity, friendliness, confidence scores
    - Cue distributions (proportions: facial, posture, context)

// ─── PHASE 4: INTER-RATER AGREEMENT ───

// 4a. Within-group categorical agreement
FOR EACH group × situation:
  COMPUTE Krippendorff's α (nominal) for:
    - emotion categories
    - relationship categories
  COMPUTE Fleiss' κ if ≥ 3 raters available

// 4b. Within-group continuous agreement
FOR EACH group × situation:
  COMPUTE ICC (intraclass correlation) for intensity/confidence scores

// 4c. Between-group profile similarity
FOR EACH pair of groups (g1, g2):
  FOR EACH situation:
    COMPUTE cosine similarity between GP_g1 and GP_g2 vectors
    COMPUTE Spearman ρ between frame salience rankings
  COMPUTE split statistics:
    - Realistic scenes mean cosine
    - Simulated scenes mean cosine
    - Overall mean cosine

// ─── PHASE 5: HYPOTHESIS TESTING ───

// 5a. Reference-based comparison (e.g., co-pilot hypothesis)
IF T specifies a reference group (e.g., "healthy"):
  FOR EACH non-reference agent/group:
    COMPUTE distance to reference centroid (Euclidean)
    COMPUTE cosine to reference centroid
  FOR EACH AI group vs patient group:
    PERFORM Mann-Whitney U test: AI cos ≥ patient cos (one-tailed)
    COMPUTE rank-biserial effect size

// 5b. Linear Mixed Models
IF T requests LMM and sufficient data (n ≥ 20):
  CONSTRUCT long-format dataset:
    DV: cos_to_reference OR individual dimension values
    Fixed: group + scene_type + group×scene_type
    Random: agent_id (intercept)
  FIT model, EXTRACT:
    - Fixed effect coefficients with p-values
    - Random effect variance (ICC)
    - Contrasts of interest (AI vs patients, realistic vs simulated)

// 5c. Per-dimension group contrasts
FOR EACH ISIT dimension:
  FIT LMM: dimension ~ is_group1 * is_simulated, random=agent_id
  EXTRACT group effect, scene effect, interaction
  FLAG dimensions with significant (p < .05) group differences

// ─── PHASE 6: RESULT ASSEMBLY ───

IR = CREATE isit:InterRaterReport:
  - hasGroupProfile: {GP_g1, GP_g2, ...}
  - hasComparison: {all pairwise ProfileComparisons}
  - hasTest: {all StatisticalTests}
  - hasDescriptives: {DescriptiveStatistics per group×situation}

RETURN IR
```

### 4.4 Cue-Channel Interaction (via m21)

When m21-CUECOHERENCE has been applied to the input cases, m24 Mode B incorporates coherence data as follows:

1. **Coherence meta-frame scoring**: For each case where m21 produced a `ccc:CoherenceReport`, m24 extracts:
   - A binary `isit:CoherenceVerdict` frame (COHERENT=1, PARTIAL=0.5, INCOHERENT=0)
   - Named tension frames with their `ccc:tensionMagnitude` as intensity
   - The `ccc:overallCoherenceScore` as a modulating factor for all other frame scores in that profile

2. **Cross-agent coherence agreement**: Mode B computes whether different agents detect the same incoherences. This is reported as a specialized inter-rater metric: "coherence detection agreement."

3. **Coherence as a co-pilot indicator**: In the reference-based comparison, cases where the AI detects incoherence that aligns with the reference group's behavior (e.g., AI flags PARTIAL_INCOHERENCE on P2, matching healthy agents' Sorpresa) receive an explicit alignment bonus annotation.

### 4.5 Output: Graphs vs. Literals

The output mode depends on pragmatic feasibility within the XKG E:

**As graph triples (preferred when feasible):**
- Individual `isit:SituationProfile` instances with `isit:FrameScore` values → always graph
- `isit:FrameOntology` with cluster structure → always graph
- `isit:ProfileComparison` instances with cosine/Spearman values → graph
- `isit:InterRaterReport` instance linking all components → graph

**As specialized comment literals (when graph encoding is impractical):**
- Large contingency tables → encode as JSON literal in `rdfs:comment` on the `isit:InterRaterReport`
- Full LMM output (coefficient tables, variance components) → encode as structured literal
- Dimension-level profiles for many groups × situations → encode as CSV/JSON literal

```turtle
# Example: LMM results as structured literal

polanyi:lmm_result_1 a isit:MixedModel ;
    rdfs:label "LMM: cos_healthy ~ group * scene_type"@en ;
    isit:pValue "0.124"^^xsd:float ;
    rdfs:comment """{"model": "cos_healthy ~ C(group) * C(scene_type)",
  "reference": "patients",
  "random": "agent_id",
  "fixed_effects": {
    "Intercept": {"beta": 0.563, "p": 0.000},
    "AI_ge_ico": {"beta": 0.218, "p": 0.169},
    "AI_ge_r1": {"beta": 0.243, "p": 0.124},
    "simulated": {"beta": 0.037, "p": 0.320}
  },
  "random_variance": 0.067,
  "residual_variance": 0.041,
  "n_observations": 240,
  "n_groups": 80}"""^^xsd:string .

[ a owl:Axiom ;
  polanyi:provenance "m24-ISITPROFILE" ;
  owl:annotatedSource polanyi:lmm_result_1 ;
  owl:annotatedProperty rdf:type ;
  owl:annotatedTarget isit:MixedModel ] .
```

```turtle
# Example: Descriptive statistics as structured literal

polanyi:descriptives_P1 a isit:DescriptiveStatistics ;
    rdfs:label "Descriptive statistics for P1"@en ;
    rdfs:comment """{"situation": "P1", "scene_type": "realistic",
  "groups": {
    "patients": {"n": 46, "emotion_mode": "Gioia", "emotion_mode_pct": 0.565,
                 "emotion_intensity_M": 3.20, "emotion_intensity_SD": 0.90,
                 "friendliness_M": 3.30, "friendliness_SD": 1.12},
    "healthy": {"n": 28, "emotion_mode": "Gioia", "emotion_mode_pct": 0.786,
                "emotion_intensity_M": 3.14, "emotion_intensity_SD": 0.69}
  }}"""^^xsd:string .
```


## 6. Empirical Grounding

m24-ISITPROFILE was designed based on the empirical findings of the ISIT interoperability study (this paper), which demonstrated:

1. **Frame extraction from XKGs is feasible**: All 6 AI agent configurations produced XKGs from which ISIT-style situation profiles could be reliably constructed, using the provenance annotations to trace each frame to its source heuristic.

2. **Continuous semantic projection improves interoperability**: Projecting categorical assignments (Ekman emotions, relationship types) onto continuous semantic spaces (valence-arousal circumplex, intimacy-formality) increased AI–human cosine similarity by +0.05 to +0.19 compared to binary encoding, because near-miss categories receive partial credit.

3. **Multi-case statistical analysis reveals pipeline effects**: Comparing 6 AI configurations × 2 engines × 3 pictures showed that engine choice (Gemini vs. Claude) explained more variance than pipeline configuration (+CLASSIFICATION vs. −CLASSIFICATION), and that the m21+m22 configuration uniquely enabled metacognitive incoherence detection.

4. **The realistic/simulated split is analytically essential**: AI agents achieved significantly higher alignment with healthy humans on realistic scenes (cosine 0.73–0.81) than on simulated/staged scenes (0.53–0.79), confirming that co-pilot feasibility is scene-type-dependent.

5. **Statistical power requires multi-agent aggregation**: With only 1 observation per AI agent per picture, formal significance testing was limited. m24 Mode B is designed to accommodate arbitrary numbers of agents and cases, enabling properly powered analyses as POLANYI++ is deployed at scale.

6. **The induced Frame Ontology seeds formal ontology design**: The FO produced by m24 Mode B — with its anchor clusters, emergent extensions, and inter-cluster relationships — provides empirically grounded input for subsequent ontology engineering via the eXtreme Design (XD) methodology. Each frame cluster defines a situated scenario; each inter-agent comparison implicitly poses a competency question; each statistical test serves as a unit test. While the full XD pipeline (pattern matching, modular ontology construction, formal verification) is beyond m24's scope, the FO bridges the gap between inductive knowledge extraction and principled ontology design, ensuring that the resulting ontology is grounded in real situated interpretations rather than armchair categorization.

<!-- POL:END:m24_SPOKE -->
