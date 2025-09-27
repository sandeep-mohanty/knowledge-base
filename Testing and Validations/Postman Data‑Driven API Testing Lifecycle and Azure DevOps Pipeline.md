# 🧪 Comprehensive Tutorial: Postman Data‑Driven API Testing Lifecycle + Azure DevOps Pipeline

This end‑to‑end tutorial consolidates **everything** into one place:

1. Dataset for **Setup Collection** (creating org + groups).
2. Dataset for **Test Collection** (search scenarios, expected outputs).
3. Dataset/structure for **Teardown Collection** (cleanup).
4. Implementation of Postman collections with sample scripts.
5. Azure DevOps pipeline YAML for CI/CD automation.
6. Optional Enhancements (timestamped org names, environment variables, HTML reports). 

---

## 1. Core Test Lifecycle

Your test harness operates in **three phases**:

1. **Setup Collection**  
   - Create a temporary test organization.  
   - Create groups (standard + role‑inheritable) within that org.  

2. **Test Collection**  
   - Find org by name.  
   - Run search API with terms, types, and categories from CSV dataset.  
   - Validate results (group `name`, `category`, count).

3. **Teardown Collection**  
   - Delete the org (and cascading delete clears groups).  

---

## 2. Setup Collection

### 2.1 Dataset: `setup-groups.csv`

```csv
groupName,category
Alpha,standard
alphaTeam,standard
Beta,standard
GammaX,standard
DeltaForce,standard
RoleAdmin,role-inheritable
roleEditor,role-inheritable
RoleViewer,role-inheritable
GammaRole,role-inheritable
deltaOps,role-inheritable
```

### 2.2 Requests

#### **1. Create Org**
- **POST** `{{baseUrl}}/orgs`  
- **Body:**
```json
{
  "name": "TestAutomationOrg_{{timestamp}}"
}
```
- **Pre-request Script:**
```javascript
let ts = Date.now();
pm.variables.set("timestamp", ts);
```
- **Tests tab:**
```javascript
let data = pm.response.json();
pm.collectionVariables.set("orgId", data.id);
pm.collectionVariables.set("orgName", data.name);
```

#### **2. Create Groups**
- **POST** `{{baseUrl}}/orgs/{{orgId}}/groups`  
- **Body:**
```json
{
  "name": "{{groupName}}",
  "category": "{{category}}"
}
```
- **Tests tab (optional store):**
```javascript
let group = pm.response.json();
pm.collectionVariables.set(group.name, group.id);
```

---

## 3. Test Collection

### 3.1 Dataset: `search-scenarios.csv`

```csv
orgName,category,searchType,searchTerm,expectedMatches,expectedCategories
{{orgName}},Standard,StartsWith-CS,Al,"Alpha","standard"
{{orgName}},Standard,StartsWith-CS,al,"",""
{{orgName}},Standard,CaseInsensitiveStartsWith,al,"Alpha,alphaTeam","standard,standard"
{{orgName}},Standard,Exact-CS,Beta,"Beta","standard"
{{orgName}},Standard,Exact-CS,beta,"",""
{{orgName}},Standard,CaseInsensitiveExactMatch,beta,"Beta","standard"
{{orgName}},Standard,Contains-CS,ma,"GammaX","standard"
{{orgName}},Standard,CaseInsensitiveContains,ma,"GammaX,DeltaForce","standard,standard"
{{orgName}},Standard,All,ma,"GammaX,DeltaForce","standard,standard"

{{orgName}},Role-inheritable,StartsWith-CS,Role,"RoleAdmin,RoleViewer","role-inheritable,role-inheritable"
{{orgName}},Role-inheritable,StartsWith-CS,role,"",""
{{orgName}},Role-inheritable,CaseInsensitiveStartsWith,role,"RoleAdmin,RoleViewer,roleEditor","role-inheritable,role-inheritable,role-inheritable"
{{orgName}},Role-inheritable,Exact-CS,RoleAdmin,"RoleAdmin","role-inheritable"
{{orgName}},Role-inheritable,Exact-CS,roleadmin,"",""
{{orgName}},Role-inheritable,CaseInsensitiveExactMatch,roleadmin,"RoleAdmin","role-inheritable"
{{orgName}},Role-inheritable,Contains-CS,amma,"GammaRole","role-inheritable"
{{orgName}},Role-inheritable,CaseInsensitiveContains,amma,"GammaRole","role-inheritable"
{{orgName}},Role-inheritable,CaseInsensitiveContains,delta,"deltaOps","role-inheritable"
{{orgName}},Role-inheritable,All,delta,"deltaOps","role-inheritable"

{{orgName}},All,StartsWith-CS,De,"DeltaForce","standard"
{{orgName}},All,StartsWith-CS,de,"",""
{{orgName}},All,CaseInsensitiveStartsWith,de,"DeltaForce,deltaOps","standard,role-inheritable"
{{orgName}},All,Exact-CS,GammaX,"GammaX","standard"
{{orgName}},All,Exact-CS,gammax,"",""
{{orgName}},All,CaseInsensitiveExactMatch,gammax,"GammaX","standard"
{{orgName}},All,Contains-CS,Team,"alphaTeam","standard"
{{orgName}},All,CaseInsensitiveContains,team,"alphaTeam","standard"
{{orgName}},All,All,role,"RoleAdmin,roleEditor,RoleViewer,GammaRole","role-inheritable,role-inheritable,role-inheritable,role-inheritable"
```

### 3.2 Requests

#### **1. Find Org by Name**
- **GET** `{{baseUrl}}/orgs?name={{orgName}}`
- **Tests tab:**
```javascript
let orgs = pm.response.json();
pm.collectionVariables.set("orgId", orgs[0].id);
```

#### **2. Search Groups**
- **GET** `{{baseUrl}}/orgs/{{orgId}}/groups/search?category={{category}}&searchType={{searchType}}&term={{searchTerm}}`

- **Tests tab:**
```javascript
let json = pm.response.json();

let expectedNames = pm.iterationData.get("expectedMatches")
    .split(",").map(s => s.trim()).filter(s => s.length > 0);

let expectedCategories = pm.iterationData.get("expectedCategories")
    .split(",").map(s => s.trim()).filter(s => s.length > 0);

let actualNames = json.groups.map(g => g.name);
let actualCategories = json.groups.map(g => g.category);

// Validate counts
pm.test("Correct group count", () => {
    pm.expect(actualNames.length).to.eql(expectedNames.length);
});

// Validate names
expectedNames.forEach(exp => {
    pm.test(`Contains group name: ${exp}`, () => {
        pm.expect(actualNames).to.include(exp);
    });
});

// Validate categories
expectedCategories.forEach(expCat => {
    pm.test(`Contains category: ${expCat}`, () => {
        pm.expect(actualCategories).to.include(expCat);
    });
});

// Strict check: no extras
pm.test("No unexpected groups", () => {
    actualNames.forEach(act => {
        pm.expect(expectedNames).to.include(act);
    });
});
```

---

## 4. Teardown Collection

### 4.1 Request: Delete Org
- **DELETE** `{{baseUrl}}/orgs/{{orgId}}`
- **Tests tab:**
```javascript
pm.test("Org deletion succeeded", function () {
    pm.expect(pm.response.code).to.be.oneOf([200,204]);
});
```

---

## 5. Azure Pipeline Integration

### 5.1 Directory Layout
```
/postman
  ├── collections/
  │     ├── SetupCollection.json
  │     ├── TestCollection.json
  │     └── TeardownCollection.json
  ├── data/
  │     ├── setup-groups.csv
  │     └── search-scenarios.csv
  ├── environments/
  │     ├── staging.postman_environment.json
  │     └── prod.postman_environment.json
  └── azure-pipelines.yml
```

### 5.2 `azure-pipelines.yml` (with enhancements)

```yaml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
  env: 'staging'   # Override via pipeline params if needed

steps:
  - task: UseNode@1
    inputs:
      versionSpec: '18.x'
    displayName: 'Use Node.js'

  - script: |
      npm install -g newman
    displayName: 'Install Newman CLI'

  - script: |
      newman run postman/collections/SetupCollection.json \
        -d postman/data/setup-groups.csv \
        -e postman/environments/$(env).postman_environment.json \
        --reporters cli,junit,html \
        --reporter-html-export SetupReport.html \
        --reporter-junit-export SetupResults.xml
    displayName: 'Run Setup Collection'

  - script: |
      newman run postman/collections/TestCollection.json \
        -d postman/data/search-scenarios.csv \
        -e postman/environments/$(env).postman_environment.json \
        --reporters cli,junit,html \
        --reporter-html-export TestReport.html \
        --reporter-junit-export TestResults.xml
    displayName: 'Run Test Collection'

  - script: |
      newman run postman/collections/TeardownCollection.json \
        -e postman/environments/$(env).postman_environment.json \
        --reporters cli,junit,html \
        --reporter-html-export TeardownReport.html \
        --reporter-junit-export TeardownResults.xml
    displayName: 'Run Teardown Collection'

  - task: PublishTestResults@2
    inputs:
      testResultsFiles: '**/*Results.xml'
      testResultsFormat: 'JUnit'
      failTaskOnFailedTests: true
    displayName: 'Publish Newman JUnit Results'

  - publish: $(System.DefaultWorkingDirectory)
    artifact: NewmanReports
    displayName: 'Publish HTML Reports'
```

---

## 6. Optional Enhancements (Included Above)

1. **Unique Orgs**: `TestAutomationOrg_{{timestamp}}` via Pre‑request Script.
2. **Environment Switching**: Select ENV file dynamically (`staging`, `prod`). 
3. **HTML Reports**: `--reporters cli,junit,html` + `--reporter-html-export`.
4. **Artifact Upload**: Publish HTML reports for download/view.

---

# 🎯 Final Outcome

- **Setup Collection** with `setup-groups.csv` → creates test org/groups.
- **Test Collection** with `search-scenarios.csv` → runs all search validations.
- **Teardown Collection** → cleans up org/groups.
- **Azure Pipeline** → runs everything in sequence, publishes results (JUnit + HTML).

Now you have a **complete, production-grade test harness**:  
- **Safe** (isolated data per run)
- **Thorough** (validates names, categories, counts)
- **Automated** (CI/CD with Azure DevOps)
- **Readable** (pretty HTML reports)

This is a framework you can scale for bigger APIs by just adding rows to CSVs or requests to collections. 🚀
