name: ultra-finder
description: AI-powered code analysis and pattern discovery engine for automated codebase exploration, vulnerability detection, and legacy pattern identification
version: 2.4.1
author: "OpenClaw Team"
maintainer: "dev@openclaw.io"
tags:
  - analysis
  - ai
  - automation
  - security
  - refactoring
type: skill
license: "MIT"
dependencies:
  - python>=3.9
  - openai>=1.0.0
  - tree-sitter==0.20.4
  - rich>=13.0.0
  - click>=8.0.0
  - pydantic>=2.0.0
required_env_vars:
  - OPENAI_API_KEY
  - ULTRAFINDER_MODEL (default: gpt-4-turbo-preview)
  - ULTRAFINDER_MAX_TOKENS (default: 4000)
  - ULTRAFINDER_TEMPERATURE (default: 0.1)
platforms:
  - linux
  - darwin
  - windows
```

# Ultra Finder - AI-Powered Analysis Engine

## Purpose

Ultra Finder is an intelligent code analysis engine that combines static analysis with LLM-powered pattern recognition to discover vulnerabilities, anti-patterns, deprecated APIs, security issues, and code smells across your entire codebase.

### Real Use Cases

1. **Pre-Commit Security Audit**: Run `ultra-finder scan --type security --staged-only` before every commit to catch SQL injection, XSS, and hardcoded secrets in changed files.

2. **Legacy Modernization**: Detect and replace deprecated patterns: `ultra-finder find --pattern "angular.forEach" --replacement "Array.prototype.map" --dry-run` (migrate from Angular 1.x to modern JavaScript).

3. **Performance Bottleneck Hunt**: Find N+1 queries, synchronous file operations, and blocking calls: `ultra-finder analyze --profile performance --path src/ --since "2 weeks ago"`.

4. **API Migration Assistant**: Identify all usages of deprecated endpoints: `ultra-finder scan --type api-deprecation --target v2.0.0 --report json > migration-plan.json`.

5. **Code Ownership Mapping**: Discover who owns which modules by analyzing commit history and complexity: `ultra-finder analyze --scope ownership --output ownership.md`.

6. **Technical Debt Quantification**: Measure cyclomatic complexity, duplication, and test coverage gaps: `ultra-finder audit --debt --threshold high --export debt-report.xlsx`.

## Scope

### Core Commands

```
ultra-finder scan
  --type <security|performance|deprecation|style|all>
  --path <directory>
  --since <git-time-ref>
  --staged-only
  --exclude <patterns>
  --format <json|sarif|markdown|html>
  --output <file>

ultra-finder find
  --pattern <regex>
  --context-lines <int>
  --replacement <string>
  --dry-run
  --apply
  --backup

ultra-finder analyze
  --profile <security|performance|maintainability|testability>
  --ai-model <gpt-4-turbo|gpt-3.5-turbo>
  --confidence <0.0-1.0>
  --max-findings <int>
  --severity <low|medium|high|critical>

ultra-finder audit
  --debt
  --complexity-threshold <int>
  --duplication-threshold <float>
  --test-coverage <path-to-coverage>
  --export <format>

ultra-finder suggest
  --finding-id <uuid>
  --type <fix|refactor|improve>
  --auto-apply
  --review
```

### Configuration Files

- `.ultrafinder.yaml` - Project configuration
- `.ultrafinder/ignore` - Files/directories to skip (glob patterns)
- `.ultrafinder/rules/` - Custom rule definitions (YAML)

## Detailed Work Process

### Step 1: Environment Setup

```bash
export OPENAI_API_KEY="sk-..."
export ULTRAFINDER_MODEL="gpt-4-turbo-preview"
mkdir -p .ultrafinder/rules
```

Create `.ultrafinder.yaml`:

```yaml
exclude:
  - "**/node_modules/**"
  - "**/dist/**"
  - "**/*.min.js"
  - "**/vendor/**"
rules:
  - "security/*.yaml"
  - "performance/*.yaml"
thresholds:
  complexity: 10
  duplication: 0.2
  test_coverage: 0.8
```

### Step 2: Scanning for Security Issues

```bash
# Scan entire codebase for security vulnerabilities
ultra-finder scan --type security --path . --format sarif --output security.sarif

# Scan only staged files (pre-commit hook)
ultra-finder scan --type security --staged-only --format json

# Exclude test files and vendor code
ultra-finder scan --type security --exclude "**/*.test.*" --exclude "**/vendor/**" --format markdown
```

adar Example Output (JSON):

```json
{
  "findings": [
    {
      "id": "sec-001",
      "type": "sql-injection",
      "file": "src/db/queries.js",
      "line": 45,
      "severity": "critical",
      "description": "User input directly concatenated into SQL query",
      "suggestion": "Use parameterized queries with pg.format()",
      "confidence": 0.96
    }
  ]
}
```

### Step 3: Finding Deprecated Patterns

```bash
# Search for specific deprecated API usage
ultra-finder find --pattern "ReactDOM\.unstable_*" --context-lines 2 --dry-run

# Apply auto-fixes with backup
ultra-finder find --pattern "var " --replacement "const " --apply --backup

# Find Promise without catch
ultra-finder find --pattern "new Promise\(.*\)(?!.*\.catch)" --context-lines 5
```

### Step 4: Performance Analysis

```bash
# Analyze performance bottlenecks
ultra-finder analyze --profile performance --path src/ --confidence 0.85 --max-findings 20

# Focus on synchronous operations in hot paths
ultra-finder analyze --profile performance --ai-model gpt-4-turbo --severity high --output perf-report.html
```

### Step 5: Technical Debt Audit

```bash
# Full debt audit
ultra-finder audit --debt --complexity-threshold 12 --duplication-threshold 0.15 --export xlsx

# With test coverage
ultra-finder audit --debt --test-coverage coverage/coverage.json --export html
```

### Step 6: Custom Rule Creation

Create `.ultrafinder/rules/custom-ssrf.yaml`:

```yaml
rule:
  id: custom-ssrf-detection
  name: "Potential SSRF in HTTP requests"
  pattern: |
    fetch\(|axios\.get\(|http\.request\(
  where:
    - field: "url.source"
      contains: ["req.body", "req.query", "userInput"]
  severity: "high"
  message: "User-controlled URL in HTTP request could lead to SSRF"
  fix: |
    Validate URLs against allowlist or use safe URL construction methods
```

### Step 7: Integration with CI/CD

`.github/workflows/ultra-finder.yml`:

```yaml
- name: Ultra Finder Security Scan
  run: |
    ultra-finder scan --type security --format sarif > results.sarif
- name: Upload SARIF
  uses: github/codeql-action/upload-sarif@v3
  with:
    sarif_file: results.sarif
```

## Golden Rules

1. **Never run ultra-finder with write permissions on production branches without --dry-run first**. Always review findings before applying fixes.

2. **Set confidence threshold >= 0.85 for automated fixes**. Lower confidence requires manual review.

3. **Always backup before --apply**: Ultra Finder automatically creates `backup/ultra-finder-YYYY-MM-DD-HHMM/` with original files.

4. **Exclude generated code**: Mark dist/, build/, node_modules/ in `.ultrafinder.yaml`. Scanning minified code produces false positives.

5. **Use staged-only for pre-commit hooks**: `ultra-finder scan --staged-only` prevents old issues from blocking commits.

6. **Never trust AI suggestions blindly**: Always verify fixes, especially for security issues. Ultra Finder is an assistant, not a replacement for security audits.

7. **Run per-analyzer, not all-at-once**: Separate scans by type (security, performance, deprecation) for clearer reporting.

8. **Pin your OpenAI model**: Set `ULTRAFINDER_MODEL` explicitly to avoid unexpected behavior from model updates.

9. **Monitor token usage**: Large codebases can exceed context windows. Use `--max-files` or split by directory.

10. **Review custom YAML rules with lint**: `ultra-finder rules lint --dir .ultrafinder/rules/` before committing.

## Examples

### Example 1: Finding Hardcoded Secrets

```bash
# Search for common secret patterns
ultra-finder find --pattern "(api_key|secret|password|token)\s*[=:]\s*['\"][^'\"]{8,}['\"]" --context-lines 3
```

Output:

```
File: src/config/production.js:12
┌─────────────────────────────────────┐
│ const API_KEY = 'sk-live-xxx...';   │
└─────────────────────────────────────┘
Suggestion: Move to environment variable: process.env.API_KEY
```

### Example 2: Detecting Async Anti-Patterns

```bash
ultra-finder analyze --profile maintainability --path utils/ --severity medium
```

Output (Markdown):

```markdown
## Async Anti-Pattern Found

**File**: `utils/api.js:67`
**Issue**: Missing await in async function
```javascript
async function getUser(id) {
  const user = fetchUser(id); // Missing await
  return user;
}
```
**Fix**: Add await or remove async if not needed.
**Confidence**: 0.94
```

### Example 3: Performance Bottleneck Detection

```bash
ultra-finder analyze --profile performance --ai-model gpt-4-turbo --severity high --output perf.json
```

```json
{
  "finding": {
    "type": "inefficient-loop",
    "location": "src/reducers/largeArray.js:23",
    "code_snippet": "for (let i = 0; i < bigArray.length; i++) { ... }",
    "issue": "O(n²) nested loop with array.includes()",
    "suggestion": "Convert inner loop to Set lookup (O(1))",
    "estimated_speedup": "~10x for 10k+ items"
  }
}
```

### Example 4: Auto-Fixing with Dry Run

```bash
# Preview changes
ultra-finder find --pattern "\.then\((.*?)\)\.catch\(\)" --replacement "await $1" --dry-run

# Apply fixes
ultra-finder find --pattern "\.then\((.*?)\)\.catch\(\)" --replacement "await $1" --apply
```

### Example 5: Custom Security Rule

`.ultrafinder/rules/ssrf.yaml`:

```bash
ultra-finder scan --type custom --custom-rules ssrf.yaml --format json
```

## Rollback Commands

### Undo All Auto-Fixes

```bash
# If you ran with --backup
ultra-finder rollback --backup-dir backup/ultra-finder-2024-03-15-143022/

# Restore entire backup
cp -r backup/ultra-finder-2024-03-15-143022/* .

# Or selective restore
ultra-finder rollback --finding-id sec-001 --restore-only
```

### Discard Specific Fix

```bash
# List applied fixes
ultra-finder history --format json

# Revert single finding
ultra-finder undo --id perf-042
```

### Remove Ultra Finder Artifacts

```bash
# Clean generated reports and cache
ultra-finder clean --all

# Just clean cache
ultra-finder clean --cache

# Remove backup directories older than 30 days
find backup/ -type d -mtime +30 -exec rm -rf {} \;
```

### Git-Based Rollback

```bash
# If you committed Ultra Finder changes
git revert --no-edit HEAD~2..HEAD  # Revert last 2 commits (scan + fix)

# Or reset to before scan
git reset --hard HEAD~1
```

### Disable Custom Rules

```bash
# Temporarily disable all custom rules
export ULTRAFINDER_DISABLE_CUSTOM=1
ultra-finder scan --type security

# Or move rules out of the way
mv .ultrafinder/rules .ultrafinder/rules.disabled
```

### Full Uninstall/Cleanup

```bash
# Remove Ultra Finder from project
rm -rf .ultrafinder/
rm .ultrafinder.yaml

# Uninstall globally
pip uninstall ultra-finder -y

# Remove API keys from shell history
history -d $(history | grep OPENAI_API_KEY | awk '{print $1}')
```

## Verification Steps

### After Installation

```bash
# Verify CLI works
ultra-finder --version

# Test OpenAI connectivity
ultra-finder ping --model gpt-4-turbo

# Run sample scan on small directory
ultra-finder scan --type security --path test/fixtures/sample --format json
```

### Before Auto-Fixing

```bash
# Dry run first
ultra-finder find --pattern "oldAPI" --dry-run

# Review generated diff
ultra-finder diff --findings sec-001,sec-002

# Check backup integrity
ls -la backup/ultra-finder-*/
```

### After Applying Fixes

```bash
# Re-scan to verify fixes
ultra-finder scan --type security --path . | grep -i "oldAPI"  # Should be empty

# Run tests to ensure no breakage
npm test  # or your test command

# Check diff for unexpected changes
git diff --stat
```

### CI Verification

```bash
# Ensure SARIF output is valid (for GitHub CodeQL)
ultra-finder scan --format sarif > results.sarif
jq . results.sarif  # Should parse without errors

# Verify exit codes
ultra-finder scan --type security --fail-on critical
echo $?  # Should be 1 if critical findings found
```

## Troubleshooting

### API Rate Limits

```bash
# Reduce concurrency
export ULTRAFINDER_MAX_CONCURRENT=2

# Use cheaper model for large scans
ultra-finder scan --type security --ai-model gpt-3.5-turbo
```

### Memory Issues on Large Codebases

```bash
# Limit file count per batch
ultra-finder scan --max-files 100

# Use local model (if available)
export ULTRAFINDER_USE_LOCAL=1
export ULTRAFINDER_LOCAL_MODEL="codellama-7b"
```

### False Positives

```bash
# Adjust confidence threshold
ultra-finder scan --confidence 0.9

# Add false positive to ignore list
echo "src/vendor/jquery.js:100-120" >> .ultrafinder/ignore

# Create suppression rule
cat > .ultrafinder/rules/suppress.yaml << EOF
rule:
  id: suppress-false-positive
  pattern: "legacy\.safe\(.*\)"
  severity: "ignore"
EOF
```

### Slow Scans

```bash
# Cache results
ultra-finder scan --cache

# Only scan changed files since last commit
ultra-finder scan --since HEAD~10

# Exclude large directories
ultra-finder scan --exclude "**/migrations/**"
```

```