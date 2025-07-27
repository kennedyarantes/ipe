# IPE File Format Specification

## Overview

The **IPE file format** is designed for **secure, lightweight and flexible document exchange** between systems, focusing on scenarios where documents are already issued but need to be transferred, validated, and audited.

The format is divided into two components:

1. **`.ipe`** – The actual document data.
2. **`pass.<type>.<name>.pass`** – The contract file describing structure, schema, and rendering templates.

The format supports **flexibility in business validation**: even if a document has business rule errors (e.g., invalid date ranges), it is still accepted for transfer but flagged accordingly.

---

## 1. IPE Document Structure (`.ipe`)

### Fields

1. **`version`** *(string | number)*
   Indicates the version of the IPE file format.
   *Example:* `"2.0"`

2. **`type`** *(string)*
   Document category.
   *Values:* `"document"`, `"certificate"`, `"authorization"`, `"declaration"`.
   *Example:* `"document"`

3. **`pass`** *(string)*
   Identifies which **contract file** (`pass.<type>.<name>.pass`) defines the structure of `content`.
   *Example:* `"GTA"`

4. **`status`** *(string)*
   Indicates the document state: `"active"`, `"cancelled"`, `"expired"`, `"invalid"`.
   *Used when business validation fails but document transfer is still accepted.*
   *Example:* `"active"`

5. **`validity`** *(string)* *(optional)*
   Marks whether business validations passed or failed: `"valid"`, `"invalid"`.
   *Example:* `"invalid"`

6. **`content`** *(array | string)*
   Holds actual data fields, **without labels** (labels are in the `.pass` file).
   If encrypted, this may be replaced by `content_encrypted`.
   *Example:*

   ```json
   "content": [
     "Farm ABC",
     "Slaughterhouse XYZ",
     "2025-07-30T00:00:00Z"
   ]
   ```

7. **`encryption_key`** *(string, Base64, optional)*
   Indicates encryption key used when `content` is encrypted.

8. **`signature`** *(string, Base64)*
   Digital signature and optionally embedded public certificate.

9. **`created_at`** *(string, ISO 8601)*
   Document creation timestamp.
   *Example:* `"2025-07-26T13:45:30Z"`

10. **`modified_at`** *(string, ISO 8601, optional)*
    Last modification timestamp.
    *Example:* `"2025-07-26T15:10:00Z"`

11. **`modified_by`** *(string, optional)*
    Identifies the agent or system that modified the document.
    *Example:* `"Inspector-12345"`

12. **`hash`** *(string)*
    Integrity hash of the document content.
    *Example:* `"ae3f0a34e2d45bb4c1e23b..."`

13. **`url_validation`** *(string, URL)*
    Endpoint to validate document authenticity.
    *Example:* `"https://validation.ipeweb.org/doc/UUID"`

---

### Example `.ipe` file

```json
{
  "version": "2.0",
  "type": "document",
  "pass": "GTA",
  "status": "active",
  "validity": "invalid",
  "content": ["Farm ABC", "Slaughterhouse XYZ", "2025-07-30T00:00:00Z"],
  "signature": "QmFzZTY0U2lnbmF0dXJl...",
  "created_at": "2025-07-26T13:45:30Z",
  "modified_at": "2025-07-26T15:10:00Z",
  "modified_by": "Inspector-12345",
  "hash": "ae3f0a34e2d45bb4c1e23b...",
  "url_validation": "https://validation.ipeweb.org/doc/UUID"
}
```

---

## 2. Contract File Structure (`pass.<type>.<name>.pass`)

### Fields

1. **`schema_version`** *(string)*
   Version of the contract file format.

2. **`type`** *(string)*
   Document category (same as in `.ipe`).

3. **`pass`** *(string)*
   Unique identifier of the document type.

4. **`fields`** *(array)*
   Ordered list of fields defining content layout.
   *Example:*

   ```json
   "fields": [
     {"label": "Emitter", "type": "string", "required": true},
     {"label": "Destination", "type": "string", "required": true},
     {"label": "Validity", "type": "timestamp", "required": true}
   ]
   ```

5. **`json_schema`** *(object)*
   Defines data types and constraints for `content`.
   *Example:*

   ```json
   "json_schema": {
     "type": "array",
     "items": [
       {"type": "string"},
       {"type": "string"},
       {"type": "string", "format": "date-time"}
     ],
     "minItems": 3,
     "maxItems": 3
   }
   ```

6. **`template`** *(object)*
   Defines rendering for visualization. Supports **Handlebars** and **Mustache**.
   *Example:*

   ```json
   "template": {
     "engine": "handlebars",
     "layout": "<div><h1>GTA</h1><p>Emitter: {{0}}</p><p>Destination: {{1}}</p><p>Validity: {{2}}</p></div>"
   }
   ```

7. **`contract_version`** *(string)*
   Ensures compatibility between `.ipe` and `.pass` files.

8. **`available`** *(boolean)* *(optional)*
   Indicates if the server currently has the contract available.

---

### Example Contract File

```json
{
  "schema_version": "1.0",
  "type": "document",
  "pass": "GTA",
  "fields": [
    {"label": "Emitter", "type": "string", "required": true},
    {"label": "Destination", "type": "string", "required": true},
    {"label": "Validity", "type": "timestamp", "required": true}
  ],
  "json_schema": {
    "type": "array",
    "items": [
      {"type": "string"},
      {"type": "string"},
      {"type": "string", "format": "date-time"}
    ],
    "minItems": 3,
    "maxItems": 3
  },
  "template": {
    "engine": "handlebars",
    "layout": "<div><h1>GTA</h1><p>Emitter: {{0}}</p><p>Destination: {{1}}</p><p>Validity: {{2}}</p></div>"
  },
  "contract_version": "1.2",
  "available": true
}
```

---

## 3. Visualization

### Supported engines

* **Handlebars**: Simple, widely used, good for JSON-based rendering.
* **Mustache**: Minimalistic, no logic, easy to embed in APIs.

Both engines allow a clear separation between **data** and **view**, making it easier to update layouts without touching the underlying data.

---

## 4. Business Validation vs Schema Validation

* **Schema validation**: ensures data types are correct (e.g., timestamp format). Failure = reject.
* **Business validation**: checks business logic (e.g., validity date > creation date). Failure = accept but flag:

  * `status: "active"`, `validity: "invalid"`.

---

## The `.ipe` File Format Purpose

The **`.ipe`** file format was created to **standardize and optimize the transmission of entities** between systems.
In many corporate and government scenarios, digital documents are transmitted repeatedly, often carrying redundant structural data, unnecessary fields, and information repeated on every transfer.
The `.ipe` format addresses this problem by adopting a **minimalistic and efficient approach**, transferring **only the essential content**.

### Standardized Structure

* Uses a contract file (`pass.<type>.<name>.pass`) to define fields, their order, and data types.
* Eliminates ambiguity in data consumption and improves interoperability.

### Transmission Optimization

* Reduces payload size by removing redundant labels.
* Ensures every data array position corresponds to a predefined field.
* Supports native compression (Gzip) for additional size reduction.

### Perfect for RESTful APIs

* Simple JSON arrays, ideal for quick parsing and integration.
* Uses explicit versioning to maintain compatibility across clients and servers.

### Validation and Flexibility

* Separates **schema validation** (structure and data types) from **business validation** (rules such as date ranges).
* Even if business validation fails, documents are accepted but flagged as `validity: "invalid"`.

### QR Code Integration

* Minimal required information can be embedded into QR Codes.
* Allows offline validation and quick document retrieval using a simple scan.


---

This ensures **all documents are transferable**, even when there are downstream business issues.

---

"Built with no love, just beer. – Ipe Web Team"
