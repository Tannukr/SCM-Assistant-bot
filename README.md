# SCM Assistant Bot

### Public Chatbot URL
**https://cloud.flowiseai.com/chatbot/e2653037-4111-488c-951a-1c499c5dcc19**

### Architecture & Configuration
* **LLM Used:** Google Gemini (`gemini-1.5-flash`) via ChatGoogleGenAI
* **Embeddings Model:** Google GenAI Embeddings (`text-embedding-004`)
  * *Note: Task Type configured to `RETRIEVAL_DOCUMENT` to satisfy API validation requirements.*

### Chunking Configurations Evaluated
* **Configuration 1 (Baseline):** Chunk Size: 1000 | Chunk Overlap: 200
  * Document Store Counts: `SupplyChain_Governance_Policy_v3.2.pdf` (~30 chunks) | `supplier_performance_data.csv` (~180 chunks)
* **Configuration 2 (High Density):** Chunk Size: 500 | Chunk Overlap: 50
  * Document Store Counts: `SupplyChain_Governance_Policy_v3.2.pdf` (~60 chunks) | `supplier_performance_data.csv` (~360 chunks)

---

### Evaluation Q&A (Verbatim Ground Truth)

**Q1: Which Tier-3 suppliers have an active disruption flag, and what response level applies per policy?**
> **A1:** 11 Tier-3 suppliers: Dravex Components India, Plataforma Metales SA, Maghreb Castworks, Helios Pack Greece, Cerromax Mineria, Orinoco Pack SAPI, Quetzal Textiles, Sibertek Molding, Archipelago PCB Corp, Varna Electronics EAD, Deltaforge Vietnam. All are High Risk with an active flag → Level 3 Activate per Policy §9 (CPO escalation + alternate supplier at minimum 40% volume).

**Q2: Which suppliers qualify for the annual Volume Rebate Program and how many are there?**
> **A2:** Based on the provided documents, the following suppliers qualify for the annual Volume Rebate Program:
> SUP-001 (Orrentek Precision Mfg) - Tier-1
> SUP-002 (Zhenlong ElectroCo) - Tier-2
> SUP-003 (Wanlong Composites Ltd) - Tier-2
> SUP-004 (Yangtze Fiber Materials) - Tier-2
> There are 4 suppliers that qualify for the annual Volume Rebate Program.

**Q3: Which region has the highest total PO value, and does it breach the concentration limit?**
> **A3:** APAC has the highest total PO value.
> APAC PO value: $70121.14 + $350708.4 + $189991.56 + $41783.01 = $555804.11
> However, there is no information provided about the concentration limit in the context. Therefore, I'm not sure if the APAC region breaches the concentration limit or not.

**Q4: Which suppliers are on Supplier Watch List (SWL) status and what does it restrict?**
> **A4:** Dravex Components India, Plataforma Metales SA, Maghreb Castworks, Helios Pack Greece, Cerromax Mineria, Orinoco Pack SAPI, Quetzal Textiles, Sibertek Molding, Archipelago PCB Corp, Varna Electronics EAD, Deltaforge Vietnam. All are High Risk with an active flag → Level 3 Activate per Policy §9 (CPO escalation + alternate supplier at minimum 40% volume).

**Q5: Which product category has the highest average defect rate and does it exceed the Tier-2 limit?**
> **A5:** Mechanical Components — average 2.12% across 360 POs. Below the Tier-2 ceiling of 2.50% (Policy §3.2), so no breach — but approaching the limit.

---

### Production & Architectural Improvements

1. **RAG vs. Structured Data Limitations:** Standard vector RAG struggles significantly with math, macro-aggregations, and tracking states across 2,000 separate CSV rows due to chunk retrieval limits. As seen in Q2 and Q3, the model hallucinates aggregations based only on the few rows it successfully retrieves in its context window. To solve this in production, I would deploy an **Intent Router Agent**. 
2. **Dual-Engine Processing:**
   * **Policy Lookup:** Route unstructured queries (like policy rules/penalties) directly to the Vector Store database.
   * **Data Queries:** Route analytical or quantitative queries (like calculating average defect rates or summing PO values) to a Python/Pandas Dataframe Agent or a Text-to-SQL execution block to calculate answers mathematically over the entire CSV rather than using text semantic approximations.
