1. Map current pipeline stages and timings

Current Pipeline Timings (baseline)
- Checkout code:        5s
- Install dependencies: 90s
- Lint:                 20s
- Unit tests:            45s
- Build Docker image:    60s
- Integration tests:     80s
------------------------------
Total:                  300s (5 min)

2. Add dependency/layer caching
- uses: docker/build-push-action@v5
    with:
      cache-from: type=gha
      cache-to: type=gha,mode=max

3. Parallelize independent jobs

jobs:
  lint:
    runs-on: ubuntu-latest
    steps: [...]

  unit-tests:
    runs-on: ubuntu-latest
    steps: [...]

  build:
    runs-on: ubuntu-latest
    steps: [...]
integration-tests:
    needs: [build]   

4. Order checks fail-fast (cheap/fast first)

1. Lint        (~5-20s, catches syntax/style issues)
2. Unit tests   (~30-60s, catches logic bugs)
3. Build        (~60s+, catches packaging issues)
4. Integration tests (~80s+, slowest, catches system-level issues)

5. Identify and fix flaky steps

 Flaky steps found
- "user-login" integration test failed intermittently due to a race 
  condition in test DB setup. Fixed by waiting for DB readiness 
  before running tests.

6. Re-measure and document the speed-up

 Pipeline Speed-up Results

| Stage              | Before | After | 
|--------------------|--------|-------|
| Install deps       | 90s    | 8s (cached) |
| Lint + tests + build | 125s (sequential) | 60s (parallel) |
| Flaky reruns needed | ~1 in 5 runs | 0 |
| **Total pipeline**  | **300s** | **~95s** |

Result: ~68% faster, and no more flaky reruns needed.

<img width="1536" height="1024" alt="ss3344" src="https://github.com/user-attachments/assets/3a13d5b4-c6c4-41ce-9a59-f79e08818376" />
