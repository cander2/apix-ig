### Validation
- Use a JSON linter in VS Code.
- Validate with HAPI FHIR Validator CLI:
  ```bash
  java -jar hapi-fhir-cli.jar validate --file input/examples/Bundle-transaction-epi-bundle.json --profile input/profiles/StructureDefinition-RegulatoryTask.json --version R5
  ```
- Or use [validator.fhir.org](https://validator.fhir.org) with R5.