# Synthetic Data Generator

Generate synthetic clinical notes and PHI manifests for evaluating deidentification.

Install and run the generator from the parent `tooling/` directory:

```bash
uv sync --all-packages --locked
uv run --package cli generate-notes --type emergency_dept --count 1
```

The generator uses Amazon Bedrock with the terminal's configured AWS credentials.
Notes and ground truth manifests are written to `data/synthetic_data/`.
See [the tooling guide](../README.md) for the complete workflow.
