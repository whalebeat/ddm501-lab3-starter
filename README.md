# Lab 3: Testing & CI/CD for ML Systems


## Overview

Comprehensive testing strategies and CI/CD pipelines for the movie rating prediction system, ensuring quality through automated verification at every layer of the stack.

**Course:** DDM501 - AI in Production: From Models to Systems  
**Weight:** 15% of total grade  
**Duration:** 3 hours (in-class) + 1 week to complete  
**Prerequisites:** Lab 1 and Lab 2 completed

> 📄 See Testing Strategy.pdf for the full testing strategy, fixture reference, and coverage gap analysis.

## Learning Objectives

- Write comprehensive unit tests for ML components
- Implement integration tests for API endpoints
- Create data validation tests
- Design model behavioral tests (invariance, directional, minimum functionality)
- Set up CI/CD pipelines with GitHub Actions
- Implement automated code quality checks

## Project Structure

```
ddm501-lab3-starter/
├── app/
│   ├── __init__.py
│   ├── main.py             # FastAPI application
│   ├── model.py            # ML model class
│   ├── schemas.py          # Pydantic schemas
│   └── config.py           # Configuration
├── tests/
│   ├── __init__.py
│   ├── conftest.py         # Shared fixtures
│   ├── unit/
│   │   ├── __init__.py
│   │   ├── test_model.py   # Model unit tests (TODO)
│   │   └── test_schemas.py # Schema tests (TODO)
│   ├── integration/
│   │   ├── __init__.py
│   │   └── test_api.py     # API tests (TODO)
│   ├── data/
│   │   ├── __init__.py
│   │   └── test_data_quality.py  # Data tests (TODO)
│   └── model/
│       ├── __init__.py
│       └── test_model_behavior.py  # Behavioral tests (TODO)
├── .github/
│   └── workflows/
│       ├── ci.yml          # CI pipeline (TODO)
│       └── cd.yml          # CD pipeline (TODO)
├── scripts/
│   └── train_model.py      # Model training script
├── models/                 # Saved models
├── .pre-commit-config.yaml # Pre-commit hooks (TODO)
├── pyproject.toml          # Project configuration
├── requirements.txt
├── requirements-dev.txt    # Development dependencies
├── Dockerfile
└── README.md
```

## Quick Start

### 1. Clone and Setup

```bash
git clone https://github.com/[your-repo]/ddm501-lab3-starter.git
cd ddm501-lab3-starter

# Create virtual environment
python -m venv venv
source venv/bin/activate  # Linux/Mac

# Install dependencies
pip install -r requirements.txt
pip install -r requirements-dev.txt
```

### 2. Train Model (if not exists)

```bash
python scripts/train_model.py
```

### 3. Run Tests

```bash
# Run all tests
pytest tests/ -v

# Run with coverage
pytest tests/ -v --cov=app --cov-report=html

# Run specific test category
pytest tests/unit/ -v
pytest tests/integration/ -v
pytest tests/data/ -v
pytest tests/model/ -v
```

### 4. Code Quality Checks

```bash
# Install pre-commit hooks
pip install pre-commit
pre-commit install

# Run all checks manually
pre-commit run --all-files

# Individual tools
black app/ tests/
flake8 app/ tests/
mypy app/
```

### 5. Run the API

```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

## Status

### Test Files
- [x] `tests/unit/test_model.py` - Unit tests for model class
- [x] `tests/unit/test_schemas.py` - Schema validation tests
- [x] `tests/integration/test_api.py` - API endpoint tests
- [x] `tests/data/test_data_quality.py` - Data quality tests
- [x] `tests/model/test_model_behavior.py` - Behavioral tests

### CI/CD Files
- [x] `.github/workflows/ci.yml` - CI pipeline
- [x] `.github/workflows/cd.yml` - CD pipeline (BONUS)
- [x] `.pre-commit-config.yaml` - Pre-commit hooks

## Coverage Report

Latest run: **89 passed, 0 failed** in 10.87 s

```
Name              Stmts   Miss  Cover
--------------------------------------
app\__init__.py       1      0   100%
app\config.py        13      0   100%
app\main.py          50     10    80%
app\model.py         42      9    79%
app\schemas.py       30      1    97%
--------------------------------------
TOTAL               136     20    85%
```

Run locally:
```bash
pytest tests/ -v --cov=app --cov-report=html
# open htmlcov/index.html
```

---

## Test Types

### Unit Tests
Test individual functions and classes in isolation.

```python
def test_model_loads_successfully(model):
    assert model.is_loaded()
```

### Integration Tests
Test component interactions and API endpoints.

```python
def test_predict_valid_request(test_client):
    response = test_client.post("/predict", json={"user_id": "196", "movie_id": "242"})
    assert response.status_code == 200
```

### Data Tests
Validate data quality and schema.

```python
def test_ratings_in_valid_range(sample_ratings):
    for r in sample_ratings:
        assert 1.0 <= r["rating"] <= 5.0
```

### Behavioral Tests
Test model behavior patterns.

```python
def test_same_input_same_output(model):
    result1 = model.predict("196", "242")
    result2 = model.predict("196", "242")
    assert result1 == result2
```

## CI/CD Pipeline

### Continuous Integration (`.github/workflows/ci.yml`)
- Triggers on every push and pull request to `main` / `develop`
- Steps: `black --check` → `flake8` → `isort --check` → `mypy` → `pytest --cov`
- Fails the build if coverage drops below **80%**

### Continuous Deployment (`.github/workflows/cd.yml`)
- Triggered by pushing a semver version tag (e.g. `v1.2.3`)
- Flow: test → build & push Docker image → GitHub Release → deploy staging → deploy production

#### How to trigger a release

```bash
# 1. Make sure all changes are committed and pushed
git add .
git commit -m "chore: prepare release v1.0.0"
git push origin main

# 2. Create and push a version tag  (vMAJOR.MINOR.PATCH)
git tag v1.0.0
git push origin v1.0.0
```

This single `git push origin v1.0.0` kicks off the full CD pipeline:

```
push tag v1.0.0
    │
    ▼
[test]  ← full pytest suite, fails if coverage < 80%
    │
    ▼
[build-and-push]  ← Docker image tagged v1.0.0, 1.0, latest
    │
    ├─────────────────────────────┐
    ▼                             ▼
[deploy-staging]          [create-release]
    │                      GitHub Release with
    │                      auto-generated notes
    ▼
[deploy-production]  ← requires manual approval
```

**Pre-release tags** (containing `-rc`, `-beta`, or `-alpha`) are automatically marked as GitHub pre-releases:

```bash
git tag v1.1.0-rc1
git push origin v1.1.0-rc1
```

**Required GitHub secrets** (Settings → Secrets → Actions):

| Secret | Purpose |
|--------|---------|
| `DOCKER_USERNAME` | Docker Hub login |
| `DOCKER_PASSWORD` | Docker Hub token |

**Required GitHub environments** (Settings → Environments):

| Environment | Protection |
|-------------|-----------|
| `staging` | Optional: add required reviewers |
| `production` | Recommended: require manual approval |

## Grading Rubric

| Criteria | Weight |
|----------|--------|
| Test Coverage (unit, integration, data, model) | 30% |
| CI/CD Pipeline | 30% |
| Code Quality | 20% |
| Documentation | 20% |

**Minimum Requirements:**
- 80% code coverage
- All CI checks passing
- Pre-commit hooks configured

## Resources

- [pytest Documentation](https://docs.pytest.org/)
- [GitHub Actions](https://docs.github.com/en/actions)
- [pre-commit](https://pre-commit.com/)
- [Black](https://black.readthedocs.io/)
- [Flake8](https://flake8.pycqa.org/)
- [mypy](https://mypy.readthedocs.io/)

## Submission

1. ✅ All TODO items complete
2. ✅ All 89 tests pass
3. ✅ 85% coverage (target: ≥ 80%)
4. ✅ Push to GitHub with CI badge
5. Submit repository link via LMS

## License

MIT License - For educational purposes only.
