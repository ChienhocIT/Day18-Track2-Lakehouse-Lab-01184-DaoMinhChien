# Reflection: Lakehouse Anti-Patterns in Practice

Among the **Top 5 Lakehouse Anti-Patterns**, our team is most at risk of **Anti-Pattern #3: Decoupled Vector Index Lifecycle & Stale External Retrieval**, compounded by **Anti-Pattern #2: Uncommitted Orphan Accumulation**.

### Why Our Team Is at Risk:
1. **Vector Index Staleness**: In our LLM RAG pipelines, document chunk updates and deletes occur in Delta Lake, but external vector stores (e.g., Pinecone/Milvus) are updated asynchronously without transactional dual-write guarantees. As demonstrated in NB7, queries continue retrieving stale or deleted embeddings ("phantom retrieval"), breaking GDPR/Decree 13 compliance and serving hallucinated context.
2. **False Confidence in VACUUM**: Our streaming ingestion jobs occasionally crash mid-write. We mistakenly relied on Delta's `VACUUM` to clean unused storage, unaware that `VACUUM` only purges tombstoned files recorded in the transaction log, completely missing uncommitted Parquet files. This silently inflated cloud object storage costs by ~35% over time.

### Mitigation:
We are transitioning to unified in-table vector storage (`fixed_size_list` / Lance format) with ACID transactional consistency, and deploying scheduled differential orphan sweeps (`filesystem_files - active_log_files`) in our daily Lakehouse maintenance cadence (Job 4).
