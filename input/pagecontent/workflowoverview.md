### Workflow
1. **Submission**: Submit a `Transaction Bundle` with a `Task` and `DocumentReference` (for ePI/PDF) or `Observation` (for PQI) to `[base]`.
2. **Processing**: Regulator retrieves `Task` and resources via `GET [base]/Task?status=requested&_include=Task:focus`.
3. **Updates**: Regulator updates `Task` status and adds `Task.output` via a `Transaction Bundle`.
4. **Notification**: Submitter polls or subscribes for `Task` updates.