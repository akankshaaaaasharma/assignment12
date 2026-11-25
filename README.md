## Issues Encountered and Fixes

**1. Dependencies failed to install (Python 3.14)**
My virtual environment was on Python 3.14, and pinned packages like `greenlet` had no prebuilt wheels, so pip tried to compile them and hung. I recreated the venv with Python 3.12, and the install worked with the original pinned versions.

**2. `ModuleNotFoundError: No module named 'distutils'`**
`aioredis` imports `distutils`, which was removed in Python 3.12. This broke 8 auth tests and caused 500 errors on `/auth/register` in the e2e tests. I replaced `aioredis` with the maintained `redis` package and changed the import in `app/auth/redis.py` to `from redis import asyncio as aioredis`.

**3. Tests passed locally but failed in GitHub Actions**
CI reported `No module named 'redis'` because I had installed `redis` locally but never added it to `requirements.txt`. CI builds a fresh environment from that file only. I added `redis` to `requirements.txt` and removed `aioredis`.

**4. Playwright install failed in CI**
The log showed `Playwright does not support chromium on ubuntu26.04-x64`. `ubuntu-latest` had moved to 26.04, so I pinned the runner to `ubuntu-24.04` in the workflow file.

**5. Docker Hub login and push errors**
Secrets are stored per repository, so I had to add `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN` to this repo again. The first token caused `401 Unauthorized: access token has insufficient scopes` on push. I generated a new access token with Read & Write permissions, updated the GitHub secret, and the image pushed successfully.

**6. "Failed to fetch" in Swagger UI**
The `/docs` page showed `Failed to fetch` when executing requests because the server was no longer running on port 8001. I restarted the app, hard-refreshed the page, and the registration, login, and calculation endpoints worked.

### Lessons Learned
- Keep the local Python version consistent with CI.
- Every package the code imports must be listed in `requirements.txt`.
- Pin runner images instead of using `ubuntu-latest`.
- Docker Hub tokens need Read & Write access to push images.