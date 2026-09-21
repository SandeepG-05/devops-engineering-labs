# 🏗️ Module 01: Production Dockerfile Engineering

This folder houses validated blueprints for production-grade Docker containers. Each file addresses a critical enterprise infrastructure standard, focusing on application security, runtime isolation, and layer optimization.

---

## 🛡️ 1. Security & Compliance: Drop-Down Execution Context
* **File:** `Dockerfile.user`
* **Concept:** Mitigates host-takeover vectors by dropping root privileges immediately after environment provisioning.

```bash
# Build command:
docker build -t test-user -f Dockerfile.user .

# Execution command:
docker run --rm test-user
```

📋 **Expected Reference Output:**
```text
appuser
```
*(💡 Security Note: If this ever returns root, your container has failed compliance checks!)*

---

## 🧪 2. Build Architecture: The Build-Time Dynamic Injector Matrix (ARG vs ENV)
* **Concept:** Leverages transient build-time variables (`ARG`) to handle compilation metadata without leaking parameters or configuration variables into live execution contexts (`ENV`). To showcase different engineering approaches, we test three distinct behavioral patterns:

### Pattern A: The Static Release Artifact (Baked File)
* **File:** `Dockerfile.args-file`
* **Mechanics:** Consumes the build argument to permanently freeze the release string into an isolated text file inside the image layer, while stripping the dynamic variable out of the active environment list.

```bash
# Build command with custom version injection:
docker build --build-arg BUILD_VERSION=2.4.1 -t test-args-file -f Dockerfile.args-file .

# Execution command:
docker run --rm test-args-file
```

📋 **Expected Reference Output:**
```text
Running in: Production (Build Ver: 2.4.1)
```

### Pattern B: The Blank Fallback Context (Vanished Env)
* **File:** `Dockerfile.args-env`
* **Mechanics:** Demonstrates a strict security configuration where variables are not written to files. It validates that the runtime environment string evaluates to completely empty at execution time, protecting secrets from extraction tools.

```bash
# Build command:
docker build --build-arg BUILD_VERSION=2.4.1 -t test-args-env -f Dockerfile.args-env .

# Execution command:
docker run --rm test-args-env
```

📋 **Expected Reference Output:**
```text
Running in: Production (Build Ver: )
```

### Pattern C: Dual-Layer Compliance Verification (Side-by-Side Audit)
* **File:** `Dockerfile.args-verify`
* **Mechanics:** Acts as an automated compliance verification audit tool. It outputs both states side-by-side to prove that filesystem compilation succeeded while simultaneously confirming the live execution runtime remains hollow.

```bash
# Build command:
docker build --build-arg BUILD_VERSION=2.4.1 -t test-args-verify -f Dockerfile.args-verify .

# Execution command:
docker run --rm test-args-verify
```

📋 **Expected Reference Output:**
```text
Config baked into file: 2.4.1
Live Environment variable: []
```

---

## 🎛️ 3. Execution Flow: Graceful CLI Parameter Interception
* **File:** `Dockerfile.exec`
* **Concept:** Combines fixed executables (`ENTRYPOINT`) with default parameters (`CMD`) to transform containers into flexible, predictable command-line utilities.

### Test A: Run with defaults
```bash
docker build -t test-exec -f Dockerfile.exec .
docker run --rm test-exec
```

📋 **Expected Reference Output:**
```text
System status: All engines nominal.
```

### Test B: Overwriting the parameter at runtime
```bash
docker run --rm test-exec "Warning: High memory load detected!"
```

📋 **Expected Reference Output:**
```text
System status: Warning: High memory load detected!
```

---

## 🩺 4. Execution Flow: Native Container Health Checks
* **File:** `Dockerfile.health`
* **Concept:** Provisions internal verification loops so upstream orchestrators can track container viability natively.

```bash
docker build -t test-health -f Dockerfile.health .
docker run -d --name my-health-check test-health

# Wait 7-10 seconds for the interval loop to fire, then run:
docker ps
```

📋 **Expected Reference Output:**
```text
CONTAINER ID   IMAGE         STATUS                     PORTS
a1b2c3d4e5f6   test-health   Up 8 seconds (healthy)     80/tcp
```
*(💡 Management Note: Look closely at the STATUS column—Docker natively tracks the text `(healthy)` right inside your terminal. Run `docker rm -f my-health-check` to clean up when finished.)*

---

## 📦 5. Runtime Isolation: Safe Sandbox Directory
* **File:** `Dockerfile.work`
* **Concept:** Enforces clean filesystem segregation by establishing explicit isolation layers (`WORKDIR`) for app execution.

```bash
docker build -t test-work -f Dockerfile.work .
docker run --rm test-work
```

📋 **Expected Reference Output:**
```text
/opt/enterprise/app
```
