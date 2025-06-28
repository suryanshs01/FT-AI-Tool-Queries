# Neo4j Graph QA - Processing Results Report

**Generated from checkpoint:** `freight_queries_all_prompts_checkpoint.csv`
**Report generated on:** 2025-06-27 22:55:29

## 📊 Executive Summary

- **Total Queries Processed:** 13
- **Successful:** 12 (92.3%)
- **Failed:** 1 (7.7%)
- **Average Attempts per Query:** 1.2
- **Processing Period:** 2025-06-27 22:34 to 2025-06-27 22:41
- **Total Processing Time:** 0:07:47.062326

### 📈 Quick Statistics

| Metric | Value |
|--------|-------|
| Success Rate | 92.3% |
| Total Queries | 13 |
| Successful | 12 |
| Failed | 1 |
| Avg Attempts | 1.2 |

---

## ✅ Successful Query Results

### 🎯 Query 1 (Row 1)

**Metadata:**
- **Timestamp:** 2025-06-27T22:34:02.132045
- **Attempts:** 1
- **Original Row:** 1

#### 💬 User Query
```
What is the overall weighted average PTPK? Formula: Average freight per ton/ Weighted lead distance by dispatch quantity
```

#### 🔧 Generated Cypher Query
```cypher
MATCH (d:Dispatch)-[:HAS_FREIGHT_SPEND]->(f:FreightSpend)
WITH avg(f.freight_per_ton) AS avgFPT,
     sum(d.quantity * d.sap_lead_distance_km) AS qtyDistanceSum,
     sum(d.quantity) AS totalQty
WITH avgFPT, qtyDistanceSum / totalQty AS weightedLead
RETURN avgFPT / weightedLead AS weighted_average_ptpk;
```

#### 📋 Query Results

**Formatted Result:**
| Metric                     | Value                |
|----------------------------|----------------------|
| Overall Weighted Average PTPK | 8.425318017855947    |

<details>
<summary>📄 Raw Result (Click to expand)</summary>

```
The overall weighted average PTPK is 8.425318017855947.
```

</details>

---

### 🎯 Query 2 (Row 2)

**Metadata:**
- **Timestamp:** 2025-06-27T22:34:26.751712
- **Attempts:** 1
- **Original Row:** 2

#### 💬 User Query
```
Show weighted average PTPK segmented by distance slabs (e.g., 0-15 KM, 16-30 KM, etc.). Formula: Average freight per ton/ Weighted lead distance by dispatch quantity
```

#### 🔧 Generated Cypher Query
```cypher
MATCH (d:Dispatch)-[:HAS_FREIGHT_SPEND]->(fs:FreightSpend)
WITH
  CASE
    WHEN d.sap_lead_distance_km <= 15 THEN '0-15 KM'
    WHEN d.sap_lead_distance_km > 15 AND d.sap_lead_distance_km <= 30 THEN '16-30 KM'
    WHEN d.sap_lead_distance_km > 30 AND d.sap_lead_distance_km <= 45 THEN '31-45 KM'
    WHEN d.sap_lead_distance_km > 45 AND d.sap_lead_distance_km <= 60 THEN '46-60 KM'
    ELSE '60+ KM'
  END AS distance_slab,
  d.sap_lead_distance_km AS distance,
  d.quantity AS qty,
  fs.freight_per_ton AS freight_per_ton
WITH
  distance_slab,
  avg(freight_per_ton) AS avg_freight_per_ton,
  sum(distance * qty) / sum(qty) AS weighted_lead_distance
RETURN
  distance_slab,
  avg_freight_per_ton / weighted_lead_distance AS weighted_avg_ptpk
ORDER BY distance_slab;
```

#### 📋 Query Results

**Formatted Result:**
| Distance Slab | Weighted-Average PTPK |
|----------------|------------------------|
| 0 – 15 KM      | 23.99                  |
| 16 – 30 KM     | 14.69                  |
| 31 – 45 KM     | 11.60                  |
| 46 – 60 KM     | 10.17                  |
| 60 + KM        | 6.71                   |

<details>
<summary>📄 Raw Result (Click to expand)</summary>

```
Here is the weighted-average PTPK (Average freight per ton ÷ Weighted lead distance by dispatch quantity) for each distance slab:

• 0 – 15 KM: 23.99  
• 16 – 30 KM: 14.69  
• 31 – 45 KM: 11.60  
• 46 – 60 KM: 10.17  
• 60 + KM:   6.71
```

</details>

---

### 🎯 Query 3 (Row 3)

**Metadata:**
- **Timestamp:** 2025-06-27T22:35:09.050860
- **Attempts:** 1
- **Original Row:** 3

#### 💬 User Query
```
List distance slabs where weighted average PTPK is higher than previous slabs with lesser lead distances. Formula: Average freight per ton/ Weighted lead distance by dispatch quantity
```

#### 🔧 Generated Cypher Query
```cypher
MATCH (d:Dispatch)-[:HAS_FREIGHT_SPEND]->(f:FreightSpend)
WHERE d.quantity IS NOT NULL 
  AND d.sap_lead_distance_km IS NOT NULL 
  AND f.freight_per_ton IS NOT NULL
WITH toInteger(floor(d.sap_lead_distance_km/100))*100 AS slab,
     f.freight_per_ton AS ftp,
     d.sap_lead_distance_km AS lead,
     d.quantity AS qty
WITH slab,
     avg(ftp) AS avg_ftp,
     sum(lead*qty) AS weighted_lead_sum,
     sum(qty) AS total_qty
WHERE total_qty > 0
WITH slab,
     avg_ftp / (weighted_lead_sum/total_qty) AS weighted_avg_ptpk
ORDER BY slab
WITH collect({slab: slab, ptpk: weighted_avg_ptpk}) AS rows
UNWIND rows AS cur
WITH cur,
     [x IN rows WHERE x.slab < cur.slab | x.ptpk] AS prev_ptpks
WITH cur,
     CASE 
        WHEN size(prev_ptpks) = 0 THEN NULL 
        ELSE reduce(m = -1.0, r IN prev_ptpks | CASE WHEN r > m THEN r ELSE m END) 
     END AS max_prev_ptpk
WHERE cur.ptpk > coalesce(max_prev_ptpk,-1)
RETURN cur.slab AS slabStartKm,
       cur.slab + 100 AS slabEndKm,
       cur.ptpk AS weightedAvgPTPK
ORDER BY slabStartKm
```

#### 📋 Query Results

```
No distance slab meets this criterion, as there is only one slab (0–100 km) and no preceding slabs with shorter lead distances to compare against.
```

---

### 🎯 Query 4 (Row 4)

**Metadata:**
- **Timestamp:** 2025-06-27T22:35:31.551903
- **Attempts:** 1
- **Original Row:** 4

#### 💬 User Query
```
Show PTPK by vehicle types (e.g., 6W, 10W, etc.) within 60-75 KM slab for trips. 
```

#### 🔧 Generated Cypher Query
```cypher
MATCH (d:Dispatch)-[:CARRIED_IN]->(v:Vehicle)
MATCH (d)-[:HAS_FREIGHT_SPEND]->(f:FreightSpend)
WHERE d.sap_lead_distance_km >= 60 AND d.sap_lead_distance_km <= 75
RETURN v.vehicle_type AS vehicleType,
       avg(f.ptpk) AS avgPTPK,
       sum(f.ptpk) AS totalPTPK,
       count(d) AS tripCount
ORDER BY avgPTPK DESC
```

#### 📋 Query Results

**Formatted Result:**
| Vehicle Type               | PTPK  |
|----------------------------|-------|
| 6W                         | 9.64  |
| 10W                        | 7.60  |
| 12W                        | 7.23  |
| 14W+                       | 6.16  |
| Unspecified vehicle type    | 9.34  |

<details>
<summary>📄 Raw Result (Click to expand)</summary>

```
PTPK within the 60-75 KM slab is:

• 6W – 9.64  
• 10W – 7.60  
• 12W – 7.23  
• 14W+ – 6.16  
• Unspecified vehicle type – 9.34
```

</details>

---

### 🎯 Query 5 (Row 5)

**Metadata:**
- **Timestamp:** 2025-06-27T22:35:46.184515
- **Attempts:** 1
- **Original Row:** 5

#### 💬 User Query
```
Show PTPK by destinations within 60-75 KM slab for 6W vehicles with lead distance details. 
```

#### 🔧 Generated Cypher Query
```cypher
MATCH (d:Dispatch)-[:DESTINATION]->(dest:Location)
MATCH (d)-[:CARRIED_IN]->(v:Vehicle)
MATCH (d)-[:HAS_FREIGHT_SPEND]->(fs:FreightSpend)
WHERE d.sap_lead_distance_km >= 60 
  AND d.sap_lead_distance_km <= 75
  AND v.vehicle_class = '6W'
RETURN dest.name AS destination,
       d.sap_lead_distance_km AS lead_distance_km,
       fs.ptpk AS ptpk
ORDER BY dest.name, lead_distance_km;
```

#### 📋 Query Results

```
I’m sorry, but I don’t have the information needed to answer that.
```

---

### 🎯 Query 6 (Row 6)

**Metadata:**
- **Timestamp:** 2025-06-27T22:36:59.235520
- **Attempts:** 1
- **Original Row:** 6

#### 💬 User Query
```
List destinations with shorter lead distances but higher PTPK for vehicle types in the same slab; show comparisons of PTPKs and volume %.
```

#### 🔧 Generated Cypher Query
```cypher
MATCH (d:Dispatch)-[:DESTINATION]->(dest:Location)
MATCH (d)-[:CARRIED_IN]->(v:Vehicle)
MATCH (d)-[:HAS_FREIGHT_SPEND]->(f:FreightSpend)
WITH dest.name AS destination,
     v.vehicle_type AS vehicle_type,
     avg(d.sap_lead_distance_km) AS dest_avg_lead_km,
     avg(f.ptpk) AS dest_avg_ptpk,
     sum(d.quantity) AS dest_qty
CALL {
  WITH vehicle_type
  MATCH (d2:Dispatch)-[:CARRIED_IN]->(v2:Vehicle {vehicle_type: vehicle_type})
  MATCH (d2)-[:HAS_FREIGHT_SPEND]->(f2:FreightSpend)
  WITH avg(d2.sap_lead_distance_km) AS type_avg_lead_km,
       avg(f2.ptpk) AS type_avg_ptpk,
       sum(d2.quantity) AS type_total_qty
  RETURN type_avg_lead_km, type_avg_ptpk, type_total_qty
}
WITH destination, vehicle_type,
     dest_avg_lead_km, type_avg_lead_km,
     dest_avg_ptpk, type_avg_ptpk,
     dest_qty, type_total_qty
WHERE dest_avg_lead_km < type_avg_lead_km AND dest_avg_ptpk > type_avg_ptpk
WITH vehicle_type,
     destination,
     dest_avg_lead_km,
     type_avg_lead_km,
     dest_avg_ptpk,
     type_avg_ptpk,
     round(dest_qty * 100.0 / type_total_qty, 2) AS volume_percent
RETURN vehicle_type,
       destination,
       dest_avg_lead_km,
       type_avg_lead_km,
       dest_avg_ptpk,
       type_avg_ptpk,
       volume_percent
ORDER BY vehicle_type, volume_percent DESC
```

#### 📋 Query Results

**Formatted Result:**
```markdown
| Route                        | Lead (km) | PTPK  | Volume (%) |
|------------------------------|-----------|-------|------------|
| CHITPUR                      | 21.00     | 15.55 | 9.57       |
| DANKUNI                      | 9.90      | 22.83 | 5.64       |
| COSSIPORE-3 RLYS-S          | 18.00     | 17.17 | 3.44       |
| HOWRAH                       | 20.98     | 16.88 | 3.37       |
| RAJARHAT NEW TOWN           | 34.93     | 11.65 | 2.56       |
| SODEPUR                      | 24.00     | 13.73 | 1.41       |
| SREERAMPUR                   | 15.91     | 17.43 | 0.93       |
| BARRACKPORE                  | 33.91     | 11.10 | 0.80       |
| RAJARHAT                     | 27.88     | 14.56 | 0.71       |
| MADHYAMGRAM                  | 27.74     | 13.59 | 0.65       |
```

<details>
<summary>📄 Raw Result (Click to expand)</summary>

```
For 10-wheelers (type average lead ≈ 56.15 km, type average PTPK ≈ 10.87), every listed route has a shorter lead distance yet commands a higher PTPK.  Their details and share of volume are:

• CHITPUR – Lead 21 km | PTPK 15.55 | Volume 9.57 %  
• DANKUNI – Lead 9.90 km | PTPK 22.83 | Volume 5.64 %  
• COSSIPORE-3 RLYS-S – Lead 18 km | PTPK 17.17 | Volume 3.44 %  
• HOWRAH – Lead 20.98 km | PTPK 16.88 | Volume 3.37 %  
• RAJARHAT NEW TOWN – Lead 34.93 km | PTPK 11.65 | Volume 2.56 %  
• SODEPUR – Lead 24 km | PTPK 13.73 | Volume 1.41 %  
• SREERAMPUR – Lead 15.91 km | PTPK 17.43 | Volume 0.93 %  
• BARRACKPORE – Lead 33.91 km | PTPK 11.10 | Volume 0.80 %  
• RAJARHAT – Lead 27.88 km | PTPK 14.56 | Volume 0.71 %  
• MADHYAMGRAM – Lead 27.74 km | PTPK 13.59 | Volume 0.65 %

Thus, all ten destinations fall into the “shorter lead / higher PTPK” category for the 10-wheel vehicle slab.
```

</details>

---

### 🎯 Query 7 (Row 7)

**Metadata:**
- **Timestamp:** 2025-06-27T22:37:25.638531
- **Attempts:** 1
- **Original Row:** 7

#### 💬 User Query
```
List destinations with significantly different PTPK values despite for a given vehicle type
```

#### 🔧 Generated Cypher Query
```cypher
MATCH (d:Dispatch)-[:DESTINATION]->(loc:Location)
MATCH (d)-[:CARRIED_IN]->(v:Vehicle)
MATCH (d)-[:HAS_FREIGHT_SPEND]->(fs:FreightSpend)
WITH loc.name AS destination,
     v.vehicle_type AS vehicle_type,
     avg(fs.ptpk) AS avg_ptpk,
     stDevP(fs.ptpk) AS sd_ptpk
WHERE sd_ptpk > 0.2 * avg_ptpk
RETURN destination,
       vehicle_type,
       avg_ptpk,
       sd_ptpk
ORDER BY sd_ptpk DESC
```

#### 📋 Query Results

**Formatted Result:**
```
| Location          |
|-------------------|
| SHALIMAR          |
| KRISHNANAGAR     |
```

<details>
<summary>📄 Raw Result (Click to expand)</summary>

```
SHALIMAR, KRISHNANAGAR
```

</details>

---

### 🎯 Query 8 (Row 8)

**Metadata:**
- **Timestamp:** 2025-06-27T22:38:01.245808
- **Attempts:** 1
- **Original Row:** 8

#### 💬 User Query
```
List top 5 lanes with highest PTPK increase in last 12 months where volume > 1000 ton/month.
```

#### 🔧 Generated Cypher Query
```cypher
MATCH (d:Dispatch)-[:ORIGIN]->(o:Location)
MATCH (d)-[:DESTINATION]->(dest:Location)
MATCH (d)-[:HAS_FREIGHT_SPEND]->(fs:FreightSpend)
WHERE d.creation_date >= date() - duration('P24M')
WITH o.name AS origin, dest.name AS destination,
     fs.ptpk AS ptpk,
     d.quantity AS qty,
     CASE WHEN d.creation_date >= date() - duration('P12M') THEN 1 ELSE 0 END AS recent
WITH origin, destination,
     avg(CASE WHEN recent = 1 THEN ptpk END)     AS recent_avg_ptpk,
     avg(CASE WHEN recent = 0 THEN ptpk END)     AS previous_avg_ptpk,
     sum(CASE WHEN recent = 1 THEN qty  ELSE 0 END) AS recent_qty
WHERE previous_avg_ptpk IS NOT NULL
  AND recent_avg_ptpk  IS NOT NULL
  AND recent_qty > 12000    // >1000 ton/month over 12 months
RETURN origin,
       destination,
       recent_avg_ptpk - previous_avg_ptpk AS ptpk_increase
ORDER BY ptpk_increase DESC
LIMIT 5
```

#### 📋 Query Results

```
I don’t know the answer.
```

---

### 🎯 Query 9 (Row 9)

**Metadata:**
- **Timestamp:** 2025-06-27T22:39:28.474834
- **Attempts:** 2
- **Original Row:** 9

#### 💬 User Query
```
Show transporters on above lanes in descending SOB% and their respective PTPK 
```

#### 🔧 Generated Cypher Query
```cypher
MATCH (d:Dispatch)-[:ORIGIN]->(o:Location)
MATCH (d)-[:DESTINATION]->(de:Location)
MATCH (d)-[:TRANSPORTED_BY]->(tr:Transporter)
MATCH (d)-[:HAS_FREIGHT_SPEND]->(fs:FreightSpend)
WITH o, de, tr, count(d) AS dispatch_cnt, avg(fs.ptpk) AS avg_ptpk
WITH o, de, collect({tr: tr, cnt: dispatch_cnt, ptpk: avg_ptpk}) AS stats, sum(dispatch_cnt) AS total_cnt
UNWIND stats AS s
WITH o, de, s.tr AS transporter, s.cnt AS tr_cnt, s.ptpk AS ptpk, total_cnt
RETURN o.name AS origin,
       de.name AS destination,
       transporter.name AS transporter,
       round(tr_cnt * 100.0 / total_cnt, 2) AS sob_percentage,
       ptpk
ORDER BY sob_percentage DESC;
```

#### 📋 Query Results

**Formatted Result:**
| Origin                     | Destination                | Transporter | SOB   | PTPK  |
|---------------------------|----------------------------|-------------|-------|-------|
| DANKUNI                   | SANTOSHPUR(J)             | 11          | 100%  | 9.14  |
| DANKUNI                   | NAKTALA                   | 11          | 100%  | 11.35 |
| DANKUNI                   | JUMAINASKAR HAT           | 11          | 100%  | 8.08  |
| DANKUNI                   | HOWRAH(CMC)               | 11          | 100%  | 10.23 |
| DANKUNI                   | BELEBERA                  | 7           | 100%  | 3.85  |
| DANKUNI                   | PETBINDHI                 | 7           | 100%  | 4.03  |
| DANKUNI                   | NARAYANGARH               | 7           | 100%  | 4.23  |
| DANKUNI                   | BELPAHARI                 | 7           | 100%  | 3.79  |
| DANKUNI                   | PALPARA (CHAKDAH)         | 2           | 100%  | 7.92  |
| DANKUNI                   | DASGHARA                  | 2           | 100%  | 7.07  |

<details>
<summary>📄 Raw Result (Click to expand)</summary>

```
DANKUNI → SANTOSHPUR(J): Transporter 11 – SOB 100 %, PTPK 9.14  
DANKUNI → NAKTALA: Transporter 11 – SOB 100 %, PTPK 11.35  
DANKUNI → JUMAINASKAR HAT: Transporter 11 – SOB 100 %, PTPK 8.08  
DANKUNI → HOWRAH(CMC): Transporter 11 – SOB 100 %, PTPK 10.23  
DANKUNI → BELEBERA: Transporter 7 – SOB 100 %, PTPK 3.85  
DANKUNI → PETBINDHI: Transporter 7 – SOB 100 %, PTPK 4.03  
DANKUNI → NARAYANGARH: Transporter 7 – SOB 100 %, PTPK 4.23  
DANKUNI → BELPAHARI: Transporter 7 – SOB 100 %, PTPK 3.79  
DANKUNI → PALPARA (CHAKDAH): Transporter 2 – SOB 100 %, PTPK 7.92  
DANKUNI → DASGHARA: Transporter 2 – SOB 100 %, PTPK 7.07
```

</details>

---

### 🎯 Query 10 (Row 10)

**Metadata:**
- **Timestamp:** 2025-06-27T22:39:47.840194
- **Attempts:** 1
- **Original Row:** 10

#### 💬 User Query
```
Show PTPK by different vehicle types (e.g., 6W, 10W) for each lane 
```

#### 🔧 Generated Cypher Query
```cypher
MATCH (d:Dispatch)-[:ORIGIN]->(o:Location),
      (d)-[:DESTINATION]->(dest:Location),
      (d)-[:CARRIED_IN]->(v:Vehicle),
      (d)-[:HAS_FREIGHT_SPEND]->(f:FreightSpend)
RETURN o.location_code AS origin,
       dest.location_code AS destination,
       v.vehicle_type AS vehicle_type,
       avg(f.ptpk) AS avg_ptpk
ORDER BY origin, destination, vehicle_type
```

#### 📋 Query Results

**Formatted Result:**
| Outbound Lane | Vehicle Type   | PTPK    |
|---------------|----------------|---------|
| W907          | 12W trucks     | 4.7013  |
|               | 14W+ trucks    | 4.5000  |
| W909          | 10W trucks     | 15.5473 |
|               | 12W trucks     | 14.3009 |
|               | Unspecified     | 15.1960 |
| W911          | 10W trucks     | 22.5112 |
|               | Unspecified     | 22.3316 |
| W915          | 10W trucks     | 9.9132  |
|               | 12W trucks     | 9.2873  |
|               | 14W+ trucks    | 8.9250  |

<details>
<summary>📄 Raw Result (Click to expand)</summary>

```
PTPK by vehicle type on each OC12 outbound lane:

• OC12 → W907  
  – 12W trucks: 4.7013  
  – 14W+ trucks: 4.5000  

• OC12 → W909  
  – 10W trucks: 15.5473  
  – 12W trucks: 14.3009  
  – (unspecified type): 15.1960  

• OC12 → W911  
  – 10W trucks: 22.5112  
  – (unspecified type): 22.3316  

• OC12 → W915  
  – 10W trucks: 9.9132  
  – 12W trucks: 9.2873  
  – 14W+ trucks: 8.9250
```

</details>

---

### 🎯 Query 11 (Row 11)

**Metadata:**
- **Timestamp:** 2025-06-27T22:40:00.979831
- **Attempts:** 1
- **Original Row:** 11

#### 💬 User Query
```
Show PTPK by vehicle categories (own, attached, market) on lanes 
```

#### 🔧 Generated Cypher Query
```cypher
MATCH (d:Dispatch)-[:ORIGIN]->(o:Location)
MATCH (d)-[:DESTINATION]->(dest:Location)
MATCH (d)-[:CARRIED_IN]->(v:Vehicle)
MATCH (d)-[:HAS_FREIGHT_SPEND]->(f:FreightSpend)
WHERE v.category IN ['own', 'attached', 'market']
RETURN o.name + ' - ' + dest.name AS lane,
       v.category AS vehicle_category,
       avg(f.ptpk) AS avg_ptpk
ORDER BY lane, vehicle_category
```

#### 📋 Query Results

```
I’m sorry, I don’t have that information.
```

---

### 🎯 Query 12 (Row 12)

**Metadata:**
- **Timestamp:** 2025-06-27T22:40:10.225010
- **Attempts:** 1
- **Original Row:** 12

#### 💬 User Query
```
Which transporter has the best (least) PTPK for Dankuni to Tiljala lane using 6W vehicles?
```

#### 🔧 Generated Cypher Query
```cypher
MATCH (:Location {name:'Dankuni'})<-[:ORIGIN]-(d:Dispatch)-[:DESTINATION]->(:Location {name:'Tiljala'})
MATCH (d)-[:TRANSPORTED_BY]->(t:Transporter)
MATCH (d)-[:CARRIED_IN]->(:Vehicle {vehicle_type:'6W'})
MATCH (d)-[:HAS_FREIGHT_SPEND]->(fs:FreightSpend)
WITH t.name AS transporter, avg(fs.ptpk) AS avg_ptpk
RETURN transporter, avg_ptpk
ORDER BY avg_ptpk ASC
LIMIT 1
```

#### 📋 Query Results

```
I don’t know the answer.
```

---

## ❌ Failed Query Results

### ⚠️ Failed Query 1 (Row 13)

**Metadata:**
- **Timestamp:** 2025-06-27T22:41:49.194371
- **Attempts:** 3
- **Original Row:** 13

#### 💬 User Query
```
Show monthly SOB% of best performing transporter (% of total lane volume for specified vehicle type)
```

#### ❌ Error Details
```
No result returned from QA system
```

---

## 📋 Report Information

- **Source Checkpoint:** `freight_queries_all_prompts_checkpoint.csv`
- **Report Generated:** 2025-06-27 22:55:29
- **Total Entries Processed:** 13
- **Markdown File:** `freight_queries_all_prompts_report.md`
