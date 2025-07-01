This page outlines a scenario demonstrating the use of the **API Exchange IG** and **Structured Regulatory Correspondence IG** for a Type II variation submission to extend the shelf life of a biologic product.

### Scenario Overview

This scenario is for a Type II variation to extend the shelf life of a biologic product from 24 to 36 months. The variation requires updates to labeling (ePI) and pharmaceutical quality (PQI) content. The submission, or `Transaction Bundle`, includes a `Task`, a `DocumentReference` with base64-encoded attachments (application form as a Collection Bundle, ePI Document Bundle, PQI Collection Bundle), and a `Provenance`. The EMA processes the `Transaction Bundle`, sends a `Transaction Bundle` back which includes a LoQ with 10 questions (2 administrative, 3 labeling, 5 CMC) as a `Questionnaire`. PharmaInc responds with a `Transaction Bundle` that includes a `QuestionnaireResponse` (including an SVG image and multi-paragraph text for specific answers). Finally, the EMA issues a Decision Letter `Document Bundle` within a final 'Transaction Bundle'. All exchanges use RESTful FHIR APIs per the API Exchange IG.

**Figure 1: Flow diagram showing the 5 Step exchange between PharmaInc and the EMA, conducted via FHIR Transaction Bundles over RESTful APIs.**

<span style="display: inline-block; vertical-align: middle;">
  <img src="shelflifeextensionflow.svg" alt="5 step process for a shelf life extension" style="width: 800px; height: auto;" />
</span>

### Step 1: PharmaInc Submits Type II Variation

PharmaInc submits a Type II variation to extend the shelf life from 24 to 36 months via a Transaction Bundle to the EMA’s FHIR server (`https://ema.example.org/fhir`). The Bundle includes:
- **Task**: Requests the variation, with PharmaInc as requester and EMA as owner.
- **DocumentReference**: References the application form (Collection Bundle), ePI Document Bundle (updated storage conditions), and PQI Collection Bundle (stability data) as base64-encoded attachments.
- **Provenance**: Records the submission’s origin.

Refer to [Type II Variation](https://build.fhir.org/ig/cander2/apix-ig/Bundle-Type-II-Variation.html) for an example of the Transaction Bundle from the Company to the Regulator which contains the original submission.

### Step 2: EMA Receives and Processes the Submission

The EMA’s FHIR server receives the Transaction Bundle at `https://ema.example.org/fhir`, validates it per the API Exchange IG, and processes the resources:
- **Task**: Logged as a variation request for `MedicinalProductDefinition/mpd-001`.
- **DocumentReference**: Base64-encoded attachments are decoded to access the application form (Collection Bundle), ePI Document Bundle, and PQI Collection Bundle.
- **Provenance**: Verified for integrity and origin from PharmaInc.

The EMA prepares to send a List of Questions (LoQ). No new FHIR resources are created in this step.

### Step 3: EMA Sends List of Questions

The EMA compiles a LoQ with 10 questions (2 administrative, 3 labeling, 5 CMC) in a `Questionnaire` resource within a Collection Bundle. The Transaction Bundle includes a `Task` (requesting responses), the `Questionnaire`, and a `Provenance`, sent to PharmaInc’s FHIR server (`https://pharma-inc.example.org/fhir`).

**Questions**:
- **Administrative**:
  1. Is the fee for the Type II variation fully paid and documented?
  2. Are all required metadata fields in the application form complete?
- **Labeling**:
  1. **Storage Instructions**: Do the updated storage conditions (2-8°C for 36 months) require additional patient guidance?
  2. **Patient Information**: Is the patient leaflet revised to clarify the extended shelf life?
  3. **Label Clarity**: Is the updated ePI text sufficiently clear for all EU languages?
- **CMC**:
  1. **Stability Data**: Do stability studies cover all required environmental conditions for 36 months?
  2. **Analytical Methods**: Are analytical methods validated for detecting degradation over 36 months?
  3. **Impurities**: Are new impurities identified in extended stability studies?
  4. **Container Closure System**: Is the container closure system validated for 36 months?
  5. **Degradation Products**: Are degradation products fully characterized for 36 months?

Refer to [List of Questions](https://build.fhir.org/ig/cander2/apix-ig/Bundle-List-Of-Questions.html) for an example of the Transaction Bundle from the Regulator to the Company which contains the List of Questions. 

### Step 4: PharmaInc Responds to List of Questions

PharmaInc’s FHIR server processes the LoQ, prepares a `QuestionnaireResponse` with answers to all 10 questions, and sends a Transaction Bundle to the EMA’s FHIR server (`https://ema.example.org/fhir`). For `cmc-4` (container closure system), the response includes a base64-encoded SVG image. For `cmc-5` (degradation products), the response includes two paragraphs. The Bundle includes a `Task` (indicating completion), the `QuestionnaireResponse`, and a `Provenance`.

**Responses**:
- **Administrative**:
  1. Fee paid on June 25, 2025, with documentation.
  2. All metadata fields are complete per EMA guidelines.
- **Labeling**:
  1. No additional patient guidance needed for storage conditions.
  2. Patient leaflet updated for 36-month shelf life.
  3. ePI text translated and clear in all EU languages.
- **CMC**:
  1. Stability studies cover all environmental conditions per ICH guidelines.
  2. Analytical methods validated, with results in PQI.
  3. No new impurities identified.
  4. Container closure system validated, with SVG diagram.
  5. Degradation products characterized, with no significant changes (two paragraphs).

Refer to [List of Questions Response](https://build.fhir.org/ig/cander2/apix-ig/Bundle-List-Of-Questions-Response.html) for an example of the Transaction Bundle from the Company to the Regulator which contains a response to the List of Questions. 

### Step 5: EMA Issues Decision Letter

The EMA reviews PharmaInc’s LoQ responses and approves the variation. The Decision Letter is a Document Bundle with a `Composition`, encoded as base64 data in a `DocumentReference`. The Transaction Bundle includes a `Task`, the `DocumentReference`, and a `Provenance`, sent to PharmaInc’s FHIR server (`https://pharma-inc.example.org/fhir`).

Refer to [Decision Letter](https://build.fhir.org/ig/cander2/apix-ig/Bundle-Decision-Letter.html) for an example of the Transaction Bundle from the Regulator to the Company which contains the Decision Letter. 