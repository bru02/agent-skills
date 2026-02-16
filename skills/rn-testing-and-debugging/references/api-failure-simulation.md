---
title: API Failure Simulation
impact: MEDIUM
tags: mitmproxy, redis, api, debugging, testing, proxy, error-simulation
---

# Skill: API Failure Simulation

Simulate API failures and override responses using MITMProxy + Redis + a companion Next.js UI, without modifying the backend.

## Quick Command

```bash
# Install dependencies
brew install mitmproxy redis

# Start Redis
brew services start redis

# Run MITMProxy with response override addon
mitmweb --listen-port 8080 -s ./filter.py
```

## When to Use

- Testing app behavior when APIs return errors (500, 404, 403)
- Verifying error handling UI without backend changes
- Simulating network failures for specific endpoints
- QA needs to reproduce API-related bugs on demand
- Testing API error resilience in React Native or web apps

## Prerequisites

- macOS with Homebrew
- MITMProxy (`brew install mitmproxy`)
- Redis (`brew install redis`)
- Node.js for the companion Next.js UI
- FoxyProxy browser extension (for web app testing)

## Step-by-Step Instructions

### 1. Install and Configure MITMProxy

```bash
brew install mitmproxy redis
brew services start redis
```

Run once to generate CA certificate:
```bash
mitmweb --listen-port 8080
# Close after startup
```

Install the CA cert for HTTPS interception:
```bash
sudo security add-trusted-cert -d -p ssl -p basic \
  -k /Library/Keychains/System.keychain \
  ~/.mitmproxy/mitmproxy-ca-cert.pem
```

### 2. Install Redis for MITMProxy's Python

MITMProxy uses its own Homebrew-managed Python. Install the Redis client into it:

```bash
# Find MITMProxy's Python path
brew info mitmproxy
# Look for the Cellar path, e.g., /opt/homebrew/Cellar/mitmproxy/10.0.0

cd /opt/homebrew/Cellar/mitmproxy/10.0.0/libexec/bin
ls -la  # Find the python symlink

# Navigate to the actual Python installation
# Follow the symlink to find pip3
cd ../../../../../opt/python@3.12/Frameworks/Python.framework/Versions/3.12/bin

# Install redis package
./pip3 install redis --break-system-packages
```

### 3. Create the Filter Addon

Create `filter.py` — the MITMProxy addon that reads override rules from Redis:

```python
import redis

class MyAddon:
    def __init__(self):
        self.r = redis.Redis(host='localhost', port=6379, db=0)

    def response(self, flow):
        urls = [url.decode('utf-8')
                for url in self.r.lrange('override:urls', 0, -1)]
        data = [self.r.hgetall(f'override:{url}') for url in urls]

        match = None
        for node in data:
            if (b'enabled' in node
                    and node[b'enabled'] == b'true'
                    and node[b'urlMatch'].decode('utf-8')
                    in flow.request.url):
                match = node
                break

        if (match is not None
                and flow.request.method != 'OPTIONS'
                and (b'method' not in match
                     or match[b'method'].decode('utf-8')
                     == flow.request.method)):
            flow.response.status_code = int(match[b'statusCode'])
            if b'body' in match:
                flow.response.content = match[b'body']

addons = [MyAddon()]
```

**How it works:**
1. On each intercepted response, check if any Redis-stored `urlMatch` pattern matches the request URL (partial match)
2. If a match is found and the rule is enabled, override the response status code and body
3. Skips OPTIONS requests (CORS preflight)
4. Optionally filters by HTTP method

### 4. Set Up the Companion UI

Clone the companion Next.js app for managing override rules:

```bash
git clone https://github.com/Killavus/request-tester.git
cd request-tester
yarn
yarn dev --port 3002
```

Open `http://localhost:3002` to create, toggle, and manage override rules via a web UI.

### 5. Configure Browser Proxy (Web Apps)

Install FoxyProxy extension for Chrome or Firefox:
1. Add proxy configuration pointing to `localhost:8080`
2. Enable the proxy when testing
3. Create override rules in the companion UI
4. Toggle rules on/off as needed

### 6. Run the Full Stack

```bash
# Terminal 1: Redis (if not already running as service)
brew services start redis

# Terminal 2: MITMProxy with addon
mitmweb --listen-port 8080 -s ./filter.py

# Terminal 3: Companion UI
cd request-tester && yarn dev --port 3002
```

For self-signed certificate APIs, enable "Ignore server certificates" in MITMProxy web UI options.

## Server-Side Interception

For Next.js or Node-based server-side requests, use `node-fetch` with `https-proxy-agent`:

```bash
npm install node-fetch https-proxy-agent
```

```javascript
import fetch from 'node-fetch';
import { HttpsProxyAgent } from 'https-proxy-agent';

const agent = new HttpsProxyAgent('http://localhost:8080');

const response = await fetch('https://api.example.com/data', {
  agent,
});
```

This routes server-side requests through MITMProxy, enabling the same override rules.

## Important Caveats

- **Side effects still execute**: Overriding a DELETE endpoint's response to 500 still deletes the resource server-side. Only the *response* is modified.
- **HTTPS requires CA cert**: Without the CA cert installed, HTTPS interception fails silently.
- **Redis must be running**: The addon crashes if Redis is unavailable.

## Common Pitfalls

- **Forgetting to install Redis in MITMProxy's Python**: The addon imports `redis` from MITMProxy's bundled Python, not system Python
- **Not installing CA certificate**: HTTPS interception won't work without it
- **Expecting request blocking**: This tool overrides *responses*, not requests — the original request still reaches the server
- **FoxyProxy not active**: Ensure the proxy is enabled in the browser extension before testing

## Related Skills

- [detox-vs-maestro.md](./detox-vs-maestro.md) — E2E testing framework comparison
