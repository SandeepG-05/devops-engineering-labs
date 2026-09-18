# 🏗️ Module 01: Production Dockerfile Engineering

This folder houses validated blueprints for production-grade Docker containers. Each file addresses a critical enterprise infrastructure standard.

---

### 🛡️ 1. The Secure Non-Root Pattern (`Dockerfile.user`)
* **Concept:** Mitigates host-takeover vectors by dropping root privileges immediately after environment provisioning.

```bash
# Build command:
docker build -t test-user -f Dockerfile.user .

# Execution command:
docker run --rm test-user
```

#### 📋 Expected Reference Output:
```text
appuser
```
*(💡 Security Note: If this ever returns `root`, your container has failed compliance checks!)*

---

### 🧪 2. The Build-Time Dynamic Injector (`Dockerfile.args`)
* **Concept:** Leverages transient build-time variables (`ARG`) to pass metadata during compilation without leaking parameters into live execution contexts (`ENV`).

```bash
# Build command with custom version injection:
docker build --build-arg BUILD_VERSION=2.4.1 -t test-args -f Dockerfile.args .

# Execution command:
docker run --rm test-args
```

#### 📋 Expected Reference Output:
```text
Running in: Production (Build Ver: )
```
*(💡 Security Note: Notice how `Build Ver:` is completely blank at execution time. This confirms the transient build variable successfully vanished after compilation, protecting your secrets).*

---

### 🎛️ 3. The Graceful Interceptor (`Dockerfile.exec`)
* **Concept:** Combines fixed executables (`ENTRYPOINT`) with default parameters (`CMD`) to transform containers into flexible, predictable command-line utilities.

#### Test A: Run with defaults
```bash
docker build -t test-exec -f Dockerfile.exec .
docker run --rm test-exec
```
#### 📋 Expected Reference Output:
```text
System status: All engines nominal.
```

#### Test B: Overwriting the parameter at runtime
```bash
docker run --rm test-exec "Warning: High memory load detected!"
```
#### 📋 Expected Reference Output:
```text
System status: Warning: High memory load detected!
```

---

### 🩺 4. The Self-Healing Health Guard (`Dockerfile.health`)
* **Concept:** Provisions internal verification loops so upstream orchestrators can track container viability natively.

```bash
docker build -t test-health -f Dockerfile.health .
docker run -d --name my-health-check test-health

# Wait 7-10 seconds for the interval loop to fire, then run:
docker ps
```

#### 📋 Expected Reference Output:
```text
CONTAINER ID   IMAGE         STATUS                     PORTS
a1b2c3d4e5f6   test-health   Up 8 seconds (healthy)     80/tcp
```
*(💡 Management Note: Look closely at the `STATUS` column—Docker natively tracks the text `(healthy)` right inside your terminal).*

---

### 📦 5. Safe Sandbox Directory (`Dockerfile.work`)
* **Concept:** Enforces clean filesystem segregation by establishing explicit isolation layers (`WORKDIR`) for app execution.

```bash
docker build -t test-work -f Dockerfile.work .
docker run --rm test-work
```

#### 📋 Expected Reference Output:
```text
/opt/enterprise/app
```
