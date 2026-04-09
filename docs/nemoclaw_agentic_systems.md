# Learning Guide: NemoClaw + OpenClaw from First Commands to Trading Agent Teams

---

## 1. Core idea in plain language

NemoClaw is **not** your agent.  
NemoClaw is the **“building + guard + rulebook”** that wraps around **OpenClaw**, the **actual agent** (AI) that thinks, plans, and works.

Think of it like this:

- **OpenClaw** = the smart worker in an office.  
- **NemoClaw + OpenShell** = the locked office building, the security desk, the network gate, and the rulebook.  

You run NemoClaw commands to **build, lock, and operate** that building.  
Inside, OpenClaw does the real job.

---

## 2. The command map: from create to destroy

### 2.1. `nemoclaw onboard` – build the office

- **What you do:**  
  ```bash
  nemoclaw onboard
  ```
- **What happens:**  
  - NemoClaw installs Node, OpenShell, and the NVIDIA gateway if needed.  
  - Creates a **sandbox** (Docker container) for your agent.  
  - Installs **OpenClaw inside that sandbox**.  
  - Sets up default **network, filesystem, and inference policies**.

- **Mental model:**  
  > “Breaking ground, wiring electricity, moving the agent’s desk in.”  

- **Link to OpenClaw:**  
  - After `onboard`, you already have an OpenClaw agent, but it’s **locked inside a sandbox**, not loose on your laptop.

---

### 2.2. `nemoclaw <assistant> connect` – open the door and start working

- **What you do:**  
  ```bash
  nemoclaw my-agent connect
  ```
- **What happens:**  
  - NemoClaw asks the **OpenShell gateway** to open the sandbox.  
  - A secure tunnel is created into that container.  
  - OpenClaw starts (or resumes) and you start chatting / running tools.

- **Mental model:**  
  > “Open the office door and sit at the agent’s desk.”

- **Link to OpenClaw:**  
  - The agent’s **logic** (thinking, tools, code) is still pure OpenClaw; NemoClaw just **pipes your input and its output** through the sandbox safely.

---

### 2.3. `nemoclaw <assistant> status` – check the building and the office

- **What you do:**  
  ```bash
  nemoclaw my-agent status
  ```
- **What happens:**  
  - Checks if the sandbox is `Running` / `Ready`.  
  - Checks if the **gateway** (NemoClaw/OpenShell) is reachable.  
  - Checks if the **inference model** (Nemotron, OpenAI, etc.) is reachable.

- **Mental model:**  
  > “Turn on the building’s power and network status panel.”

- **When to use it:**  
  - Before you run `connect` after a reboot.  
  - After you change policies or models.  
  - If `connect` ever hangs or fails.

---

### 2.4. `nemoclaw <assistant> logs [--follow]` – watch the security camera feed

- **What you do:**  
  ```bash
  nemoclaw my-agent logs
  nemoclaw my-agent logs --follow
  ```
- **What happens:**  
  - Reads the Docker/OpenShell log stream for that sandbox.  
  - Shows OpenClaw’s stdout/stderr (tool calls, errors, plan traces).  
  - Shows OpenShell’s security traces (blocked files, blocked URLs, PII‑stripping messages).

- **Mental model:**  
  > “Flip on the security‑camera feed for the office; if you add `--follow`, you watch it live.”

- **Why it helps:**  
  - Lets you see the **agent’s behavior** and the **sandbox’s reactions** at the same time.  
  - Crucial for debugging policy and tool issues.

---

### 2.5. `nemoclaw <assistant> destroy` – tear down the office and erase its files

- **What you do:**  
  ```bash
  nemoclaw my-agent destroy
  ```
- **What happens:**  
  - Stops the sandbox container.  
  - Removes the container from Docker/OpenShell.  
  - Deletes the agent’s **persistent workspace** inside that sandbox (logs, notes, configs, memory).

- **Mental model:**  
  > “Lock the office, shred the papers, and demolish the building.”

- **Important:**  
  - After `destroy`, you **cannot simply resume** that agent.  
  - You must `onboard` (or recreate it), and it will **lose all previous memory** unless you backed it up.

---

### 2.6. `nemoclaw stop` – power down the whole building (keep everything)

- **What you do:**  
  ```bash
  nemoclaw stop
  ```
- **What happens:**  
  - Stops the **gateway** and **NIM inference containers** (if any).  
  - Stops the sandboxes.  
  - Keeps all **config and disk** unchanged.

- **Mental model:**  
  > “Turn off the main breaker, but leave the offices and furniture intact.”

- **Use when:**  
  - You want to save GPU/CPU between sessions.  
  - You expect to come back and use the same assistants.

---

### 2.7. `nemoclaw uninstall` – erase the whole campus

- **What you do:**  
  ```bash
  nemoclaw uninstall
  ```
- **What happens:**  
  - Removes all NemoClaw/OpenShell containers, images, and config.  
  - Wipes all sandboxes and their contents.  

- **Mental model:**  
  > “Demolish the whole campus, erase the blueprints, clear the land.”

- **Use when:**  
  - You’re switching workflows or machines.  
  - You want a clean start without old agents.

---

## 3. The “layer cake” of NemoClaw

Imagine the whole system as a **stack** (like layers of a cake):

```text
HOST (your machine)
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  LAYER 1: You (the human) running nemoclaw CLI               │
│  → Type: onboard, connect, status, logs, destroy, stop, etc.│
│                                                              │
│  LAYER 2: NemoClaw plugin + OpenShell gateway                │
│  → The “control tower” that talks to and guards the agents.  │
│                                                              │
│  LAYER 3: OpenShell sandboxes (containers)                   │
│  → Each locked office for one agent.                         │
│                                                              │
│  LAYER 4: OpenClaw agents (inside their sandboxes)           │
│  → The smart workers that plan, run tools, and chat.         │
│                                                              │
│  LAYER 5: Network + models (outside the building)            │
│  → Cloud models, APIs, websites, data feeds, etc.            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

From here, we add **logical layers** on top of this physical stack:

- **Policy layer** (rulebook).  
- **Filesystem layer** (desk and drawers).  
- **Network layer** (gate to the outside).  
- **Inference layer** (phone line to models).  
- **Skills & tools layer** (toolbelt and apps).  

Each of these is enforced **by NemoClaw + OpenShell** on top of the sandbox.

---

## 4. Policy layer: the rulebook

### 4.1. What the policy is

The **policy** is a **YAML file** (the “rulebook”) that defines:

- What the agent can touch on disk (`/sandbox`, `/tmp`, etc.).  
- Which URLs it can call (`allowed_urls`).  
- Which model the gateway routes to (`inference`).  
- Which system calls and escalations it can perform (`process`).

Simplified example:

```yaml
filesystem:
  read_only:
    - /usr
    - /lib
    - /etc
  read_write:
    - /sandbox
    - /tmp

network:
  allowed_urls:
    - "nvidia.com"
    - "openai.com"
  require_approval_for_new_domains: true

inference:
  provider: nemotron
  gateway: "integrate.api.nvidia.com"

process:
  allow_privilege_escalation: false
  allow_dangerous_syscalls: false
```

---

### 4.2. How you change the rulebook

You **can** edit the YAML file, but **when** it applies matters:

- **Filesystem / process rules** are “frozen” at sandbox creation.  
  - To change them, you must destroy and recreate the agent:
    ```bash
    nemoclaw my-agent destroy
    nemoclaw onboard
    ```
- **Network / inference rules** can be changed **on the fly** via:
  - `nemoclaw <assistant> policy-add <preset>` (adds URLs, tools, etc.).  
  - `nemoclaw term` (OpenShell TUI) to approve new domains dynamically.

Best practice:

- Keep your edited YAML in your own `~/.nemoclaw/` or project folder.  
- After `npm install -g nemoclaw`, copy your version back into the `node_modules` policy file if needed.  

In short:

- **Yes, you can change the policy YAML.**  
- **No, it’s not silently updated inside a running sandbox; you must recreate it to apply structural changes.**

---

## 5. Filesystem layer: the agent’s desk and drawers

### 5.1. What NemoClaw does

By default, the sandbox is configured so:

- **System paths** (`/usr`, `/lib`, `/etc`) → **read‑only**.  
- **Your home, SSH keys, AWS config** → **blocked**.  
- **`/sandbox` and `/tmp`** → **read‑write**, the agent’s workspace.

That means:

- The agent can:
  - Create, edit, and delete files in `/sandbox` and `/tmp`.  
- The agent cannot:
  - Modify system binaries.  
  - Read your `~/.ssh`, `~/.aws`, or similar sensitive files.

---

### 5.2. How policy controls it

In the YAML, the `filesystem` block says:

```yaml
filesystem:
  read_only:
    - /usr
    - /lib
    - /etc
  read_write:
    - /sandbox
    - /tmp
```

- Add a path to `read_write` → the agent can edit there.  
- Remove it → the agent cannot.

Because these are enforced at **sandbox creation**, significant changes require:

```bash
nemoclaw my-agent destroy
nemoclaw onboard
```

to rebuild the agent with the updated rules.

---

## 6. Network layer: the gate to the outside

### 6.1. What the network layer is

The **network layer** is the **gate** that decides:

- Which websites and APIs the agent can call.  
- Which outgoing requests are blocked or require approval.

Behind the scenes:

- All HTTP calls from the agent go through the **NemoClaw gateway** (or local NIM, if you run it).  
- The gateway:
  - Reads `allowed_urls` from the policy.  
  - Blocks or logs other domains.  
  - Can strip PII from requests before sending them to cloud models.

---

### 6.2. The “network commands”

There’s no `nemoclaw network` command, but you work with:

- `nemoclaw <assistant> policy-add <preset>`  
  - Example: `github`, `openai`, `npm`, `huggingface`.  
  - Adds those URLs to the allowed list.  
- `nemoclaw term`  
  - Opens the OpenShell TUI.  
  - Lets you **watch and approve** new domains the agent wants to call.

And you see the network layer in action in:

```bash
nemoclaw my-agent logs --follow
```

where you see lines like:

- “ALLOWED: github.com”.  
- “BLOCKED: bad-site.com”.  

---

## 7. Inference layer: the secure phone line to models

### 7.1. What the inference layer is

The **inference layer** is the **phone line**:

- OpenClaw needs to “think deeply” → it calls a model (Nemotron, OpenAI, Claude, etc.).  
- Instead of calling directly, it goes via the **NVIDIA gateway** (or a local NIM container).  
- The gateway:
  - Routes the request to the right model endpoint.  
  - Can strip or redact PII (emails, names, card‑like strings).  

So the agent **thinks** it’s talking to a model, but the **real request flows through** the inference layer.

---

### 7.2. How it plugs into your commands

- `nemoclaw onboard` → **chooses the model provider** (Nemotron, OpenAI, etc.) and gateway URL.  
- `nemoclaw my-agent status` → reports whether the gateway and model endpoint are healthy.  
- `nemoclaw my-agent logs --follow` → shows inference‑routing and PII‑stripping traces.

Intuitively:

> **Inference layer = the phone line; the gateway is the call‑center guard that cleans the conversation.**

---

## 8. Skills & tools layer: the agent’s toolbelt

### 8.1. What a “skill” is

A **skill** is a **small program** (script or service) that the agent can call, like:

```text
“Use the `price_action` skill on this OHLC data.”
```

Behind the scenes, OpenClaw translates that into:

- Run `python /sandbox/skills/price_action/run.sh` and pass data via stdin.  
- Or call `http://localhost:8000/price_action` and read the JSON result.

---

### 8.2. Example: `price_action` skill (full code)

#### 8.2.1. Folder layout in the sandbox

Inside the agent’s sandbox, for example:

```text
/sandbox/skills/
  └── price_action/
      ├── price_action.py   # logic
      └── run.sh            # wrapper
```

Create the files:

**`/sandbox/skills/price_action/price_action.py`**

```python
# price_action.py
import sys
import json
import statistics

def simple_trend(ohlc_list):
    close_prices = [float(bar["close"]) for bar in ohlc_list]
    if len(close_prices) < 2:
        return "flat"
    start = close_prices
    end = close_prices[-1]
    if end > start:
        return "up"
    elif end < start:
        return "down"
    return "flat"

def simple_support_resistance(ohlc_list):
    lows = [float(bar["low"]) for bar in ohlc_list]
    highs = [float(bar["high"]) for bar in ohlc_list]
    if not lows or not highs:
        return {"support": [], "resistance": []}

    support_candidates = lows
    resistance_candidates = highs

    # Very basic: take min and max for support and resistance.
    support = [
        statistics.quantiles(support_candidates, n=10),
        statistics.quantiles(support_candidates, n=10),[1]
    ]
    resistance = [
        statistics.quantiles(resistance_candidates, n=10)[-2],
        statistics.quantiles(resistance_candidates, n=10)[-1],
    ]

    return {"support": sorted(support), "resistance": sorted(resistance)}

def main():
    # Read JSON from stdin (the agent sends data)
    data = json.load(sys.stdin)

    # Assume data["bars"] is a list of { "open", "high", "low", "close", "time" }
    bars = data.get("bars", [])

    if not bars:
        print(json.dumps({"error": "no bars provided"}))
        return

    trend = simple_trend(bars)
    sr = simple_support_resistance(bars)
    summary = f"Trend: {trend}. Key support: {sr['support']}, resistance: {sr['resistance']}"

    result = {
        "trend": trend,
        "support": sr["support"],
        "resistance": sr["resistance"],
        "summary": summary,
        "last_price": bars[-1]["close"],
    }

    print(json.dumps(result))


if __name__ == "__main__":
    main()
```

**`/sandbox/skills/price_action/run.sh`**

```bash
#!/bin/bash

# This is what the agent will call.
python3 /sandbox/skills/price_action/price_action.py
```

Make it executable:

```bash
chmod +x /sandbox/skills/price_action/run.sh
```

Then, in your OpenClaw tools config, register:

```json
{
  "name": "price_action",
  "description": "Analyze price action on a bar list and return trend, support, resistance.",
  "cli_command": "/sandbox/skills/price_action/run.sh"
}
```

From now on, the agent can call `price_action` as a tool.

---

## 9. Multi‑agent & memory: teams of agents, with long‑term memory

### 9.1. Multi‑agent: multiple offices, one building

In a **multi‑agent setup**:

- Each agent has its own name and sandbox:
  - `dataflow`, `trader`, `risk`, `monitor`, etc.  
- Each has its own:
  - Policy.
  - Filesystem layout.
  - URLs.
  - Model endpoint (if you want).
  - Skills (e.g., `price_action` only for `trader`).

They “talk” by:

- **Shared folders** (a volume mounted into multiple sandboxes).  
- **HTTP services** (each agent exposes a local API, allowed in the other’s `allowed_urls`).  
- **You** copying files between sandboxes.

Example for trading:

- `dataflow` → fetches OHLC, stores it in `/shared/data`.  
- `trader` → reads `/shared/data`, calls `price_action`.  
- `trader` → reads `/shared/data`, calls `price_action`.  
- `risk` → computes sizing, watches account metrics.  
- `monitor` → watches logs, disk, network, alerts you.

This keeps each agent **focused and isolated**, but they can cooperate via **controlled channels**.

---

### 9.2. Memory – the agent’s journal on the shelf

“Memory” is **just files** the agent writes to its sandbox:

- Notes.  
- Plans.  
- Tool results.  
- Chat logs.  

Typically stored under something like:

```text
/sandbox/.openclaw/memory/
```

- As long as the **sandbox exists**, the agent remembers.  
- When you run `nemoclaw my-agent destroy`, those files are gone → the agent “forgets everything.”  

To back up memory:

- Before `destroy` (or host wipe), copy the memory folder:

  ```bash
  docker cp <sandbox-container>:/sandbox/.openclaw/memory \
           /backup/agent-my-agent/memory
  ```

- Restore it into a new sandbox when you recreate the agent.

In plain language:

> **Memory is the agent’s personal journal on the shelf; you can back it up, restore it, or even share it carefully between agents (via shared volumes).**

---

## 10. Putting it all together: a practical “last step” wiring‑up pattern

### 10.1. Goal

Wire up:

- A **`dataflow`** agent that fetches and stores OHLC.  
- A **`trader`** agent that:
  - Reads that OHLC.  
  - Calls your **`price_action`** skill.  
  - Makes trading‑related decisions.  

Both respect NemoClaw’s layers (policy, filesystem, network, inference, skills, memory).

---

### 10.2. Concrete folder layout (on your host)

On your host machine, you might structure it like:

```text
~/nemoclaw-practice/
  ├── shared/
  │   └── data/             # shared data volume for agents
  │       ├── ohlc.json
  │       ├── latest_price_action.json
  │       ├── risk.json
  │       └── trade_log.json
  ├── sandboxes/            # conceptually: where NemoClaw puts sandboxes
  └── backups/
      └── trader/
      └── dataflow/
```

The idea is:

- All agents get `/shared` mounted into their sandboxes, so they all see `/shared/data/*.json`.  
- `dataflow` writes OHLC.  
- `trader` reads that and calls `price_action`.  
- `risk` and `monitor` read the results and logs.

---

### 10.3. How you mount the shared data volume (NemoClaw / Docker style)

In your OpenShell / policy config, when sandboxes are created, ensure:

```yaml
volumes:
  - type: bind
    source: /home/$USER/nemoclaw-practice/shared
    target: /shared
    readonly: false
```

(`/home/$USER` is your home directory, e.g., `/home/you`.)

From inside any agent, you can:

```bash
cat /shared/data/ohlc.json
```

and both `dataflow` and `trader` can read/write the shared data.

---

### 10.4. `dataflow` agent: fetch and store OHLC

#### 10.4.1. `dataflow` role

- Periodically fetches OHLC for a symbol (e.g., `ES` or `AAPL`).  
- Writes it as JSON to `/shared/data/ohlc.json`.  
- In this example, we’ll **sketch** a `dataflow` that uses **IBKR TWS** (via `ib_insync`) as the data source.

#### 10.4.2. Skeleton `dataflow` script

Inside `dataflow`’s sandbox, create:

**`/sandbox/scripts/fetch_ibkr_ohlc.py`**

```python
# fetch_ibkr_ohlc.py
import json
import time
from datetime import datetime, timedelta
from ib_insync import IB, Future, Stock

# --- CONFIG ---
TWS_HOST = "127.0.0.1"
TWS_PORT = 7497
CLIENT_ID = 1
SYMBOL = "ES"            # or "AAPL", etc.
EXCHANGE = "GLOBEX"
CURRENCY = "USD"
DURATION = "1 D"
BAR_SIZE = "1 min"
WHAT_TO_SHOW = "TRADES"
DAYS = 1

def connect_tws():
    ib = IB()
    ib.connect(TWS_HOST, TWS_PORT, clientId=CLIENT_ID)
    time.sleep(1)
    return ib

def fetch_ohlc(symbol, interval="1 min", days=1):
    ib = connect_tws()
    try:
        if symbol == "ES":
            contract = Future(symbol, "202609", EXCHANGE)
        elif symbol == "AAPL":
            contract = Stock(symbol, "SMART", CURRENCY)
        else:
            raise ValueError("Unsupported symbol")

        end = ""
        dur = f"{days} D"
        bsi = f"1 {interval.upper()}"

        bars = ib.reqHistoricalData(
            contract,
            endDateTime=end,
            durationStr=dur,
            barSizeSetting=bsi,
            whatToShow=WHAT_TO_SHOW,
            useRTH=False,
            formatDate=1,
        )

        ohlc = []
        for b in bars:
            ohlc.append({
                "time": b.date.timestamp(),
                "open": b.open,
                "high": b.high,
                "low": b.low,
                "close": b.close,
                "volume": b.volume,
            })
        return ohlc
    finally:
        ib.disconnect()

def main():
    print("Connecting to TWS and fetching OHLC...")
    bars = fetch_ohlc(SYMBOL, interval=BAR_SIZE, days=DAYS)

    print(f"Fetched {len(bars)} bars.")

    with open("/shared/data/ohlc.json", "w") as f:
        json.dump(bars, f, indent=2)

    print("OHLC saved to /shared/data/ohlc.json")


if __name__ == "__main__":
    main()
```

#### 10.4.3. How to hook into IBKR via TWS

- Run TWS on your host on `127.0.0.1:7497`.  
- Inside `dataflow`, install `ib_insync`:

  ```bash
  pip install ib_insync
  ```

- Run `fetch_ibkr_ohlc.py` periodically (e.g., via a loop or cron).

You can now **plug in OpenBB** as a fallback:

```python
from openbb_ibkr import ibkr_client

bars = ibkr_client.get_ohlc(symbol="ES", interval="1m", days=1)
```

and write that to `/shared/data/ohlc.json` instead or in addition.

---

### 10.5. `trader` agent: call `price_action` on OHLC

#### 10.5.1. `trader` role

- Reads `/shared/data/ohlc.json`.  
- Calls the `price_action` skill.  
- Uses the result to make a **mock trade decision** (you can plug in real IBKR transactions later).

Ensure:

- `/sandbox/skills/price_action/run.sh` is registered as a tool.  
- `run_trade_cycle.py` can read and serialize data correctly.

#### 10.5.2. `run_trade_cycle.py`

Inside `trader`’s sandbox, create:

**`/sandbox/scripts/run_trade_cycle.py`**

```python
# run_trade_cycle.py
import json
import subprocess

def read_ohlc():
    with open("/shared/data/ohlc.json", "r") as f:
        return json.load(f)

def call_price_action(bars):
    p = subprocess.run(
        ["/sandbox/skills/price_action/run.sh"],
        input=json.dumps({"bars": bars}),
        text=True,
        capture_output=True,
    )
    if p.returncode != 0:
        raise Exception(f"price_action failed: {p.stderr}")
    return json.loads(p.stdout)

def main():
    print("Reading OHLC from /shared/data/ohlc.json...")
    bars = read_ohlc()

    print("Calling price_action skill...")
    result = call_price_action(bars)

    with open("/shared/data/latest_price_action.json", "w") as f:
        json.dump(result, f, indent=2)

    print("Result saved to /shared/data/latest_price_action.json")

if __name__ == "__main__":
    main()
```

This is the **core trading loop**:  
`OHLC → price_action → decision` (mocked, for now).

---

### 10.6. `risk` agent: position sizing, constraints, and “no‑blow‑up”

#### 10.6.1. Role of the `risk` agent

- Reads:
  - `/shared/data/latest_price_action.json` (from `trader`).  
  - Your **account balance and positions** (from IBKR API or mock).  
- Computes:
  - Max position size (e.g., % of account).  
  - VaR‑style loss‑cap (e.g., don’t lose more than `VaR` in a trade).  
- Outputs:
  - A **`risk.json`** file: allowed direction, size, stop‑level, etc.  
- Stays **isolated** in its own sandbox, talking only via:
  - `/shared/data/`.
  - Approved URLs to IBKR / OpenBB.

#### 10.6.2. Skeleton `risk` Python script

Inside `risk`’s sandbox, create:

**`/sandbox/scripts/risk_engine.py`**

```python
# risk_engine.py
import json
import math
from datetime import datetime

# ----- CONFIG -----
ACCOUNT_RISK_PERCENT = 0.01       # 1% of account per trade
MAX_POSITION_DOLLAR = 10_000      # hard cap per trade
ACCOUNT_BALANCE = 100_000         # mock IBKR balance; replace with actual
TICK_SIZE = 0.25                  # ES tick size
POINT_VALUE = 50                  # ES point value

# --- VaR Lite (Standard Deviation of recent price moves)
def compute_var(bars, percentile=0.01):
    """
    Very simple VaR: assume 1‑tick move is the worst‑case for N bars.
    Replace with real stats if you want.
    """
    if not bars:
        return 10.0  # default safe VaR

    log_returns = []
    for i in range(1, len(bars)):
        p1 = bars[i-1]["close"]
        p2 = bars[i]["close"]
        if p1 > 0:
            log_returns.append(math.log(p2 / p1))

    if not log_returns:
        return 10.0

    mean = sum(log_returns) / len(log_returns)
    std = math.sqrt(sum((r - mean)**2 for r in log_returns) / len(log_returns))
    # 1% quantile approx: -2.33 * std
    var = 2.33 * std
    if var < 0.001:
        var = 0.001
    return var

# --- POSITION SIZING
def compute_position_size(
    account_balance,
    price,
    risk_percent,
    max_dollar,
    var,
):
    """
    Compute position size in contracts.
    """
    # 1. Risk in dollars: account * risk_percent
    risk_dollars = account_balance * risk_percent

    # 2. Maximum loss per contract
    #    (based on VaR-style expected move)
    max_loss_per_contract = price * var

    # 3. How many contracts can we buy with risk_dollars?
    if max_loss_per_contract <= 0:
        contracts = 0
    else:
        contracts = int(risk_dollars / max_loss_per_contract)

    # 4. Cap at dollar max
    contract_value = contracts * price * POINT_VALUE
    if contract_value > max_dollar:
        # Re‑cap
        contracts = int(max_dollar / (price * POINT_VALUE))

    return max(0, contracts)

# --- CORE RISK FUNCTION
def main():
    # 1. Read price_action result
    with open("/shared/data/latest_price_action.json", "r") as f:
        action = json.load(f)

    trend = action["trend"]
    support = action.get("support", [])
    resistance = action.get("resistance", [])
    last_price = action["last_price"]

    # 2. Read OHLC for VaR
    with open("/shared/data/ohlc.json", "r") as f:
        bars = json.load(f)

    # 3. Compute VaR
    var = compute_var(bars)

    # 4. Compute position size
    size = compute_position_size(
        account_balance=ACCOUNT_BALANCE,
        price=last_price,
        risk_percent=ACCOUNT_RISK_PERCENT,
        max_dollar=MAX_POSITION_DOLLAR,
        var=var,
    )

    # 5. Decide direction and constraints
    direction = "flat"
    if trend == "up" and last_price > resistance[-1] * 0.99:
        direction = "long"
    elif trend == "down" and last_price < support * 1.01:
        direction = "short"

    # 6. Output a risk.json for the trader
    risk_plan = {
        "direction": direction,
        "position_size": size,
        "last_price": last_price,
        "trend": trend,
        "var": var,  # "expected move"
        "support": support,
        "resistance": resistance,
        "timestamp": datetime.now().isoformat(),
        "account_balance": ACCOUNT_BALANCE,
        "risk_percent": ACCOUNT_RISK_PERCENT,
        "max_dollar": MAX_POSITION_DOLLAR,
    }

    with open("/shared/data/risk.json", "w") as f:
        json.dump(risk_plan, f, indent=2)

    print("RISK DECISION:")
    print(json.dumps(risk_plan, indent=2))


if __name__ == "__main__":
    main()
```

This is a **complete, minimal risk engine** you can run inside the `risk` agent’s sandbox.

---

### 10.7. `monitor` agent: logs, disk, network, alerts

#### 10.7.1. Role of the `monitor` agent

- Watches:
  - `nemoclaw my-agent logs --follow` style output (via NemoClaw/OpenShell log tail).  
  - Disk usage (`/shared/data`, `/sandbox`).  
  - Memory and CPU from inside the sandbox.  
- Sends:
  - Email or Discord alerts if something goes wrong (e.g., logs show “BLOCKED”, disk full, model down).

#### 10.7.2. Skeleton `monitor` Python script

Inside `monitor`’s sandbox, create:

**`/sandbox/scripts/monitor.py`**

```python
# monitor.py
import json
import os
import time
import psutil
from datetime import datetime
import smtplib
from email.mime.text import MIMEText

# ----- CONFIG -----
CHECK_INTERVAL = 10      # seconds
DISK_THRESHOLD = 0.9     # 90% disk usage → alert
HOST_EMAIL = "your-email@example.com"
DISCORD_WEBHOOK = "https://discord.com/api/webhooks/..."  # optional
LOG_PATH = "/var/log/nemoclaw.log"  # in real life, adapt to your log path

# --- EMAIL ALERT
def send_email_alert(subject, body):
    try:
        msg = MIMEText(body)
        msg["Subject"] = subject
        msg["From"] = HOST_EMAIL
        msg["To"] = HOST_EMAIL

        with smtplib.SMTP("localhost", 25) as s:  # if using a local SMTP
            s.send_message(msg)
        print("Email alert sent.")
    except Exception as e:
        print(f"Failed to send email: {e}")

# --- DISK CHECK
def check_disk():
    usage = psutil.disk_usage("/")
    if usage.percent > DISK_THRESHOLD * 100:
        msg = f"DISK USAGE HIGH: {usage.percent:.2f}% used.\nAvailable: {usage.free / 1024 / 1024:.2f} GB"
        subject = f"[CRITICAL] Disk usage high on {os.uname().nodename}"
        send_email_alert(subject, msg)
        return True
    return False

# --- LOG ANOMALY DETECTION (simplified)
def check_log_for_blocks(log_path):
    if not os.path.exists(log_path):
        print(f"Log file {log_path} not found.")
        return

    with open(log_path, "r", encoding="utf-8", errors="ignore") as f:
        f.seek(0, 2)  # go to end
        while True:
            line = f.readline()
            if not line:
                time.sleep(1)
                continue

            # Simple pattern: if log shows “BLOCKED” or “error”, alert
            line_lower = line.lower()
            if "blocked" in line_lower or "error" in line_lower:
                msg = f"LOG ANOMALY: {line.strip()}\nFile: {log_path}"
                subject = f"[ALERT] NemoClaw security event"
                send_email_alert(subject, msg)
                # Optional: also send to Discord webhook
                # requests.post(DISCORD_WEBHOOK, json={"content": line.strip()})
```

**`check_log_for_blocks`** is a **tail‑style log watcher** you can run in a loop, and **`check_disk`** is a simple disk‑watcher.

---

#### 10.7.3. How to run `monitor` in the sandbox

Create a small wrapper:

**`/sandbox/scripts/run_monitor.sh`**

```bash
#!/bin/bash
# Ensure dependencies installed
pip install psutil requests

python3 /sandbox/scripts/monitor.py
```

Make it executable:

```bash
chmod +x /sandbox/scripts/run_monitor.sh
```

Then, from outside the sandbox:

```bash
nemoclaw monitor onboard
nemoclaw monitor connect
```

and run:

```bash
/sandbox/scripts/run_monitor.sh
```

or wire it into a **cron‑like loop** inside the sandbox:

```bash
cat > /sandbox/scripts/run_monitor_forever.sh << 'EOF'
#!/bin/bash
python3 /sandbox/scripts/monitor.py
EOF

chmod +x /sandbox/scripts/run_monitor_forever.sh
/sandbox/scripts/run_monitor_forever.sh &
```

Now `monitor` is watching logs, disk, and memory continuously.

---

## 11. `cq‑frm`‑style risk thinking and advanced roadmap

### 11.1. What `cq‑frm` means for this stack

`cq‑frm` (CQ Senior‑level Financial Risk Management) is a **risk‑thinking framework**, not a library:

- Focus on **market risk**, **capital‑at‑risk**, and **loss‑constraints**, not just profit.  
- Use **VaR**, **stress tests**, and **scenario analysis** to quantify worst‑case damage.  
- Enforce **hard limits**: daily max‑loss, max‑position size, max‑instruments, etc.

In this guide, we **embed `cq‑frm`** into:

- The `risk` agent (`risk_engine.py`).  
- Policy‑layer limits (allowed URLs, model usage, sandbox isolation).  
- Governance (who can change what, and when).

This turns your NemoClaw‑OpenClaw‑OpenShell stack into a **cq‑frm‑aware, risk‑controlled** trading system.

---

### 11.2. `cq‑frm` in the `risk` agent

You already have a basic `risk_engine.py`. Now enrich it with `cq‑frm`‑style thinking.

#### 11.2.1. CQ‑style VaR and stress‑test

Inside `risk_engine.py`, add:

```python
import math

def cq_compute_var(bars, alpha=0.01):
    """
    CQ‑style parametric VaR approximation.
    """
    if not bars:
        return {"var_percent": 10.0}

    log_returns = []
    for i in range(1, len(bars)):
        b1, b2 = bars[i-1], bars[i]
        if b1["close"] > 0:
            log_returns.append(math.log(b2["close"] / b1["close"]))

    if not log_returns:
        return {"var_percent": 10.0}

    mean = sum(log_returns) / len(log_returns)
    std = math.sqrt(sum((r - mean)**2 for r in log_returns) / len(log_returns))

    # 1‑percentile approx: -2.33 * std
    var = 2.33 * std
    if var < 0.001:
        var = 0.001

    return {
        "var_percent": var,
        "mean_daily_return_percent": mean,
        "volatility_daily_percent": std,
    }

def cq_stress_test_move(bars, shock_percent=0.05):
    """
    Simple CQ‑style stress test.
    """
    if not bars:
        return {"worst_case_loss": 0}

    last = bars[-1]["close"]
    up_shock = last * (1 + shock_percent)
    down_shock = last * (1 - shock_percent)

    return {
        "stress_up_percent": shock_percent,
        "stress_down_percent": shock_percent,
        "price_up_shock": up_shock,
        "price_down_shock": down_shock,
    }
```

And in `main()`:

```python
def main():
    with open("/shared/data/ohlc.json", "r") as f:
        bars = json.load(f)

    cq_var = cq_compute_var(bars, alpha=0.01)
    cq_stress = cq_stress_test_move(bars, shock_percent=0.05)

    risk_plan = {
        # ... your existing fields ...
        "cq_var": cq_var,
        "cq_stress": cq_stress,
        "capital_at_risk_by_trade": ACCOUNT_RISK_PERCENT * ACCOUNT_BALANCE,
    }

    with open("/shared/data/risk.json", "w") as f:
        json.dump(risk_plan, f, indent=2)
```

This embeds **cq‑frm‑style VaR and stress tests** directly into your `risk` agent’s output.

---

#### 11.2.2. CQ‑style hard limits

Add these at the top of `risk_engine.py`:

```python
DAILY_MAX_LOSS_PERCENT = 0.02   # 2% of account per day
MAX_POSITIONS = 10              # total open positions
MAX_CONTRACTS_PER_TRADE = 100
```

Then enforce them:

```python
def cq_safe_position_size(
    account_balance,
    price,
    raw_size,
    daily_loss_at_risk,
):
    if raw_size == 0:
        return 0

    # 1. Daily max loss constraint
    if daily_loss_at_risk > DAILY_MAX_LOSS_PERCENT * account_balance:
        return 0  # no trade

    # 2. Hard contract cap
    if raw_size > MAX_CONTRACTS_PER_TRADE:
        raw_size = MAX_CONTRACTS_PER_TRADE

    return max(0, raw_size)
```

These are **cq‑frm‑style capital‑and‑risk caps** wired into your `risk` agent.

---

### 11.3. `cq‑frm` in policy‑and‑governance (NemoClaw layer)

`cq‑frm` is also about **governance** and **control**, not just math.

In your NemoClaw‑OpenShell setup, enforce:

- **No direct access to sensitive files**:
  - `/etc/passwd`, `~/.ssh`, `~/.aws` must be blocked or read‑only in the policy YAML.  
- **Network‑level rules**:
  - Only allow URLs for approved brokers, data feeds, and logging services.  
  - Use `nemoclaw <agent> policy-add <preset>` carefully; keep a **change log** of who changed what.  
- **Model‑usage rules**:
  - Route through the NVIDIA gateway for PII redaction.  
  - Cap token usage per agent (via model‑provider settings or your own proxy).  

These are **cq‑frm‑style controls over the “infrastructure”** around your agents.

---

## 12. Learning roadmap: from basic to advanced, `cq‑frm`‑aware agents

Use this **six‑phase roadmap** to internalize everything.

### Phase 1: Core NemoClaw & agents (Weeks 1–2)

Goal: master the **command map** and **sandbox intuition**.

#### What you should do:

1. Practise the core commands:

   ```bash
   nemoclaw onboard
   nemoclaw my-agent status
   nemoclaw my-agent connect
   nemoclaw my-agent logs --follow
   nemoclaw my-agent destroy
   nemoclaw stop
   nemoclaw uninstall
   ```

2. For each, write a **one‑sentence mental model** (like “`status` = check if the office is healthy”).  
3. Create a sandbox, run a toy script, then destroy it and repeat.

#### Outcome:

- You can explain, in plain language, what each command does under the hood (NemoClaw, OpenShell, OpenClaw).

---

### Phase 2: Policy, filesystem, network, inference (Weeks 3–4)

Goal: internalize the **four “enforcement” layers**.

#### What you should do:

1. Edit the **policy YAML** once, then:

   ```bash
   nemoclaw my-agent destroy
   nemoclaw onboard
   ```

   to see the policy change take effect.  

2. Use `nemoclaw <agent> policy-add <preset>`:

   - Add `github`, `openai`, `npm`, etc., and watch `logs --follow` show allowed/blocked calls.  

3. For inference:

   - Change model provider in `nemoclaw onboard`.  
   - Watch `status` and `logs` to see how it routes via the gateway.  

#### Outcome:

- You know **which layer blocks what** (filesystem vs network vs policy vs inference).

---

### Phase 3: Skills & tools + `price_action` (Weeks 5–6)

Goal: turn **Python scripts into skills**.

#### What you should do:

1. Build:

   - `price_action` skill (trend, support, resistance).  
   - At least one more skill (e.g., `backtester`, `journal_writer`).  

2. Teach:

   - `trader` to call `price_action`.  
   - `risk` to call a stress‑test or VaR‑style skill.  

3. Run experiments with **mock data** first, then with **real OHLC**.

#### Outcome:

- Each agent is a **tool‑user**, not just a conversational agent.

---

### Phase 4: Multi‑agent & risk‑aware workflow (Weeks 7–8)

Goal: implement the **`dataflow` → `trader` → `risk` → `monitor`** stack.

#### What you should do:

1. On Ubuntu, create:

   ```bash
   mkdir -p ~/nemoclaw-practice/shared/data
   ```

2. Create the agents:

   ```bash
   nemoclaw dataflow onboard
   nemoclaw trader onboard
   nemoclaw risk onboard
   nemoclaw monitor onboard
   ```

3. Paste the scripts into each sandbox:

   - `fetch_ibkr_ohlc.py` in `dataflow`.  
   - `run_trade_cycle.py` in `trader`.  
   - `risk_engine.py` in `risk`.  
   - `monitor.py` and `run_monitor.sh` in `monitor`.  

4. Run them in the background and watch `/shared/data/*.json` evolve.

#### Outcome:

- You have a **risk‑aware, multi‑agent trading pipeline**.

---

### Phase 5: `cq‑frm` + Dynamo‑style automation (Weeks 9–10)

Goal: **embed `cq‑frm` risk thinking** and move to **Dynamo‑style orchestration**.

#### What you should do:

1. Enhance `risk_engine.py` with:

   - VaR, stress tests, daily‑loss caps, max‑positions, instrument‑limits.  

2. Define a **Dynamo / OpenShell app** that runs:

   ```yaml
   services:
     dataflow:
       # ... image, command, volumes ...
     trader:
       # ... image, command, volumes ...
     risk:
       # ... image, command, volumes ...
     monitor:
       # ... image, command, volumes ...
   ```

3. Start thinking of each agent as a **risk‑component**:

   - `dataflow` → data‑risk.  
   - `trader` → strategy‑risk.  
   - `risk` → risk‑engine.  
   - `monitor` → operational‑risk.  

#### Outcome:

- You can **design** risk‑aware systems, not just scripts.

---

### Phase 6: Advanced experimentation (Months 3–6+)

Goal: push everything to production‑edge.

#### What you should do:

1. Swap mocks for **real IBKR + TWS + OpenBB**.  
2. Add:

   - Real‑trade execution via `ib_insync`.  
   - Real‑time VaR‑and‑PnL‑recording.  
   - Daily and monthly risk reports.  

3. Use Dynamo to:

   - Scale across GPU nodes.  
   - Run parallel strategies with different `price_action` logic.  

4. Every month, revisit `price_action` and `risk_engine.py` and ask:

   - “Can I push this closer to cq‑frm best practices?”

#### Outcome:

- A **self‑learning, risk‑aware, multi‑agent trading pipeline** you can confidently run as a core part of your workflow.

---

## 13. Using GitHub for your book and code

Use Git + GitHub to version‑control your book and scripts.

### 13.1. Folder structure

```text
~/nemoclaw-learning-book/
  ├── docs/
  │   ├── index.md
  │   ├── 01-core-concepts.md
  │   └── ...
  ├── scripts/
  │   ├── trading/
  │   │   ├── fetch_ibkr_ohlc.py
  │   │   ├── run_trade_cycle.py
  │   │   └── risk_engine.py
  │   └── ag/
  │       ├── soil_controller.py
  │       └── sensorflow.py
  └── README.md
```

### 13.2. Git setup

```bash
cd ~/nemoclaw-learning-book
git init
git add .
git commit -m "Initial commit: full NemoClaw learning book"

# On GitHub, create a repo named:
#   https://github.com/YOURNAME/nemoclaw-learning-book.git
git remote add origin https://github.com/YOURNAME/nemoclaw-learning-book.git
git push -u origin main
```

### 13.3. Everyday workflow

```bash
cd ~/nemoclaw-learning-book
git add .
git commit -m "Updated docs to improve clarity"
git push
```

This keeps your **book + code versioned, backed up, and sharable**.

---

## 14. How this adapts to other systems (like precision ag)

You can **reuse the same stack** for any domain:

### 14.1. Precision ag example

- `dataflow` → `sensorflow` (sensors + weather + drones).  
- `trader` → `farm_planner` (crop‑health, irrigation, fertilizer).  
- `price_action` → `crop_health` (trend, support, resistance for yields).  
- `risk` → `ag_risk` (yield‑risk, input‑risk, water‑risk, cost‑per‑hectare caps).  
- `monitor` → `farm_monitor` (sensors, energy, water, alerts).

### 14.2. How to port it

1. Reuse:
   - The **same NemoClaw + OpenShell stack**.  
   - The **same multi‑agent pattern** (`dataflow`, `planner`, `risk`, `monitor`).  

2. Swap:
   - IBKR → sensor data, MQTT, farm‑cloud APIs.  
   - `price_action` → `crop_health` or `soil_controller`.  
   - cq‑frm financial risk → cq‑frm yield/input/water risk.

3. Keep:
   - Policy, filesystem, network, inference, skills, and memory patterns the same.

In short:

- **The system** is **general**;  
- **The domain** is **swap‑in‑ready**.

---
