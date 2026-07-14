---
type: GUIDELINE
title: "Performance Optimization, RAM Lazy-Loading, Accurate Call Graph, and Relative Path Storage"
description: "Guidelines and architecture of async database loading, in-memory schema validation, lazy-loading ANTLR indexers, FQN-based method call resolution, and relative path storage for project portability."
references:
  - ArcadeDbService.java
  - GraphDbService.java
  - CodeIndexerService.java
  - SourceCodeIndexer.java
  - JavaIndexer.java
  - GraphController.java
  - DashboardController.java
  - MainController.java
  - SummaryEngineImpl.java
  - AiAnalysisService.java
---

# Context
Initially, the Quado desktop application experienced UI freezing/blocking on startup, had a high idle memory/RAM footprint (>700MB) due to eager class loading, generated false positive `CALLS` relationships where method calls with common names (like `getName()`, `run()`) were linked to all classes containing methods of that name, and stored absolute file paths in the database which broke graph indexing and analysis when projects were moved to a different directory. These issues were resolved by:
1. Blocking operations (database connection, filesystem scanning, statistics queries) were moved to background worker threads.
2. Property/index verification is now performed using in-memory schema cache instead of slow SQL queries.
3. Strategy indexers are lazy-loaded only when a file with their matching extension is first scanned.
4. Unique FQN and signatures are used for method vertices, and target class resolution via imports and local scoping is used to link calls accurately without fallback name-matching.
5. Method-to-method edges (`Function` to `Function`) are filtered out from the Graph Viewer to reduce visual complexity and clutter.
6. Storing relative file paths (relative to project root) in the database to ensure project portability, resolving relative paths back to absolute paths on the fly when reading physical files from disk.

# Architecture & Implementation Details

## 1. Thread-Safe Asynchronous Database & Stats Setup
To ensure the UI starts instantly and remains completely responsive:
- The `pathField` listener in `DashboardController` handles project switching asynchronously using `CompletableFuture.runAsync()`.
- The database connection (`initializeProjectDatabase`) and the file scan counts (`refreshStats`) run in a background worker thread.
- Graph loading is safely wrapped in `Platform.runLater()` in `MainController`'s `notifyPathChanged()` to avoid concurrent modifications to JavaFX UI nodes from background threads.
- Duplicate startup DB connection calls have been eliminated.

## 2. In-Memory Schema Verification Cache
Instead of executing sequential `CREATE PROPERTY ... IF NOT EXISTS` and `CREATE INDEX ... IF NOT EXISTS` commands on every startup (which incurs database parser/transaction overhead):
- `ArcadeDbService` checks the memory-cached schema representation of ArcadeDB (`existsProperty` and `getPolymorphicIndexByProperties`).
- SQL creation commands are only triggered if a property or index is missing, resulting in **0** database setup query executions on subsequent startups.

## 3. Lazy-Loaded Strategy Indexers
To reduce Metaspace and Heap memory usage:
- Removed Spring's `@Component` annotation from all strategy indexers: `JavaIndexer`, `GoIndexer`, `PythonIndexer`, `TypeScriptIndexer`, `VueIndexer`, `DelphiIndexer`, `SqlIndexer`.
- Refactored `CodeIndexerService` to manage indexers using a thread-safe registry of `Supplier` instances.
- Indexer classes and their transitive ANTLR dependencies are loaded and instantiated on-demand only when a file with their matching extension is first encountered.
- Projects containing only a subset of supported languages (e.g. only Java) will never load or instantiate the massive parser classes for other languages, saving up to 200MB+ in class metadata and garbage collection overhead.

## 4. Accurate FQN & Target-Resolved Call Graph
To completely eliminate false positive `CALLS` edges in the graph database:
- **Fully Qualified Method Names:** Java methods are matched and created in `JavaIndexer` using a name pattern that includes the package, class name, and method signature: `packageName.ClassName.methodSignature`.
- **Target Class Resolution:** When resolving the receiver of a method call, `JavaIndexer` checks the local variable types and imports:
  - If a variable's type is found in the imports of the file, it is resolved to its fully qualified type name (e.g. `vn.cxn.graph.controller.MainController`).
  - If a call is local and has no scope (implicit `this` receiver), it is automatically mapped to the current class FQN.
- **Strict Matching in Database:** In `CodeIndexerService.java`, the `CALLS` edge Cypher query matches target vertices using either `target.fullClassName = $targetClass` or `target.className = $targetClass`.
- **No Name-Only Fallback:** The fallback name-only matching logic (which matched callers to all functions of the same short name across unrelated classes when the receiver class was unknown) has been removed. `CALLS` relationships are now only created when the receiver type is resolvable. Other languages accept this trade-off when target class is unresolvable.

## 5. Filtered Graph Visualization for Method-to-Method Edges
To avoid visual complexity on the Graph Viewer:
- Cypher queries in `GraphController.java` (`loadGraphVisualization()`, search queries, and `expandNode()`) include the condition `AND NOT (a:Function AND b:Function)`.
- This filters out all method-to-method (`Function` -> `Function` CALLS) connections from the UI, keeping the graph focused on class containment (`Type` -> `Function`) and class-to-class dependency/injection structures.

## 6. Relative Path Storage & Project Portability
To ensure that scanned projects can be moved or copied without breaking the indexed database:
- **Relative DB Properties:** Every indexer converts absolute paths of scanned files to project-relative paths using `SourceCodeIndexer.toRelativePathStr(file, projectRoot)` before saving properties (like `filepath`) into database vertices.
- **Incremental Indexing Consistency:** `CodeIndexerService` diffs local file hashes and cleans up previous file states using relative path keys in `currentHashes` and `previousHashes`.
- **On-Demand Absolute Path Resolution:** When services need to access physical files (e.g., `SummaryEngineImpl` for building project summaries, or `AiAnalysisService` for reading file contents to generate embeddings/summaries), they resolve the stored relative paths back to absolute paths on the fly by calling `graphDbService.resolveToAbsolutePath(filepath)`. The project root is derived dynamically from the active database folder (`currentDbPath`'s grandparent folder).