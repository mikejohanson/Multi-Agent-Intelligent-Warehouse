- need to login to nvcr.io with `docker login nvcr.io` -- should prob included instructions on how to do this.

- docker command in jupyter notebook for self hosting example should be: 
**Self-Hosting Example:**
```bash
# Deploy LLM NIM on your server
docker run --gpus all -p 8000:8000 -e NVIDIA_API_KEY=\"\" nvcr.io/nim/nvidia/llama-3.3-nemotron-super-49b-v1:latest \
```
- annoy a transitive dependency of nemoguardrails requires C++ compiler. so need a pre req for build-essential
    - subsequntely python-dev is needed in order to provide the headers so annoy can compile when doing pip install
- syntax error in device_ids_yaml line
- step 8, failed to find python despite already being in the venv
- step 9 has same problem, failed to find python due to hardcoded /env path
- step 11, a bit confusing theres no step to run to do docker for me
  - invalid lockfile for package.json : 
    ```
    => ERROR [frontend 4/5] RUN npm ci                                                                                                                                                                                                                                                                                                                      2.3s
    ------
    > [frontend 4/5] RUN npm ci:
    2.243 npm error code EUSAGE
    2.244 npm error
    2.244 npm error `npm ci` can only install packages when your package.json and package-lock.json or npm-shrinkwrap.json are in sync. Please update your lock file with `npm install` before continuing.
    2.244 npm error
    2.244 npm error Invalid: lock file's eslint@9.39.2 does not satisfy eslint@8.57.1
    2.244 npm error Invalid: lock file's @eslint/eslintrc@3.3.3 does not satisfy @eslint/eslintrc@2.1.4
    2.244 npm error Invalid: lock file's @eslint/js@9.39.2 does not satisfy @eslint/js@8.57.1
    2.244 npm error Missing: @humanwhocodes/config-array@0.13.0 from lock file
    2.244 npm error Missing: @ungap/structured-clone@1.3.0 from lock file
    2.244 npm error Missing: doctrine@3.0.0 from lock file
    2.244 npm error Invalid: lock file's eslint-scope@8.4.0 does not satisfy eslint-scope@7.2.2
    2.244 npm error Invalid: lock file's espree@10.4.0 does not satisfy espree@9.6.1
    2.244 npm error Invalid: lock file's file-entry-cache@8.0.0 does not satisfy file-entry-cache@6.0.1
    2.244 npm error Invalid: lock file's globals@14.0.0 does not satisfy globals@13.24.0
    2.244 npm error Missing: is-path-inside@3.0.3 from lock file
    2.244 npm error Missing: js-yaml@4.1.1 from lock file
    2.244 npm error Invalid: lock file's ajv@6.12.6 does not satisfy ajv@6.14.0
    2.244 npm error Missing: @humanwhocodes/object-schema@2.0.3 from lock file
    2.244 npm error Invalid: lock file's flat-cache@4.0.1 does not satisfy flat-cache@3.2.0
    2.244 npm error Missing: type-fest@0.20.2 from lock file
    2.244 npm error Missing: argparse@2.0.1 from lock file
    2.244 npm error
    2.244 npm error Clean install a project
    2.244 npm error
    2.244 npm error Usage:
    2.244 npm error npm ci
    2.244 npm error
    2.244 npm error Options:
    2.244 npm error [--install-strategy <hoisted|nested|shallow|linked>] [--legacy-bundling]
    2.244 npm error [--global-style] [--omit <dev|optional|peer> [--omit <dev|optional|peer> ...]]
    2.244 npm error [--include <prod|dev|optional|peer> [--include <prod|dev|optional|peer> ...]]
    2.244 npm error [--strict-peer-deps] [--foreground-scripts] [--ignore-scripts] [--no-audit]
    2.244 npm error [--no-bin-links] [--no-fund] [--dry-run]
    2.244 npm error [-w|--workspace <workspace-name> [-w|--workspace <workspace-name> ...]]
    2.244 npm error [-ws|--workspaces] [--include-workspace-root] [--install-links]
    2.244 npm error
    2.244 npm error aliases: clean-install, ic, install-clean, isntall-clean
    2.244 npm error
    2.244 npm error Run "npm help ci" for more info
    2.246 npm notice
    2.246 npm notice New major version of npm available! 10.8.2 -> 11.10.1
    2.246 npm notice Changelog: https://github.com/npm/cli/releases/tag/v11.10.1
    2.246 npm notice To update run: npm install -g npm@11.10.1
    2.246 npm notice
    [+] up 11/13ror A complete log of this run can be found in: /root/.npm/_logs/2026-02-21T18_43_43_761Z-debug-0.log
    ✔ Image nginx:1.25.3-alpine Pulled                                                                                                                                                                                                                                                                                                                       3.3s
    ⠙ Image compose-backend     Building                                                                                                                                                                                                                                                                                                                     9.0s
    ⠙ Image compose-frontend    Building                                                                                                                                                                                                                                                                                                                     9.0s
    Dockerfile.frontend:12

    --------------------

    10 |     

    11 |     # Install dependencies

    12 | >>> RUN npm ci

    13 |     

    14 |     # Copy frontend source

    --------------------

    target frontend: failed to solve: process "/bin/sh -c npm ci" did not complete successfully: exit code: 1
    ```

    - need to regen your package-lock.json and make sure its in sync with package.json.
        - Side note, addressing the 40+ high vulnerabilities would probably be a good idea

    - backend docker file missing some dependencies: 
    namely tiktoken, gaurdrails,
    needed to update requirements.docker.txt to:
    ```
    fastapi>=0.120.0
    starlette>=0.49.1
    uvicorn[standard]==0.30.1
    pydantic>=2.7
    httpx>=0.27
    python-dotenv>=1.0
    loguru>=0.7
    langgraph>=1.0.5  # Security: Fixed CVE-2025-8709 (SQL injection in langgraph-checkpoint-sqlite). We use 1.0.5+ (includes fix). Note: We use in-memory state (no SQLite checkpoint) as additional defense.
    langgraph-checkpoint>=3.0.0  # Security: Fixed CVE-2025-64439 (RCE in JsonPlusSerializer). Version 3.0.0+ fixes RCE vulnerability. Note: We use in-memory state (no checkpoint backend), but pinning secure version for transitive dependencies.
    asyncpg>=0.29.0
    pymilvus>=2.3.0
    numpy>=1.24.0
    langchain-core>=1.2.6  # Security: Fixed CVE-2025-68664 (serialization injection) and CVE-2024-28088 (directory traversal). We use 1.2.6 (latest, includes fixes). Note: We use json.dumps(), not LangChain serialization, as additional defense.
    aiohttp>=3.13.3  # Security: Fixed zip bomb DoS (BDSA) in 3.13.3+ (DEFAULT_MAX_DECOMPRESS_SIZE=32MiB). Also fixes: CVE-2024-52304, CVE-2024-30251, CVE-2023-37276, CVE-2024-23829. Client-only usage (not server) = additional defense.
    PyJWT>=2.8.0
    passlib[bcrypt]>=1.7.4
    email-validator>=2.0.0
    PyYAML>=6.0
    prometheus-client>=0.19.0
    paho-mqtt>=1.6.0
    websockets>=11.0.0
    pymodbus>=3.0.0
    bacpypes3>=0.0.100  # BACnet protocol library for IoT safety sensors (optional - only needed for BACnet integration)
    requests>=2.31.0
    pyserial>=3.5
    redis>=5.0.0
    tiktoken
    python-multipart
    scikit-learn>=1.5.0
    pandas>=1.2.4
    xgboost>=1.6.0
    psutil>=5.9.0
    click>=8.0.0
    psycopg[binary]>=3.0.0
    anyio>=4.0.0
    # Document Processing
    Pillow>=10.3.0
    pdf2image==1.17.0
    pdfplumber==0.11.8
    # NeMo Guardrails SDK
    nemoguardrails>=0.19.0
    ```

- healthcheck for backend giving 200 despite DB access errors:
```
ERROR:src.api.routers.health:Database health check failed: [Errno 111] Connection refused
INFO:     127.0.0.1:55122 - "GET /api/v1/health HTTP/1.1" 200 OK
ERROR:src.api.routers.health:Database health check failed: [Errno 111] Connection refused
INFO:     127.0.0.1:33102 - "GET /api/v1/health HTTP/1.1" 200 OK
ERROR:src.api.routers.health:Database health check failed: [Errno 111] Connection refused
INFO:     127.0.0.1:45422 - "GET /api/v1/health HTTP/1.1" 200 OK
ERROR:src.api.routers.health:Database health check failed: [Errno 111] Connection refused
INFO:     127.0.0.1:49812 - "GET /api/v1/health HTTP/1.1" 200 OK
```

which point to issue that .env is setup to use localhost instead of the container name -- i think these types of issues would be solved by having two different notebooks -- one for only docker, only for only dev, instead of trying to mix it all in 1. This caused a similar issue with the proxy targets: The proxy targets http://localhost:8001, but inside the Docker container, the backend service is reachable at http://backend:8001, not localhost. spots in other pythong files (like sql_retriever) are also hardcoded to localhost and not picking env var which prevents a docker env from working.