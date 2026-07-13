Transform source text into Semantic IR (SIR). Optimize for 3 goals in priority order:

1. KEY INFORMATION (100%) — every action, causal chain, constraint, deferred work must be captured.
   — ⚠️ Semantic Coverage: Every entity/concern targeted in Act: or defer: MUST have a corresponding
     causal chain in Ctx: mentioning its keywords (e.g., rm:X requires X to appear in a Ctx: chain).
   — Missing rm:, missing Ctx chain, or dropped defer: = automatic failure.
   — ⚠️ Parameter and Constraint Preservation: Do not omit or simplify concrete constraints, thresholds, or parameter values (e.g., specific counts like `summary_sentences=3` or thresholds like `>= 1000`) mentioned in the source text. They must be explicitly preserved in `param:` or `logic:`.

2. SEMANTIC ACCURACY (≥96%) — entity names and verbs must reflect the source.
   "replacing X with Y" → rm:X + upd/cr:Y. "eliminating X" → rm:X (never encode as why:).
   Split different concerns into separate entries, even within same class/method.
   — ⚠️ No Extrapolation: Do not add, guess, or hallucinate metadata (such as `@Meta module`, `@Meta date`, or `@Chg scope`) that is not explicitly present or directly derivable from the source text. Omit these fields if the source does not provide them.

3. TOKEN COMPRESSION (≥40%) — eliminate prose, headers, passive voice. Extract verb-target-param.

### NOTATION

@Meta date:[YYYY-MM-DD]
@Chg:[Name] [scope:Module]         — block scope inherited by ALL Act: items below
Ctx: cause -> effect -> symptom    — ONE chain per Ctx: line; no comma-joining. Left-to-right flow only.
                                   — ⚠️ Ctx: lines MUST NOT start with `- `. A hyphen will break the parser.
Act:                               — items MUST start with `- ` (hyphen+space). `*` = FORBIDDEN.
- verb:target [key:value, ...]
defer:                             — top-level section; items also use `- `
- task [cond:...]
Cons:                              — OMIT unless source explicitly states constraints
-wrong [pitfall:consequence]       — no space after prefix
+must_do [why:reason]

Verbs: upd (existing), cr/mk (new), rm, set, tune, ref, add, inj.
Modifiers: logic, param, type, scope, cond, why. Same key → | separator. Diff keys → , separator.

### EXEMPLAR

Source text:
  "Replaced field injection of ModelService with constructor injection (field marked final).
   Updated ArcadeDbServiceVectorTest to use direct instantiation, eliminating ReflectionTestUtils.
   Updated prefSize to Math.sqrt(nodeCount). Extracted MIN_CANVAS_SIZE=1500.0, CANVAS_SCALE_FACTOR=150.0.
   Replaced widthProperty listeners with Platform.runLater(() -> smartGraphPanel.init()) after scene mount.
   Set SmartRandomPlacementStrategy to prevent bounding-circle collisions.
   Tuned ForceDirectedSpringGravityLayoutStrategy: repulsiveForce=2000, attractionForce=0.2, gravity=0.1.
   Added canvas recalculation in expandNode on vertex load; stored expanded node IDs.
   Deferred applyDefaultStyle until after smartGraphPanel.update() so nodes exist on JavaFX thread."

SIR output:
@Meta
date:[YYYY-MM-DD]
module:[quado-core|quado-desktop]

@Chg:ArcadeDbService_DI [scope:quado-core]
Ctx: field_injection -> obscured_dependencies -> reflection_reliance
Act:
- ref:ArcadeDbService [type:constructor_injection, param:modelService=final]
- upd:ArcadeDbServiceVectorTest [logic:direct_instantiation]
- rm:ReflectionTestUtils

@Chg:GraphPanel_Layout_Fix [scope:GraphController]
Ctx: linear_scaling -> massive_canvas_amplified_gravity -> jitter
Ctx: expandNode_missing_prefSize_update -> cramped_new_nodes
Ctx: applyDefaultStyle_before_update -> missing_JavaFX_nodes -> unstyled_nodes
Ctx: bounding_circle_placement -> boundary_collisions -> jitter
Ctx: widthProperty_listener -> timing_issues -> initialization_race
Act:
- upd:prefSize [logic:Math.sqrt(nodeCount), param:MIN_CANVAS_SIZE=1500.0|CANVAS_SCALE_FACTOR=150.0]
- set:PlacementStrategy [type:SmartRandomPlacementStrategy, why:prevent_boundary_collisions]
- rm:widthProperty_listener
- upd:GraphController [logic:Platform.runLater(smartGraphPanel.init()), cond:after_scene_mount]
- tune:ForceDirectedSpringGravityLayoutStrategy [param:repulsiveForce=2000|attractionForce=0.2|gravity=0.1]
- upd:expandNode [logic:recalculate_canvas_size]
- upd:Pipeline [logic:store_expanded_node_IDs]
defer:
- applyDefaultStyle [cond:after_smartGraphPanel.update]
