# Tutorial: Upgrading Gradle in Spring Boot Projects

## Overview

This tutorial guides you through upgrading Gradle to the latest version in an existing Spring Boot project that uses the Gradle wrapper (`gradlew`). This process is safe, straightforward, and ensures all team members use the same Gradle version.

**Latest Gradle Version:** 9.2.0 (as of October 2025)

***

## Prerequisites

Before upgrading:
- Your project has existing `gradlew` (Unix) or `gradlew.bat` (Windows) files
- You have terminal/command line access to your project root directory
- You're using Spring Boot (compatible with Gradle 4.x+)
- **Recommended:** Your project is under Git version control

***

## Step 1: Backup Current State (Recommended)

Before upgrading, create a safety checkpoint:

### Option A: Using Git (Recommended)

If using Git, ensure your current state is committed:

```bash
# Check status
git status

# Commit any pending changes
git add .
git commit -m "Pre-Gradle upgrade checkpoint"

# Optional: Create a backup branch
git branch backup-gradle-upgrade
```

### Option B: Manual Backup

If not using version control, backup these files:

```bash
# Create backup directory
mkdir ../gradle-backup

# Copy wrapper files
cp -r gradle ../gradle-backup/
cp gradlew ../gradle-backup/
cp gradlew.bat ../gradle-backup/
```

***

## Step 2: Check Your Current Gradle Version

Verify your current Gradle version:

```bash
./gradlew --version
```

**Note:** Record this version number for potential rollback.

***

## Step 3: Upgrade to Latest Gradle

### Option A: Upgrade to Latest Version (Recommended)

Run the wrapper task to upgrade to the latest stable Gradle version:

```bash
./gradlew wrapper --gradle-version latest
./gradlew wrapper
```

### Option B: Upgrade to Specific Version

If you prefer a specific version (e.g., Gradle 9.2.0):

```bash
./gradlew wrapper --gradle-version 9.2.0
./gradlew wrapper
```

### Why Run the Command Twice?

**Critical:** Always run the wrapper task **twice**:
1. **First run**: Updates `gradle-wrapper.properties` with the new Gradle version
2. **Second run**: Regenerates wrapper files (`gradlew`, `gradlew.bat`, `gradle-wrapper.jar`) optimized for the new version

---

## Step 4: Verify the Upgrade

Confirm the upgrade was successful:

```bash
./gradlew --version
```

You should see the new Gradle version displayed.

***

## Step 5: Test Your Build

Run a clean build to ensure everything works:

```bash
./gradlew clean build
```

If you encounter issues, see the **Rollback** section below.

***

## What Files Are Updated?

The wrapper task modifies these files in your project:

```
gradle/wrapper/
├── gradle-wrapper.jar       # Updated wrapper JAR
└── gradle-wrapper.properties # Updated distribution URL

gradlew                       # Updated Unix shell script
gradlew.bat                   # Updated Windows batch file
```

***

## Rollback: Reverting to Previous Gradle Version

If something goes wrong, you can easily revert to your previous Gradle version.

### Method 1: Using Git (Recommended)

If you committed before upgrading, revert all wrapper changes:

```bash
# View changed files
git status

# Revert specific wrapper files
git checkout HEAD -- gradle/wrapper/gradle-wrapper.jar
git checkout HEAD -- gradle/wrapper/gradle-wrapper.properties
git checkout HEAD -- gradlew
git checkout HEAD -- gradlew.bat

# Or revert all changes at once
git checkout HEAD -- gradle/ gradlew gradlew.bat
```

Verify the rollback:

```bash
./gradlew --version
```

### Method 2: Using Git Commit History

If you already committed the upgrade:

```bash
# Find the commit hash before upgrade
git log --oneline

# Revert to specific commit for wrapper files
git checkout <commit-hash> -- gradle/ gradlew gradlew.bat

# Commit the revert
git add gradle/ gradlew gradlew.bat
git commit -m "Revert Gradle upgrade to version X.Y.Z"
```

### Method 3: Downgrade Using Wrapper Task

Run the wrapper task with your previous version number:

```bash
# Example: Downgrade to Gradle 8.5.0
./gradlew wrapper --gradle-version 8.5.0
./gradlew wrapper
```

Verify the downgrade:

```bash
./gradlew --version
./gradlew clean build
```

### Method 4: Manual Restoration from Backup

If you created a manual backup:

```bash
# Restore from backup directory
cp -r ../gradle-backup/gradle ./
cp ../gradle-backup/gradlew ./
cp ../gradle-backup/gradlew.bat ./

# Verify restoration
./gradlew --version
```

### Method 5: Quick Manual Fix

Edit `gradle/wrapper/gradle-wrapper.properties` and change the version:

```properties
distributionUrl=https\://services.gradle.org/distributions/gradle-8.5.0-bin.zip
```

Then regenerate wrapper files:

```bash
./gradlew wrapper
./gradlew wrapper
```

***

## Troubleshooting Build Failures

### Issue: Dependency Resolution Errors

**Symptoms:**
```
Could not resolve all dependencies for configuration ':compileClasspath'
```

**Solution:**
```bash
# Clear Gradle cache
rm -rf ~/.gradle/caches/

# Retry build
./gradlew clean build --refresh-dependencies
```

### Issue: Plugin Compatibility Errors

**Symptoms:**
```
Plugin [id: 'org.springframework.boot', version: 'X.Y.Z'] was not found
```

**Solution:**
Check Spring Boot Gradle plugin compatibility with new Gradle version. You may need to update your plugin version in `build.gradle`:

```groovy
plugins {
    id 'org.springframework.boot' version '3.2.0'
    id 'io.spring.dependency-management' version '1.1.4'
}
```

### Issue: Deprecated API Warnings

**Symptoms:**
```
Deprecated Gradle features were used in this build
```

**Solution:**
Run build with stack trace to identify deprecated usage:

```bash
./gradlew clean build --warning-mode all --stacktrace
```

Update deprecated syntax based on warnings, or revert to previous Gradle version if immediate fix isn't feasible.

***

## Spring Boot Compatibility

### Spring Boot 3.x
- **Minimum Gradle:** 7.5+
- **Minimum Java:** 17
- **Recommended Gradle:** 8.x or 9.x

### Spring Boot 2.x
- **Minimum Gradle:** 4.x+
- **Minimum Java:** 8 or 11
- **Recommended Gradle:** 7.x or 8.x

**Note:** Modern Gradle versions (8.x, 9.x) maintain backward compatibility with older Spring Boot versions.

***

## Step 6: Commit Changes to Version Control

After successful upgrade and testing, commit all modified wrapper files:

```bash
git add gradle/wrapper/gradle-wrapper.jar
git add gradle/wrapper/gradle-wrapper.properties
git add gradlew
git add gradlew.bat
git commit -m "Upgrade Gradle wrapper to version 9.2.0"
git push
```

This ensures all team members use the same Gradle version when they pull your changes.

***

## Alternative: Manual Upgrade Method

If you need manual control, you can edit `gradle/wrapper/gradle-wrapper.properties` directly:

```properties
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-9.2.0-bin.zip
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
```

**Warning:** This method only updates the distribution URL. It doesn't regenerate wrapper scripts, so the wrapper task method is strongly preferred.

***

## Advanced Troubleshooting

### Complete Gradle Cache Cleanup

If builds fail mysteriously after upgrade:

```bash
# Stop all Gradle daemons
./gradlew --stop

# Clean local build directory
./gradlew clean

# Remove Gradle cache (Linux/Mac)
rm -rf ~/.gradle/caches/
rm -rf ~/.gradle/wrapper/

# Remove Gradle cache (Windows)
# rmdir /s %USERPROFILE%\.gradle\caches
# rmdir /s %USERPROFILE%\.gradle\wrapper

# Rebuild project
./gradlew build
```

### Verify Wrapper Integrity

Check if wrapper files are correctly updated:

```bash
# Display wrapper properties
cat gradle/wrapper/gradle-wrapper.properties

# Verify wrapper JAR exists
ls -lh gradle/wrapper/gradle-wrapper.jar

# Check wrapper scripts are executable (Linux/Mac)
ls -la gradlew
```

### Check for Breaking Changes

Before upgrading across major versions, review release notes:

- **Gradle 8.x to 9.x:** https://docs.gradle.org/current/userguide/upgrading_version_8.html
- **Gradle 7.x to 8.x:** https://docs.gradle.org/current/userguide/upgrading_version_7.html
- **Gradle 6.x to 7.x:** https://docs.gradle.org/current/userguide/upgrading_version_6.html

---

## Testing Checklist

Before committing the upgrade, verify:

- [ ] `./gradlew --version` shows correct version
- [ ] `./gradlew clean build` completes successfully
- [ ] All unit tests pass: `./gradlew test`
- [ ] Integration tests pass (if applicable)
- [ ] Application starts successfully: `./gradlew bootRun`
- [ ] No unexpected deprecation warnings
- [ ] Build time is reasonable (not significantly slower)
- [ ] Docker builds work (if using Docker)
- [ ] CI/CD pipelines pass (check before merging)

***

## Best Practices

1. **Regular Updates**: Upgrade Gradle quarterly or when major versions release
2. **Test Thoroughly**: Always run full test suite after upgrading
3. **Commit Before Upgrade**: Always commit clean state before upgrading
4. **Team Communication**: Notify team before pushing wrapper upgrades
5. **CI/CD Pipelines**: Verify builds pass in CI before merging
6. **Documentation**: Keep track of Gradle versions used across environments
7. **Gradual Upgrades**: Don't skip multiple major versions at once
8. **Read Release Notes**: Review breaking changes before upgrading

***

## Quick Reference Card

| Command | Purpose |
|---------|---------|
| `./gradlew --version` | Check current Gradle version |
| `./gradlew wrapper --gradle-version latest` | Upgrade to latest |
| `./gradlew wrapper --gradle-version X.Y.Z` | Upgrade to specific version |
| `./gradlew wrapper --gradle-version X.Y.Z` (twice) | Downgrade to specific version |
| `./gradlew clean build` | Test build after upgrade |
| `git checkout HEAD -- gradle/ gradlew gradlew.bat` | Revert upgrade using Git |
| `./gradlew --stop` | Stop all Gradle daemons |
| `rm -rf ~/.gradle/caches/` | Clear Gradle cache |

***

## Emergency Rollback Procedure

If your build is completely broken after upgrade:

1. **Immediately stop any running Gradle processes:**
   ```bash
   ./gradlew --stop
   ```

2. **Revert all wrapper files using Git:**
   ```bash
   git checkout HEAD -- gradle/ gradlew gradlew.bat
   ```

3. **Clean and rebuild:**
   ```bash
   ./gradlew clean build
   ```

4. **If still broken, clear Gradle cache:**
   ```bash
   rm -rf ~/.gradle/caches/
   ./gradlew clean build
   ```

5. **Verify version:**
   ```bash
   ./gradlew --version
   ```

***

## Version History Template

Keep track of your upgrades in your project README:

```markdown
## Gradle Version History
- 2025-10-31: Upgraded to Gradle 9.2.0 (Success)
- 2024-08-15: Upgraded to Gradle 8.5.0 (Success)
- 2024-01-10: Upgraded to Gradle 8.0.0 (Reverted - plugin compatibility issues)
```

***

## Summary

Upgrading Gradle using the wrapper ensures consistency across development, testing, and production environments. The two-step wrapper command is the safest and most reliable method. Always commit your changes before upgrading, verify compatibility with your Spring Boot version, test thoroughly, and commit all wrapper files to version control.

If something goes wrong, you have multiple rollback options: Git revert, downgrade using wrapper task, or manual restoration. Having a clean Git state before upgrading makes rollback trivial and risk-free.

**Remember:** 
- Run `./gradlew wrapper` twice—the first updates the properties, the second regenerates optimized wrapper files!
- Always commit before upgrading—it makes rollback instant and safe!
- Test thoroughly before pushing to ensure your team doesn't face issues!
