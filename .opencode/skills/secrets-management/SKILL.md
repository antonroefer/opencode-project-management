---
name: secrets-management
description: Secure handling of secrets, API keys, and credentials. Covers .env files, vault solutions, cloud secret managers, and best practices for never committing secrets. Use when managing secrets, setting up secure credential handling, configuring .gitignore, or when the user mentions secrets, API keys, credentials, .env, vault, or secret management.
---

# Secrets Management Skill

Handle secrets securely across development, CI/CD, and production environments.

## When to Use
- Storing API keys, database passwords, tokens
- Setting up .env files
- Configuring secret management systems (Vault, AWS Secrets Manager)
- Preventing accidental secret commits
- When the user mentions secrets, credentials, or .env files

## The Golden Rule

**NEVER commit secrets to version control.**

## .env Files (Development)

### Basic `.env` File
```bash
# .env (never commit this file!)
DATABASE_URL=postgres://user:password@localhost:5432/mydb
API_KEY=sk-1234567890abcdef
JWT_SECRET=super-secret-jwt-key
```

### `.env.example` (Commit this!)
```bash
# .env.example (template for other developers)
DATABASE_URL=postgres://user:password@localhost:5432/mydb
API_KEY=your-api-key-here
JWT_SECRET=your-jwt-secret-here
```

### Loading .env in Code

**Python (python-dotenv)**
```python
from dotenv import load_dotenv
load_dotenv()

import os
api_key = os.getenv('API_KEY')
```

**Node.js (dotenv)**
```javascript
require('dotenv').config();

const apiKey = process.env.API_KEY;
```

## .gitignore Configuration

Ensure secrets never enter version control:

```gitignore
# Environment variables
.env
.env.local
.env.*.local

# Secrets directory
secrets/
*.pem
*.key
*.p12
*.pfx

# Cloud credentials
.google/
.aws/
.azure/

# IDE secrets
*.secret
```

## Secret Management Systems

### HashiCorp Vault

#### Basic Usage
```bash
# Start Vault dev server
vault server -dev -dev-root-token="root"

# Set a secret
vault kv put secret/myapp/api_key value="sk-123456"

# Get a secret
vault kv get secret/myapp/api_key

# List secrets
vault kv list secret/
```

#### In Application Code
```javascript
// Node.js with node-vault
const vault = require('node-vault')({
  apiVersion: 'v1',
  endpoint: process.env.VAULT_ADDR
});

async function getSecret() {
  const token = await vault.approleLogin({
    role_id: process.env.VAULT_ROLE_ID,
    secret_id: process.env.VAULT_SECRET_ID
  });
  vault.token = token.auth.client_token;
  
  const secret = await vault.read('secret/myapp/api_key');
  return secret.data.value;
}
```

### AWS Secrets Manager

```javascript
// Node.js
const AWS = require('aws-sdk');
const secretsManager = new AWS.SecretsManager();

async function getSecret() {
  const data = await secretsManager.getSecretValue({
    SecretId: 'myapp/api_key'
  }).promise();
  
  return JSON.parse(data.SecretString);
}
```

### Doppler

```bash
# Install Doppler CLI
curl -Ls https://cli.doppler.com/install.sh | sh

# Setup
doppler setup

# Run with secrets
doppler run -- npm start
```

## CI/CD Secrets

### GitHub Actions
```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Use secret
        env:
          API_KEY: ${{ secrets.API_KEY }}
        run: echo "Secret is available"
```

### GitLab CI
```yaml
deploy:
  script:
    - echo "API Key is $API_KEY"
  variables:
    API_KEY: $CI_API_KEY  # From GitLab project settings
```

## Secret Rotation

### Why Rotate?
- Limits blast radius if compromised
- Complies with security standards (SOC 2, PCI-DSS)
- Reduces risk from leaked credentials

### Rotation Strategy
1. **Generate new secret**
2. **Update in secret manager**
3. **Deploy with both old and new** (grace period)
4. **Verify new secret works**
5. **Revoke old secret**

## Detecting Secrets in Code

### Pre-commit Hook (detect-secrets)
```bash
# Install
pip install detect-secrets

# Initialize baseline
detect-secrets scan --baseline .secrets.baseline

# Check in pre-commit
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
```

### GitLeaks
```bash
# Install
brew install gitleaks

# Scan repository
gitleaks detect --source . --verbose

# Scan specific commit range
gitleaks detect --source . --commit-from=HEAD~5 --commit-to=HEAD
```

## What NOT to Do

❌ **Never:**
- Commit `.env` files with real values
- Hardcode API keys in source code
- Print secrets to logs
- Send secrets via chat/email
- Store secrets in plain text files
- Use production secrets in development
- Share secrets in screenshots

## Secure Alternatives

| Instead of... | Use... |
|---------------|--------|
| Hardcoding API key | Environment variable or secret manager |
| Committing `.env` | Add to `.gitignore`, use `.env.example` |
| Sharing keys via Slack | Use secret sharing tools (1Password, Vault) |
| Logging full requests | Redact sensitive fields |
| Storing in config files | Use dedicated secret management |

## Checklist for New Projects

- [ ] `.gitignore` includes `.env`, `*.key`, `secrets/`
- [ ] `.env.example` created with placeholder values
- [ ] Pre-commit hook for secret detection installed
- [ ] CI/CD secrets configured in platform settings
- [ ] Secret management system chosen (Vault, AWS, etc.)
- [ ] Team trained on secret handling practices
- [ ] Secret rotation schedule established

## Verification

After setting up secrets management:
1. Run `git log --all --full-history -S "API_KEY"` to check for past leaks
2. Run `detect-secrets scan` to scan current code
3. Verify `.env` is in `.gitignore`: `git check-ignore .env`
4. Test that app works with secrets from manager
5. Confirm secrets are not in build artifacts
