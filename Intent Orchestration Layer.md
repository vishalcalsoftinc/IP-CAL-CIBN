
# FLOW

---

## 🛠️ Flow Explained in Simple Words

### 1. **User sends a 5G slice request**

- A user (like an operator or business system) sends a **business intent**:    
    - Example: _“I want a 5G slice with 100 Mbps bandwidth, 10 ms latency, and guaranteed QoS.”_        
- The system acknowledges: **“Got it, processing…”** (202 Accepted).

### 2. **Intent gets validated**

- The **Intent Parser** checks if the request is valid:    
    - It uses an **ontology** (a knowledge model that explains what terms mean, e.g., “bandwidth,” “latency”).        
    - It uses **SHACL rules** (validation constraints, e.g., _bandwidth must be >0_).        
- If validation **fails** → User immediately gets an error, and the request is **not stored**.    
- If validation **succeeds** → The intent is stored in the **GraphDB** with status = **Validated**.    

### 3. **Intent is decomposed into smaller pieces**

- The **Intent Analyzer/Decomposer** picks up the validated intent.    
- It breaks it down into **sub-intents**:    
    - SubIntent1 → RAN slice (radio access part, 100 Mbps)        
    - SubIntent2 → Core slice (core network, latency 10 ms)        
    - SubIntent3 → QoS policy (guaranteed service level)        
- Each sub-intent is stored in the database with status = **Processing**.    

### 4. **Orchestration of sub-intents**

- The **Intent Orchestrator** takes each sub-intent and provisions it:    
    - It talks to **ONAP NSMF** (Network Slice Management Function).        
    - Example: sends an API request → “Create a RAN slice with 100 Mbps.”        
- The orchestrator updates status:    
    - If provisioning **works** → status = **Fulfilled**.        
    - If provisioning **fails** → status = **Failed**.        

### 5. **Parent intent status update**

- Once all sub-intents finish:    
    - If **all are successful** → Parent intent = **Fulfilled**.        
    - If **some failed** → Parent intent = **Degraded**.        
    - If **all failed** → Parent intent = **Failed**.        

### 6. **Notify the user**

- The system (Status Notifier) tells the user the **final outcome**:    
    - “Your slice request is Fulfilled.” ✅        
    - Or “Your slice request is Degraded (some parts failed).” ⚠️        
    - Or “Your slice request Failed.” ❌        

---

## 🔑 Key Ideas in This Flow

- **Validation first**: Only valid intents are stored/processed.    
- **Decomposition**: Big request → smaller actionable sub-requests.    
- **Event-driven**: Each status change (Validated → Processing → Active → Fulfilled/Failed) triggers the next step automatically.    
- **TMF Standard**: Statuses follow TM Forum’s agreed lifecycle states.    
- **User always informed**: Final result is pushed back to the user.    

---

# Progress

Collecting workspace informationFiltering to most relevant informationHere’s a detailed breakdown of what’s already implemented and what remains to be done for the intent lifecycle in your ibn-backend project, based on your codebase and the sequence diagram you provided.

## Steps Already Implemented

### 1. **POST /intent (business intent)**
- **API Layer:**  
  - `app/api/intent_routes.py`:  
    - `@intent_bp.route('/intent', methods=['POST'])` receives the request and calls `IntentService.create_intent`.
- **Validation:**  
  - `app/services/intent_service.py`:  
    - Validates input using Pydantic model (`Intent`).
    - Converts JSON to RDF using `convert_intent_json_to_rdf`.
    - Validates RDF using SHACL (`validate_rdf`).
- **Persistence:**  
  - Stores the validated intent in the SQL DB (`IntentDB`).
  - Stores the RDF in GraphDB (`store_rdf_to_graphdb`).
- **Response:**  
  - Returns `202 Accepted` or validation errors.

### 2. GET /intent & GET /intent/id
- **API Layer:**  
  - Implemented in `intent_routes.py`.
- **Service Layer:**  
  - Fetches from SQL DB and/or GraphDB.

### 3. **PATCH /intent/id (Update intent)**
- **API Layer:**  
  - Implemented in `intent_routes.py`.
- **Service Layer:**  
  - Merges updates, validates, updates RDF, and persists changes.

### 4. **DELETE /intent/id**
- **API Layer:**  
  - Implemented in `intent_routes.py`.
- **Service Layer:**  
  - Deletes from SQL DB and GraphDB.

---

## Steps To Be Done

### 5. **Intent Decomposition and Subintent Handling**
- **IntentAnalyzer/Decomposer:**  
  - **Not yet implemented:**  
    - Logic to decompose a validated intent into subintents.
    - Store subintents (with parentIntentId) in SQL DB and GraphDB.
    - Update status of subintents and parent intent as per processing.

### 6. **IntentOrchestrator and Downstream Processing**
- **IntentOrchestrator:**  
  - **Not yet implemented:**  
    - Orchestrate subintent execution.
    - PATCH subintent status to "Active", "Fulfilled", "Failed", etc.
    - POST to downstream systems (e.g., ONAP NSMF).

### 7. **Status Notification**
- **StatusNotifier:**  
  - **Not yet implemented:**  
    - Notify external systems or users of status changes.
    - Event-driven updates when intent/subintent status changes.

### 8. **Advanced Status Handling**
- **TMF Statuses:**  
  - **Partial:**  
    - Only basic statuses ("acknowledged", "updated") are handled.
    - Need to implement full TMF status lifecycle: "Validated", "Processing", "Active", "Fulfilled", "Failed", "Degraded", etc.
    - Aggregate subintent statuses to update parent intent status.

### 9. **Error Handling and Rollback**
- **Atomicity:**  
  - **Partial:**  
    - Rollback SQL if GraphDB fails (optional, not fully implemented).
    - Need robust error handling and rollback for all steps.

### 10. **Testing**
- **Unit/Integration Tests:**  
  - **Partial:**  
    - Some tests exist in `tests/test_json_to_rdf.py`.
    - Need comprehensive tests for decomposition, orchestration, and status flows.

---

## Summary Table

| Step                                 | Status         | File(s) / Area                                      |
|---------------------------------------|---------------|-----------------------------------------------------|
| POST /intent (validation, store)      | Done          | `intent_routes.py`, `intent_service.py` |
| GET /intent, GET /intent/<id>         | Done          | `intent_routes.py` |
| PATCH /intent/<id>                    | Done          | `intent_routes.py` |
| DELETE /intent/<id>                   | Done          | `intent_routes.py` |
| Intent decomposition (subintents)     | To be done    | New service/module needed                           |
| Orchestration & downstream POST       | To be done    | New service/module needed                           |
| Status notification                   | To be done    | New service/module needed                           |
| Full TMF status lifecycle             | Partial       | `intent_service.py` |
| Error handling/rollback               | Partial       | `intent_service.py` |
| Comprehensive testing                 | Partial       | `tests/`                      |

---

## Next Steps (To Do)

1. **Implement Intent Decomposition:**
   - Add a service/class to split intents into subintents.
   - Store subintents with parentIntentId.

2. **Implement Orchestration:**
   - Add orchestration logic to process subintents and update statuses.
   - Integrate with downstream systems (e.g., ONAP NSMF).

3. **Implement Status Notification:**
   - Add event-driven notifications for status changes.

4. **Enhance Status Handling:**
   - Implement full TMF status lifecycle and aggregation logic.

5. **Improve Error Handling:**
   - Ensure atomicity and rollback across all persistence layers.

6. **Expand Testing:**
   - Add tests for decomposition, orchestration, and status flows.


---

# Explaination

Here’s a **step-by-step explanation** of each process in your 5G slice intent flow, showing which files and code implement each part, in the order data moves through your system.

---

## 1. **User Sends a 5G Slice Request**

**Process:**  
A user (operator/system) sends a POST request with intent JSON.

**Code & Files:**
- **API Endpoint:**  
  intent_routes.py  
  ```python
  @intent_bp.route('/intent', methods=['POST'])
  def create_intent():
      result = intent_service.create_intent(request.json)
      return jsonify(result), 202
  ```
- **Entrypoint:**  
  run.py runs the Flask app and registers the blueprint.

---

## 2. **Intent Gets Validated**

### a. **Pydantic Validation (JSON Schema)**
**Process:**  
Checks JSON structure, required fields, and value constraints.

**Code & Files:**
- **Model:**  
  intent_model.py  
  ```python
  class Intent(BaseModel):
      ...
      @validator('characteristic')
      def check_required_characteristics(cls, char_list):
          ...
  ```
- **Service:**  
  intent_service.py  
  ```python
  validated = Intent(**data)
  ```

### b. **Convert JSON to RDF**
**Process:**  
Transforms validated JSON into RDF triples for semantic validation/storage.

**Code & Files:**
- **Utility:**  
  json_to_rdf.py  
  ```python
  rdf_graph = convert_intent_json_to_rdf(data, intent_id)
  ```

### c. **SHACL Validation**
**Process:**  
Validates RDF against SHACL rules (ontology constraints).

**Code & Files:**
- **Validator:**  
  shacl_validator.py  
  ```python
  conforms, report_text = validate_rdf(rdf_graph)
  ```
- **SHACL Shapes:**  
  shacl_shapes.ttl

---

## 3. **Store Validated Intent**

### a. **Store in SQL Database**
**Process:**  
Stores intent metadata for audit/logging.

**Code & Files:**
- **Model:**  
  intent_sql_model.py  
- **Service:**  
  intent_service.py  
  ```python
  intent_db = IntentDB(...)
  self.db.add(intent_db)
  self.db.commit()
  ```

### b. **Store in GraphDB (RDF)**
**Process:**  
Stores the RDF triples in a triplestore for semantic querying.

**Code & Files:**
- **Utility:**  
  rdf_store.py  
  ```python
  store_rdf_to_graphdb(rdf_graph)
  ```

---

## 4. **Querying and Retrieving Intents**

**Process:**  
User or system can GET all or specific intents.

**Code & Files:**
- **API Endpoints:**  
  intent_routes.py  
  - `/intent` (GET all)
  - `/intent/<id>` (GET by ID)
- **Service:**  
  intent_service.py  
  - `get_all_intents()`
  - `get_intent_by_id()`
- **GraphDB Query:**  
  graphdb_reader.py  
  - `get_rdf_by_intent_id()`
  - `get_all_rdf_intents()`
- **Serialization:**  
  serializer.py  
  - `serialize_rdf_graph()`
  - `serialize_rdf_graph_list()`

---

## 5. **Update or Delete Intents**

**Process:**  
Allows PATCH (update) or DELETE on intents.

**Code & Files:**
- **API Endpoints:**  
  intent_routes.py
- **Service:**  
  intent_service.py  
  - `update_intent()`
  - `delete_intent()`
- **SPARQL Templates:**  
  sparql_templates.py  
  - `delete_intent_query()`
- **GraphDB Query:**  
  rdf_query.py  
  - `run_update_query()`

---

## 6. **Status Handling**

**Process:**  
Status is set to `"acknowledged"` on creation.  
No further status transitions (e.g., `"processing"`, `"fulfilled"`, `"failed"`) or sub-intent logic yet.

**Code & Files:**
- **Status Field:**  
  intent_sql_model.py  
  - `status = Column(String, default="acknowledged")`
- **SHACL Shape:**  
  shacl_shapes.ttl  
  - Restricts allowed status values.

---

## **Summary Table**

| Step                        | Code/Files                                                                                 | Implemented? |
|-----------------------------|-------------------------------------------------------------------------------------------|--------------|
| API receives request        | intent_routes.py, run.py                | ✔️           |
| Pydantic validation         | intent_model.py, `intent_service.py` | ✔️           |
| JSON→RDF conversion         | json_to_rdf.py                                    | ✔️           |
| SHACL validation            | shacl_validator.py, `shacl_shapes.ttl` | ✔️           |
| Store in SQL DB             | intent_sql_model.py, `intent_service.py` | ✔️           |
| Store in GraphDB            | rdf_store.py                                   | ✔️           |
| Query/serialize intents     | graphdb_reader.py, serializer.py | ✔️           |
| Update/delete intent        | intent_service.py, sparql_templates.py | ✔️           |
| Sub-intent decomposition    | *Not implemented*                                                                         | ❌           |
| Orchestration/status update | *Not implemented*                                                                         | ❌           |
| Async notification          | *Not implemented*                                                                         | ❌           |

---

## **Data Flow Recap**

1. **POST /intent** → Flask API → Pydantic validation → JSON→RDF → SHACL validation  
2. **If valid:** Store in SQL + GraphDB  
3. **GET/PATCH/DELETE** → API → Service → GraphDB/SQL  
4. **Status**: Only `"acknowledged"` is set; no further automation yet.

---

**In summary:**  
Your backend currently covers API, validation, RDF/SHACL, and storage.  
**Decomposition, orchestration, sub-intents, and advanced status flows are not yet implemented.**