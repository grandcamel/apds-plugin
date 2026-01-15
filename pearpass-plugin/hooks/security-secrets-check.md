---
event: PreToolUse
tools:
  - Bash
match_command:
  - "git commit"
  - "git add"
---

# Security Check: Potential Secrets in Commit

Before committing, verify that no secrets or credentials are being added to version control.

## Automated Checks to Run

Before proceeding with this commit, check for:

### 1. Hardcoded Secrets
```bash
# Check staged files for potential secrets
git diff --cached --name-only | xargs grep -l "apiKey\|secretKey\|password\s*=\|API_KEY\|SECRET" 2>/dev/null
```

### 2. Environment Files
```bash
# Ensure .env files are not staged
git diff --cached --name-only | grep -E "\.env$|\.env\."
```

### 3. Credential Patterns
```bash
# Check for common credential patterns
git diff --cached | grep -E "(password|secret|api_key|token)\s*[:=]\s*['\"][^'\"]+['\"]"
```

### 4. Private Keys
```bash
# Check for private key content
git diff --cached | grep -E "BEGIN (RSA |DSA |EC |OPENSSH )?PRIVATE KEY"
```

## Files That Should NEVER Be Committed

| Pattern | Reason |
|---------|--------|
| `.env` | Environment variables with secrets |
| `.env.*` | Environment-specific secrets |
| `*.pem` | Private keys |
| `*.key` | Private keys |
| `credentials.json` | Service account credentials |
| `*secret*` | Any file with "secret" in name |

## Gitignore Verification

Ensure `.gitignore` includes:
```
.env
.env.*
*.pem
*.key
credentials.json
```

## If Secrets Are Detected

1. **Remove from staging:**
   ```bash
   git reset HEAD <file>
   ```

2. **Add to .gitignore:**
   ```bash
   echo "<pattern>" >> .gitignore
   ```

3. **Use environment variables instead:**
   ```javascript
   // ❌ Bad
   const apiKey = 'sk-1234567890'

   // ✅ Good
   const apiKey = process.env.API_KEY
   ```

4. **For development, use .env.example:**
   ```bash
   # .env.example (commit this)
   API_KEY=your-api-key-here

   # .env (do NOT commit)
   API_KEY=sk-actual-secret-key
   ```

## PearPass-Specific Concerns

### Never Commit:
- Vault encryption keys
- Master password test fixtures with real passwords
- P2P pairing secrets
- Browser extension signing keys

### Safe to Commit:
- Test fixtures with dummy data
- Mock encryption results
- Example configurations

## Recommendation

If you're unsure whether content contains secrets, run:
```bash
/pearpass-security-audit credentials
```

Proceed only after verifying no secrets are included in the commit.
