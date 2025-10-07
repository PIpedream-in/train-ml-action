# Pipedream Train Ml Action

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-train-ml-blue.svg?colorA=24292e&colorB=0366d6&style=flat&longCache=true&logo=github)](https://github.com/marketplace/actions/train-ml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub release](https://img.shields.io/github/release/PIpedream-in/train-ml-action.svg)](https://github.com/PIpedream-in/train-ml-action/releases)


Train machine learning models on AWS SageMaker with zero infrastructure setup.

## Quick Start

```yaml
- uses: pipedream/train-ml@v1
  with:
    api-key: ${{ secrets.PDTRAIN_API_KEY }}
    code-path: ./training
    dataset: mnist:latest
    framework: pytorch
    entry-point: train.py
```

---

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Usage](#usage)
  - [Script Mode](#script-mode-pytorch-tensorflow-sklearn-xgboost)
  - [Docker Mode](#docker-mode-custom-containers)
- [Inputs](#inputs)
- [Outputs](#outputs)
- [Examples](#examples)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

---

## Overview

This GitHub Action integrates with the Pipedream ML Orchestrator to train models on AWS SageMaker. It supports:

- ✅ **Script Mode:** PyTorch, TensorFlow, scikit-learn, XGBoost, Hugging Face
- ✅ **Docker Mode:** Custom container images
- ✅ **Hyperparameter sweeps** via GitHub Actions matrix strategy
- ✅ **Automatic artifact management**
- ✅ **Cost tracking and reporting**
- ✅ **Real-time training logs**

---

## Prerequisites

### 1. Get API Key

Visit [https://pipedream.in/api-keys](https://pipedream.in/api-keys) to create an API key.

### 2. Add GitHub Secret

1. Go to your repository **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Name: `PDTRAIN_API_KEY`
4. Value: Your API key (e.g., `sdk_xxxxx`)

---

## Usage

### Script Mode (PyTorch, TensorFlow, sklearn, XGBoost)

Train using managed framework containers. Your code is uploaded as a bundle.

```yaml
- uses: pipedream/train-ml@v1
  with:
    api-key: ${{ secrets.PDTRAIN_API_KEY }}

    # Training code
    code-path: ./training          # Directory with train.py
    entry-point: train.py           # Your training script

    # Dataset
    dataset: ds_abc123-def456-7890  # Dataset ID from upload-dataset action

    # Framework
    framework: pytorch
    framework-version: 2.2.0
    python-version: py310

    # Hyperparameters
    hyperparameters: |
      epochs: 10
      learning_rate: 0.001
      batch_size: 32
```

### Docker Mode (Custom Containers)

Train using your own Docker image.

```yaml
- uses: pipedream/train-ml@v1
  with:
    api-key: ${{ secrets.PDTRAIN_API_KEY }}

    # Docker image (no framework needed)
    image: 123456.dkr.ecr.us-east-1.amazonaws.com/my-trainer:latest
    entry-point: train.py

    # Dataset
    dataset: custom-data:v1

    # Parameters for Docker Mode
    parameters: |
      fit_intercept: true
      normalize: false
```

---

## Inputs

### Required Inputs

| Input | Description | Example |
|-------|-------------|---------|
| `api-key` | Pipedream API key | `${{ secrets.PDTRAIN_API_KEY }}` |
| `entry-point` | Training script | `train.py` |

### Execution Mode (Choose ONE)

#### Script Mode Inputs

| Input | Required | Description | Example |
|-------|----------|-------------|---------|
| `framework` | Yes | ML framework | `pytorch`, `tensorflow`, `sklearn`, `xgboost` |
| `code-path` | Yes | Training code directory | `./training` |
| `dataset` | No | Dataset reference | `mnist:latest` or `abc-123` |
| `framework-version` | No | Framework version | PyTorch: `2.2.0`, TF: `2.13.0`, XGB: `1.7-1` |
| `python-version` | No | Python version | `py38`, `py39`, `py310`, `py311` (default: `py310`) |
| `hyperparameters` | No | Hyperparameters (YAML) | See [examples](#hyperparameters) |

#### Docker Mode Inputs

| Input | Required | Description | Example |
|-------|----------|-------------|---------|
| `image` | Yes | Docker image URI | `123456.dkr.ecr.us-east-1.amazonaws.com/trainer:latest` |
| `dataset` | No | Dataset ID | `ds_abc123-def456-7890` |
| `parameters` | No | Parameters (YAML) | See [Docker example](#docker-mode-custom-containers) |

### Workflow Behavior

| Input | Default | Description |
|-------|---------|-------------|
| `wait-for-completion` | `true` | Wait for training to complete |
| `download-artifacts` | `true` | Download artifacts after training |
| `artifacts-path` | `./artifacts` | Local path for artifacts |
| `bundle-name` | Auto-generated | Custom bundle name |
| `timeout-minutes` | `120` | GitHub Actions job timeout |

### Advanced Options

| Input | Default | Description |
|-------|---------|-------------|
| `api-url` | `https://ml-orchestrator.pipedream.in` | Orchestrator API URL |
| `exclude-patterns` | `*.pyc,__pycache__,.git,.venv` | Exclude from bundle upload |
| `max-cost-usd` | None | Fail if estimated cost exceeds this |

---

## Outputs

All outputs are available via `steps.<step-id>.outputs.<output-name>`.

| Output | Description | Example |
|--------|-------------|---------|
| `run-id` | Training run ID | `abc-123-def-456` |
| `status` | Final status | `completed`, `failed`, `stopped` |
| `cost` | Training cost in USD | `2.45` |
| `runtime-seconds` | Runtime in seconds | `1234` |
| `artifacts-path` | Local path to artifacts | `./artifacts` |
| `job-arn` | SageMaker job ARN | `arn:aws:sagemaker:...` |
| `model-url` | S3 URL to model | `s3://...` |
| `bundle-id` | Bundle ID | `bundle-xyz` |
| `dataset-id` | Dataset ID used | `dataset-abc` |

### Using Outputs

```yaml
- uses: pipedream/train-ml@v1
  id: train
  with:
    # ... inputs

- name: Show results
  run: |
    echo "Run ID: ${{ steps.train.outputs.run-id }}"
    echo "Cost: \$${{ steps.train.outputs.cost }}"
    echo "Status: ${{ steps.train.outputs.status }}"

- name: Upload artifacts
  uses: actions/upload-artifact@v4
  with:
    name: model
    path: ${{ steps.train.outputs.artifacts-path }}
```

---

## Examples

### PyTorch Training

```yaml
- uses: pipedream/train-ml@v1
  with:
    api-key: ${{ secrets.PDTRAIN_API_KEY }}
    code-path: ./training
    dataset: ds_abc123-def456-7890
    framework: pytorch
    framework-version: 2.2.0
    entry-point: train.py
    hyperparameters: |
      epochs: 2
      batch_size: 16
      learning_rate: 0.001
```

### TensorFlow Training

```yaml
- uses: pipedream/train-ml@v1
  with:
    api-key: ${{ secrets.PDTRAIN_API_KEY }}
    code-path: ./models
    dataset: ds_abc123-def456-7890
    framework: tensorflow
    framework-version: 2.13.0
    entry-point: train.py
    hyperparameters: |
      epochs: 3
      batch_size: 32
      learning_rate: 0.001
```

### scikit-learn Clustering

```yaml
- uses: pipedream/train-ml@v1
  with:
    api-key: ${{ secrets.PDTRAIN_API_KEY }}
    code-path: ./clustering
    dataset: ds_abc123-def456-7890
    framework: sklearn
    framework-version: 1.4.2
    entry-point: train.py
    hyperparameters: |
      n_clusters: 3
      max_iter: 100
```

### XGBoost Regression

```yaml
- uses: pipedream/train-ml@v1
  with:
    api-key: ${{ secrets.PDTRAIN_API_KEY }}
    code-path: ./xgboost
    dataset: ds_abc123-def456-7890
    framework: xgboost
    framework-version: 1.7-1
    entry-point: train.py
    hyperparameters: |
      num_round: 50
      max_depth: 4
      eta: 0.1
      subsample: 0.8
```

### Hyperparameter Sweep (Matrix)

```yaml
strategy:
  matrix:
    learning_rate: [0.001, 0.01, 0.1]
    batch_size: [32, 64]

steps:
  - uses: pipedream/train-ml@v1
    with:
      api-key: ${{ secrets.PDTRAIN_API_KEY }}
      code-path: ./training
      dataset: cifar10:v2
      framework: pytorch
      hyperparameters: |
        learning_rate: ${{ matrix.learning_rate }}
        batch_size: ${{ matrix.batch_size }}
        epochs: 10
      bundle-name: sweep-lr${{ matrix.learning_rate }}-bs${{ matrix.batch_size }}
```

### Custom Docker Image

```yaml
- uses: pipedream/train-ml@v1
  with:
    api-key: ${{ secrets.PDTRAIN_API_KEY }}
    image: 123456.dkr.ecr.us-east-1.amazonaws.com/my-trainer:latest
    dataset: ds_abc123-def456-7890
    entry-point: train.py
    parameters: |
      fit_intercept: true
      normalize: false
```

---

## Best Practices

### 1. Use GitHub Secrets for API Keys

Never hardcode API keys. Always use GitHub secrets:

```yaml
api-key: ${{ secrets.PDTRAIN_API_KEY }}  # ✅ Good
api-key: sdk_abc123                       # ❌ Bad
```

### 2. Use Cost Limits for Safety

```yaml
max-cost-usd: 10.0  # Fail if estimated cost > $10
```

### 3. Custom Bundle Names for Tracking

```yaml
bundle-name: prod-${{ github.ref_name }}-${{ github.sha }}
```

### 4. Use Matrix for Hyperparameter Optimization

```yaml
strategy:
  matrix:
    lr: [0.001, 0.01]
    bs: [32, 64]
```

### 5. Download Artifacts Conditionally

```yaml
download-artifacts: ${{ github.ref == 'refs/heads/main' }}
```

---

## Troubleshooting

### Error: "Must specify either 'framework' or 'image'"

**Cause:** You must choose Script Mode (`framework`) OR Docker Mode (`image`), not both.

**Solution:**
```yaml
# Script Mode
framework: pytorch
code-path: ./training

# OR Docker Mode
image: 123456.dkr.ecr.us-east-1.amazonaws.com/trainer:latest
```

### Error: "Script Mode requires 'code-path'"

**Cause:** When using `framework`, you must provide `code-path`.

**Solution:**
```yaml
framework: pytorch
code-path: ./training  # Required for Script Mode
```

### Error: "Dataset not found"

**Cause:** Dataset ID is invalid or not found.

**Solution:**
- Use dataset ID from `upload-dataset` action output
- Or get dataset ID from `pdtrain dataset list`
- Example: `ds_abc123-def456-7890`

### Artifacts not downloaded

**Cause:** Training didn't complete successfully.

**Solution:** Check training status:
```yaml
- name: Check status
  run: |
    echo "Status: ${{ steps.train.outputs.status }}"
```

Only downloads if `status == 'completed'`.

---

## Framework Versions

Based on real examples from our test suite:

| Framework | Version Examples | Python | Notes |
|-----------|-----------------|--------|-------|
| PyTorch | `2.2.0`, `2.1.0`, `2.0.1` | `py310` | Deep learning and neural networks |
| TensorFlow | `2.13.0`, `2.12.0` | `py310` | Supports Keras models |
| scikit-learn | `1.4.2`, `1.3.0` | `py310` | Classical ML algorithms |
| XGBoost | `1.7-1`, `1.6-1` | `py310` | Gradient boosting |

---

## Cost Optimization Tips

1. **Skip artifact download for PRs:**
   ```yaml
   download-artifacts: ${{ github.ref == 'refs/heads/main' }}
   ```

2. **Use cost limits:**
   ```yaml
   max-cost-usd: 5.0  # Fail if > $5
   ```

3. **Check quota before expensive training:**
   ```yaml
   - uses: pipedream/check-quota@v1
     with:
       api-key: ${{ secrets.PDTRAIN_API_KEY }}
       min-wallet-balance: 50.0
   ```

---

## Complete Workflow Example

```yaml
name: Train Model

on:
  push:
    branches: [main]

jobs:
  train:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5

      - uses: pipedream/train-ml@v1
        id: train
        with:
          api-key: ${{ secrets.PDTRAIN_API_KEY }}
          code-path: ./training
          dataset: ds_abc123-def456-7890
          framework: pytorch
          framework-version: 2.2.0
          entry-point: train.py
          hyperparameters: |
            epochs: 10
            learning_rate: 0.001
            batch_size: 32
          max-cost-usd: 10.0

      - name: Upload model
        if: steps.train.outputs.status == 'completed'
        uses: actions/upload-artifact@v4
        with:
          name: trained-model
          path: ${{ steps.train.outputs.artifacts-path }}

      - name: Create summary
        run: |
          echo "## Training Results" >> $GITHUB_STEP_SUMMARY
          echo "- Run ID: \`${{ steps.train.outputs.run-id }}\`" >> $GITHUB_STEP_SUMMARY
          echo "- Cost: \$${{ steps.train.outputs.cost }}" >> $GITHUB_STEP_SUMMARY
          echo "- Runtime: ${{ steps.train.outputs.runtime-seconds }}s" >> $GITHUB_STEP_SUMMARY
```

---

## Support

- **Documentation:** [https://docs.pipedream.in](https://docs.pipedream.in)
- **Examples:** [See examples/](../examples/)
- **Issues:** [GitHub Issues](https://github.com/pipedream/actions/issues)
- **Email:** [hello@pipedream.in](mailto:hello@pipedream.in)

---

## License

MIT License - see [LICENSE](../LICENSE) for details.
