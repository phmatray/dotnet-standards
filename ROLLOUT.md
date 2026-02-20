# Standards Rollout Guide

Comprehensive guide for rolling out Atypical.Build.Standards across the Matray .NET ecosystem (250+ repositories).

## Rollout Strategy Overview

### Phase 1: Foundation ✅
- **Status:** Complete
- **Deliverable:** NuGet package `Atypical.Build.Standards` v1.0.0
- **Duration:** Complete

### Phase 2: Pilot (Week 1-2)
- **Goal:** Validate standards on 3 Tier 1 repos
- **Repos:** VatBe, Mutty, FormCraft
- **Success criteria:** All builds pass, tests pass, no critical issues

### Phase 3: Tier 1 Expansion (Week 3-4)
- **Goal:** Roll out to remaining 7 Tier 1 repos
- **Success criteria:** 100% of Tier 1 repos standardized

### Phase 4: Mass Rollout (Month 2-3)
- **Goal:** Apply to 80% of active repos (commit within 6 months)
- **Method:** Automated bulk update scripts

### Phase 5: Ecosystem Building (Month 4+)
- **Goal:** Extract common packages (Atypical.Validation, Atypical.Blazor.Components, etc.)
- **Method:** Refactor shared code into NuGet packages

## Phase 2: Pilot Repo Testing

### Pilot Repo 1: VatBe

**Why VatBe?**
- Production web application
- Moderate complexity
- Active development
- Good test coverage

**Steps:**

1. **Clone and branch:**
   ```bash
   git clone https://github.com/phmatray/VatBe.git
   cd VatBe
   git checkout -b adopt-standards
   ```

2. **Backup current configuration:**
   ```bash
   cp Directory.Build.props Directory.Build.props.backup 2>/dev/null || true
   cp .editorconfig .editorconfig.backup 2>/dev/null || true
   ```

3. **Install standards package:**
   ```bash
   dotnet add package Atypical.Build.Standards --version 1.0.0
   ```

4. **Remove conflicting properties from existing files:**
   - Review `Directory.Build.props` (if exists)
   - Remove duplicate settings now in shared props
   - Keep VatBe-specific overrides

5. **Build and measure:**
   ```bash
   dotnet clean
   dotnet build | tee build-output.txt
   ```

6. **Run tests:**
   ```bash
   dotnet test | tee test-output.txt
   ```

7. **Analyze results:**
   ```bash
   # Count warnings
   grep -c "warning" build-output.txt || echo "0 warnings"
   
   # Check for errors
   grep "error" build-output.txt || echo "No errors"
   
   # Test results
   grep "Test Run Successful" test-output.txt || echo "Tests need review"
   ```

8. **Document issues:**
   Create `STANDARDS_ADOPTION.md`:
   ```markdown
   # VatBe Standards Adoption Report
   
   **Date:** YYYY-MM-DD
   **Package Version:** Atypical.Build.Standards 1.0.0
   
   ## Build Results
   - Build: ✅ Success / ❌ Failed
   - Warnings before: X
   - Warnings after: Y
   - Build time before: X seconds
   - Build time after: Y seconds
   
   ## Test Results
   - Total tests: X
   - Passed: X
   - Failed: X (list failures)
   
   ## Issues Encountered
   - Issue 1: Description + resolution
   - Issue 2: Description + resolution
   
   ## Overrides Needed
   - Override 1: Reason
   - Override 2: Reason
   
   ## Recommendations
   - Recommendation 1
   - Recommendation 2
   ```

9. **If successful, commit and push:**
   ```bash
   git add .
   git commit -m "Adopt Atypical.Build.Standards v1.0.0"
   git push origin adopt-standards
   ```

10. **Create pull request:**
    - Title: "Adopt Atypical.Build.Standards v1.0.0"
    - Link to STANDARDS_ADOPTION.md in description
    - Request review from Philippe

### Pilot Repo 2: Mutty

**Why Mutty?**
- Already has .editorconfig
- Smaller scope (validation library)
- Easy to validate

**Steps:** (Same as VatBe, adapted for Mutty)

**Expected Issues:**
- May already have custom .editorconfig (merge needed)
- Roslynator rules might conflict (adjust severity)

### Pilot Repo 3: FormCraft

**Why FormCraft?**
- Complex build setup
- Multiple projects in solution
- Good test coverage
- Revenue-generating library

**Steps:** (Same as VatBe, adapted for FormCraft)

**Expected Issues:**
- Multi-project solution may need per-project overrides
- Blazor-specific settings might need adjustment

## Automated Rollout Script

For Phase 4 (mass rollout), use this script:

```bash
#!/bin/bash
# rollout-standards.sh - Automate standards adoption

REPO_URL=$1
BRANCH_NAME="adopt-standards"
STANDARDS_VERSION="1.0.0"

if [ -z "$REPO_URL" ]; then
  echo "Usage: ./rollout-standards.sh <github-repo-url>"
  exit 1
fi

echo "🚀 Rolling out standards to $REPO_URL"

# Clone repo
REPO_NAME=$(basename "$REPO_URL" .git)
git clone "$REPO_URL" "$REPO_NAME-standards-rollout"
cd "$REPO_NAME-standards-rollout"

# Create branch
git checkout -b "$BRANCH_NAME"

# Backup existing files
cp Directory.Build.props Directory.Build.props.backup 2>/dev/null || true
cp .editorconfig .editorconfig.backup 2>/dev/null || true

# Install standards
dotnet add package Atypical.Build.Standards --version "$STANDARDS_VERSION"

# Build
echo "📦 Building..."
dotnet clean
dotnet build > build-output.txt 2>&1
BUILD_EXIT=$?

# Test
echo "🧪 Testing..."
dotnet test > test-output.txt 2>&1
TEST_EXIT=$?

# Report
echo "📊 Results:"
echo "  Build: $([ $BUILD_EXIT -eq 0 ] && echo '✅ Success' || echo '❌ Failed')"
echo "  Tests: $([ $TEST_EXIT -eq 0 ] && echo '✅ Passed' || echo '❌ Failed')"
echo "  Warnings: $(grep -c 'warning' build-output.txt || echo 0)"
echo "  Errors: $(grep -c 'error' build-output.txt || echo 0)"

# If successful, commit
if [ $BUILD_EXIT -eq 0 ] && [ $TEST_EXIT -eq 0 ]; then
  git add .
  git commit -m "Adopt Atypical.Build.Standards v$STANDARDS_VERSION"
  echo "✅ Ready to push and create PR"
  echo "   Commands:"
  echo "   git push origin $BRANCH_NAME"
  echo "   gh pr create --title 'Adopt Atypical.Build.Standards v$STANDARDS_VERSION' --body 'Automated standards adoption.'"
else
  echo "❌ Issues found. Manual intervention needed."
  echo "   See build-output.txt and test-output.txt for details."
fi
```

**Usage:**
```bash
chmod +x rollout-standards.sh
./rollout-standards.sh https://github.com/phmatray/SomeRepo.git
```

## Bulk Rollout (Phase 4)

### Step 1: Identify Target Repos

```bash
# List all repos with recent activity (6 months)
gh repo list phmatray --limit 1000 --json name,pushedAt --jq '.[] | select(.pushedAt > "2025-08-01") | .name'
```

### Step 2: Categorize Repos

Create `repos.csv`:
```csv
repo_name,tier,complexity,priority,notes
VatBe,1,medium,high,Production app
Mutty,1,low,high,Has .editorconfig
FormCraft,1,high,high,Multi-project
TaLibStandard,1,medium,high,Already good practices
...
```

### Step 3: Automate Rollout

```bash
#!/bin/bash
# bulk-rollout.sh - Roll out to multiple repos

while IFS=, read -r repo_name tier complexity priority notes; do
  if [ "$priority" == "high" ]; then
    echo "Processing $repo_name..."
    ./rollout-standards.sh "https://github.com/phmatray/$repo_name.git"
    sleep 5  # Rate limiting
  fi
done < repos.csv
```

### Step 4: Monitor Progress

Track in GitHub Project:
- **To Do:** Repos not yet processed
- **In Progress:** PRs created
- **Blocked:** Issues need manual fix
- **Done:** PRs merged

## Common Issues & Solutions

### Issue: Build failures after adoption

**Causes:**
- Stricter code style rules
- Nullable reference type warnings
- Missing accessibility modifiers

**Solution:**
1. Review warnings in build output
2. Fix high-severity warnings first
3. Use `<NoWarn>` for false positives (temporarily)
4. Gradually fix remaining warnings

**Example fix:**
```xml
<!-- Temporarily suppress specific warnings -->
<PropertyGroup>
  <NoWarn>$(NoWarn);CS8618;CS8601</NoWarn>
</PropertyGroup>
```

### Issue: Test failures

**Causes:**
- Tests assume specific build configuration
- Test naming conventions conflict

**Solution:**
1. Review test output
2. Check if test files need .editorconfig override
3. Add test-specific overrides:

```ini
# In .editorconfig
[*.Tests/**/*.cs]
dotnet_diagnostic.CA1707.severity = none  # Allow underscores in test names
```

### Issue: MinVer conflicts with existing versioning

**Causes:**
- Repo already uses custom versioning
- Version property manually set

**Solution:**
1. Remove manual `<Version>` from .csproj
2. Let MinVer handle versioning
3. Or skip MinVer:

```xml
<PropertyGroup>
  <MinVerSkip>true</MinVerSkip>
  <Version>1.2.3</Version> <!-- Manual version -->
</PropertyGroup>
```

### Issue: Package metadata conflicts

**Causes:**
- Repo has specific author/company different from defaults

**Solution:**
Override in project's .csproj:
```xml
<PropertyGroup>
  <Authors>Custom Author</Authors>
  <Company>Custom Company</Company>
</PropertyGroup>
```

## Rollback Plan

If standards cause critical issues:

### Quick Rollback (Per Repo)

```bash
# Restore backups
cp Directory.Build.props.backup Directory.Build.props
cp .editorconfig.backup .editorconfig

# Remove package
dotnet remove package Atypical.Build.Standards

# Rebuild
dotnet clean
dotnet build
dotnet test
```

### Full Rollback (Ecosystem-wide)

```bash
# Unpublish package (if needed)
dotnet nuget delete Atypical.Build.Standards 1.0.0 --source nuget.org --api-key <key>

# Revert all PRs
gh pr list --search "author:bmad-agent label:standards" --json number --jq '.[].number' | xargs -I {} gh pr close {}
```

## Success Metrics

Track these metrics during rollout:

### Technical Metrics
- **Adoption rate:** % of repos with standards applied
- **Build success rate:** % of repos that build after adoption
- **Test pass rate:** % of repos with passing tests
- **Warning reduction:** Average % reduction in build warnings
- **Build time impact:** Average % change in build time

### Quality Metrics
- **Code style consistency:** Measured by linter violations
- **Documentation coverage:** % of public APIs documented
- **Test coverage:** Average test coverage %

### Business Metrics
- **Time to new project:** Time to scaffold new project (target: <5 min)
- **Onboarding time:** Time for new dev to understand standards
- **Cross-package usage:** # of repos using other ecosystem packages

## Next Steps After Rollout

1. **Extract common packages:**
   - Atypical.Validation (from Mutty, FormCraft)
   - Atypical.Blazor.Components (from FormCraft, VatBe)
   - Atypical.Testing (shared test utilities)

2. **Create templates:**
   - `dotnet new atypical-lib` — Class library template
   - `dotnet new atypical-blazor` — Blazor app template
   - `dotnet new atypical-api` — Web API template

3. **Continuous improvement:**
   - Gather feedback from devs
   - Update standards based on learnings
   - Version 2.0 with improvements

## Support & Questions

- **Issues with standards:** Open issue in `dotnet-standards` repo
- **Rollout blockers:** Tag @BMAD team (James, Winston, Quinn)
- **Custom requirements:** Discuss in #dotnet-standards channel

---

**Timeline:**
- Week 1-2: Pilot repos (3)
- Week 3-4: Tier 1 expansion (10 total)
- Month 2: Mass rollout (80+ repos)
- Month 3+: Ecosystem building

**Owner:** BMAD Team (James lead, Quinn QA, Barry automation)
