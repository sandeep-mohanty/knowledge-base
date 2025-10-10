# Playbook: Partner Admin Role Clean‑Up for Organization Access Groups

This playbook guides an SRE through a **one‑time clean‑up activity** in the production environment.  
The objective: safely remove the **Partner Admin** role from groups under the **Organization Access** category to resolve a customer issue.  

The playbook uses a **two‑phase approach** to ensure safety:
1. **Validation (Dry Run)** → Execute steps with the `USER_ADMIN` role to validate the full workflow.  
2. **Actual Clean‑Up** → Execute the validated workflow with the `PARTNER_ADMIN` role to perform the actual fix.  

---

## Important Context

- A code **fix has been deployed to all environments** (QA and Prod) to prevent addition of the `PARTNER_ADMIN` role to groups under the *Organization Access* tab.  
- Therefore, we **cannot add `PARTNER_ADMIN` roles** to groups anymore for testing purposes.  
- This playbook/script is specifically intended to **clean up historical Partner Admin role assignments** that were created *prior* to the deployment of this fix.  
- For the **dry run**, we must use a different role that is still allowed for assignment and behaves similarly in this workflow.  
- `USER_ADMIN` was selected as the surrogate role for the dry run for this reason.  

---

## Prerequisites

Before starting, ensure you have the following values and artifacts:

### For Production (`prod‑us`)
- **IMS Service Base URL**  
  https://apis.us.iss.lexmark.com/ciam/identity-management-service  

- **IAS Service Base URL**  
  https://apis.us.iss.lexmark.com/ciam/identity-authorization-service  

### For QA (`qa‑us`)
- **IMS Service Base URL**  
  https://apis.qa.us.iss.lexmark.com/ciam/identity-management-service  

- **IAS Service Base URL**  
  https://apis.qa.us.iss.lexmark.com/ciam/identity-authorization-service  

### Common Values (to be provided by SRE)
- **Root Admin Email**  
- **Badge Reader Login Service Client ID**  
- **Badge Reader Login Service Client Secret**  
- **Role To Be Deleted**  
  - Dry run: `USER_ADMIN` (since `PARTNER_ADMIN` cannot be used anymore for testing)  
  - Actual run: `PARTNER_ADMIN` (scripted removal of legacy roles affected pre‑fix deployment)  

### Artifacts from Development Team
- SQL script: `query_to_check_affected_groups.sql`  
- Postman collection: `group_role_delete_collection.json`  
- Postman environment JSON: `group_role_delete_environment.json`  
- Sample query output CSV: `sample_output.csv`  

---

## Phase 1: Validation (Dry Run) with User Admin Role

This phase validates the process workflow with `USER_ADMIN` (since `PARTNER_ADMIN` cannot be re‑added for dry run testing).

### Step 1: Prepare Test Data in Admin UI
- Log in to Admin UI as a **CIAM System Admin**.  
- Under the **Organization** tab, create groups inside at least **3 test orgs**.  
- In each, create 1–2 groups.  
- Add the **User Admin** role to each group.  

### Step 2: Modify the SQL Script
- Open `query_to_check_affected_groups.sql`.  
- Replace all `PARTNER_ADMIN` occurrences with `USER_ADMIN`.  
- Save the modified script.  

### Step 3: Run SQL in Target Environment
- Connect to **prod‑us** database for production dry run, or **qa‑us** if validating in QA.  
- Run the modified SQL.  
- Export result to `affected_groups_user_admin.csv`.  

### Step 4: Validate and Filter CSV
- Compare headers with `sample_output.csv` (must exactly match `Organization_Id, Group_Id`).  
- Remove rows not belonging to your test orgs.  
- Save the filtered CSV.  

### Step 5: Update Postman Environment
- Open `group_role_delete_environment.json`.  
- Set variables for the correct environment (QA or Prod):  
  - `ROOT_ADMIN_EMAIL`  
  - `IAS_URL`  
  - `IMS_URL`  
  - `BADGE_CLIENT_ID`  
  - `BADGE_CLIENT_SECRET`  
  - `ROLE_TO_BE_DELETED = USER_ADMIN`  
- Save JSON and import it into **Postman Desktop**.  

### Step 6: Run the Postman Collection
- Import `group_role_delete_collection.json` into Postman.  
- Select the collection and click **Run** to open Collection Runner.  
- In Runner:  
  - Choose updated environment (QA or Prod).  
  - Under **Data**, click **Select File**, attach `affected_groups_user_admin.csv`.  
  - Click **Run Bulk Delete Roles From Group**.  

**Expectations:**  
- Iteration count = number of rows in CSV (excluding header).  
- All iterations **Pass**.  

### Step 7: Validate in Admin UI
- In QA: verify test groups no longer have `USER_ADMIN`.  
- In Prod (dry run): verify created test orgs lost `USER_ADMIN`.  
- Document success in change ticket.  

---

## Phase 2: Actual Clean‑Up with Partner Admin Role

Once Phase 1 validation is complete, perform clean‑up for the historical `PARTNER_ADMIN` assignments.

### Step 1: Run SQL for Partner Admin
- Use the original `query_to_check_affected_groups.sql` (with `PARTNER_ADMIN`).  
- Connect to **prod‑us** (actual execution) or **qa‑us** (if doing a QA rehearsal).  
- Export results to `affected_groups_partner_admin.csv`.  

### Step 2: Validate CSV
- Ensure headers correctly match `sample_output.csv`.  

### Step 3: Update Postman Environment
- Set environment variable:  
  - `ROLE_TO_BE_DELETED = PARTNER_ADMIN`.  
- Ensure environment is pointing to the correct base URLs (`prod‑us` for production clean‑up).  
- Save changes.  

### Step 4: Execute Clean‑Up
- Go to Collection Runner in Postman.  
- Choose environment.  
- Under **Data**, attach `affected_groups_partner_admin.csv`.  
- Click **Run Bulk Delete Roles From Group**.  
- Confirm:  
  - Iteration count matches CSV rows (excluding header).  
  - All iterations **Pass**.  

### Step 5: Validate Results with SQL
- Re‑run the unmodified SQL script in **prod‑us**.  
- Expect **no rows returned** if clean‑up succeeded.  

---

## Post‑Activity Checklist
- Confirm SQL returns **zero rows**.  
- Save logs, Postman summary, and SQL outputs.  
- Update change ticket with details.  
- Notify stakeholders.  
- Monitor the environment for 24–48h.  

---

## Notes for SRE
- Dry run uses `USER_ADMIN` only, because the fix prevents assigning `PARTNER_ADMIN` to org groups post‑deployment.  
- Always validate dry run before execution of actual clean‑up.  
- Double‑check which environment (`qa‑us` vs `prod‑us`) you’re targeting.  
- Halt and escalate if dry run results are not as expected.  

---

## Summary

- **Dry Run**: Uses `USER_ADMIN` surrogate role in QA or test orgs on Prod since `PARTNER_ADMIN` cannot be added anymore.  
- **Actual Run**: Removes legacy `PARTNER_ADMIN` roles via script + Postman Collection in Prod.  
- **Validation**: SQL must return **no rows** on successful clean‑up.  

This **Playbook** enables safe, auditable removal of incorrect historical Partner Admin assignments with controlled risk.