## Lab Pre-requisites

### Changing project
1. From the left hand-side menu click on account
2. From the top navigation bar select projects
3. By clicking the available project everything will be scoped to that project

<img width="410" height="275" alt="image" src="https://github.com/user-attachments/assets/b4e736c3-8b7a-4043-89fa-3a690ff169ff" />

### Changing harness modules
- As part of this lab we will switch between modules several times

In order to switch modules
1. Click on the **nine dot** menu icon
2. Select the module relevant to the step
3. The lab begins in the code repository
<img width="362" height="193" alt="image" src="https://github.com/user-attachments/assets/2d953dd9-b370-4308-9504-a9bf13f7fb9a" />


## Lab 1: Deploy a Change

## Key Outcomes
- No friction for developers  
- Automated changes  

## Overview
In this lab, the user sets up a pipeline integrating a schema source, a target database, change application steps, and built-in change management.  
The user then pushes a new database changelog to Git (e.g., adding a column). This triggers the pipeline and creates a pull request review step for the database change, just like any other code change.

---

## Walkthrough

### Step 1: Create a Deployment Pipeline
1. In the Harness UI, navigate to the **Database DevOps** module
2. From the left menu, select **Pipelines**.  
3. Click **Create a Pipeline**, enter a name

| Input      | Value     | Notes |
| ---------- | --------- | ----- |
| Name       | <pre>`Deploy DB Schema`</pre>||
| Setup | Inline ||

4. Click **Start**.  
5. Click **Add Stage** and choose **Custom Stage**

| Input      | Value     | Notes |
| ---------- | --------- | ----- |
| Stage Name | <pre>`Deploy Dev`</pre>||


6. Click **Add Step** and select **Add Step Group**

| Input      | Value     | Notes |
| ---------- | --------- | ----- |
| Name       | <pre>`DB`</pre>||
| Enable     | Containerized Execution||
| Kuberentes Cluster | k8s-prod || 


---

### Step 2: Add the Schema Deployment Step
1. Inside the step group, click **Add Step**.
2. From the available out of the box steps select **Apply Schema**
3. Configure accordingly


| Input      | Value     | Notes |
| ---------- | --------- | ----- |
| Name       | <pre>`Schema Apply`</pre>||
| Select DB Schema     | DB||
| Database Instance | DB1 || 

4. Click **Apply Changes**.

---

### Step 3: Save and Test the Pipeline
1. Click **Save** to finalize the pipeline.  
2. Click **Run** to manually execute it.  

Verify that:
- The schema changes are applied to the target database.  
- The pipeline completes successfully.

---

### Step 4: Enable Automatic Git-Based Deployment
1. While in the pipeline studio use the top right navigation bar
2. Click **Triggers**, then **New Trigger**.  
3. Choose **Harness** as the trigger type.

| Input      | Value     | Notes |
| ---------- | --------- | ----- |
| Name       | <pre>`Update`</pre>||
| Repository | db_changes||
| Event      | Push ||

4. Click **continue**
5. Under conditions

| Input      | Operator     | Value |
| ---------- | --------- | ----- |
| Branch name(s)       | Equals | <pre>`main`</pre>|
| hanged File | Equals | <pre>`changelog.yaml`</pre>|

6. Click **Continue**, then **Create Trigger**.

---

### Step 6: Push a Changelog to Git
1. Navigate to **Code Repository Module**.  
2. Click into repo **db_changes**.  
3. Click the **Edit** button.  

In your configured Git repo, add a changeSet to alter the schema:

```yaml
 - changeSet:
     id: add-second-email-column
     author: harness-lab
     changes:
       - addColumn:
           tableName: users
           columns:
             - column:
                 name: second_email
                 type: varchar(255)
                 constraints:
                   nullable: true
```


The pipeline will automatically trigger:
The schema change will be applied to the target database.

### Value Callouts
| Aspect | Description |
| :--- | :--- |
| **Familiar Workflow** | Developers stay in Git. |
| **Single Source of Truth** | All changes are versioned together. |
| **Lower Learning Curve** | GitOps-aligned workflows. |
| **Accelerated Velocity** | No manual DB scripts or tickets. |
| **Compliance by Design** | TAutomated, policy-driven changes. |


## Lab 2: Roll Back a Change

### Key Outcomes
- Safe failure handling  
- Pre-validated rollback plans  
- Confidence in change velocity  

---

### Overview
In this lab, the user intentionally deploys a database changelog that introduces a breaking or invalid change (for example, dropping a required column). The pipeline detects the failure during deployment to a non-production environment and automatically triggers a rollback using a predefined backout script.

This simulates a real-world scenario where a change fails validation or breaks application behavior, and highlights how Harness enables fast recovery without manual intervention or firefighting.

---

### Walkthrough

#### Step 1: Add a Rollback Step
1. Navigate to your pipeline and locate the step group containing your **DBSchemaApply** step.  
2. Hover below the apply step and click the **➕** icon to add a new step to the right of the existing step.  
3. In the search bar, type **rollback**, and select the **DBSchemaRollback** step.  

**Step Configuration**
- **DB Schema**: DB  
- **Database Instance**: db1  
- **Rollback Count**: 1  

**Add Conditional Execution**
1. Navigate to the **Advanced** tab in the step configuration.  
2. Click **Conditional Execution**.  
3. Choose **If the previous step fails**.  
4. Click **Apply Changes**, then **Save** the pipeline.

---

#### Step 2: Push a Breaking Change to Git
In your configured Git repository, add a changeSet that attempts to make an invalid change:

```yaml
- changeSet:
    id: add-duplicate-column
    author: harness-lab
    changes:
      - addColumn:
          tableName: users
          columns:
            - column:
                name: id
                type: int
    rollback:
      - sql:
          comment: This is a no-op rollback for the invalid column addition.
          sql: SELECT 1;
```

Commit and push the change to the monitored branch by updating the commit comments and pushing the commit button.

### Result

The pipeline will automatically trigger:
1. The breaking change will attempt to apply and fail.
2. The rollback plan will be executed automatically.
3. The environment will be restored to a stable state.

### Value Callouts

| Aspect | Description |
| :--- | :--- |
| **Automatic Rollbacks** | Backout plans are pre-validated and ready to run, reducing mean time to recovery (MTTR) |
| **No Manual Intervention:** | Developers do not need to SSH into environments or locate old scripts—rollback is built into the pipeline. |
| **Resilience as Default** | Pipelines are designed to fail gracefully, keeping environments stable and deploy-ready. |


## Lab 3: Enforce Governance and Policy

## Key Outcomes

* Targeted **policy-as-code** enforcement
* Standardized review and **approval gates**
* **Risk mitigation** before deployment

---

## Overview

In this lab, the user attempts to push a database changelog that **drops a table**—an action disallowed in this environment. The pipeline evaluates the change using integrated policy-as-code (OPA) rules and immediately blocks execution.

The user reviews the policy failure, understands the violation, and updates the changelog to meet organizational standards before resubmitting.

---

## Walkthrough

### Step 1: Create a Policy

1.  In the left-hand panel, go to **Project Settings**.
2.  Select **Policies**, then click **New Policy**.
    > You may have to click the **X** in the upper right to dismiss a pop-up.
3.  Name the policy: \`Block Destructive SQL\`
4.  In the policy editor, paste the following content and click **Save**:

```opa
package db_sql

rules := [
  {
    "types": ["mssql","oracle","postgres","mysql"],
    "environments": ["prod"],
    "regex": [
      "drop\\s+table",
      "drop\\s+column",
      "drop\\s+database",
      "drop\\s+schema"
    ]
  },{
    "types": ["oracle"],
    "environments": ["prod"],
    "regex": ["drop\\s+catalog"]
  }
]

deny[msg] {
  some i, k, l
  rule := rules[i]
  regex.match(concat("",[".*",rule.regex[k],".*"]), lower(input.sqlStatements[l]))
  msg := "dropping data is not permitted"
}
```

### Step 2: Create a Policy Set and Enforce in Pipeline

1.  Click **Policy Sets**, then click **New Policy Set**.
2.  Name the Policy Set: \`Prevent Destructive Changes\`.
3.  Under Entity Type, select **Custom**.
4.  Under Event, select **On Step**.
5.  Click **Continue**.
6.  Under Policy to Evaluate, click **Add Policy**.
7.  Select the \`Block Destructive SQL\` policy.
8.  Set the evaluation mode to **Error and Exit**.
9.  Click **Apply**, then **Finish**.
10. If necessary, toggle **Enforce**.
11. **In the pipeline:**
    * Open the **Apply Change** step.
    * Switch to the **Advanced** tab and expand the **Policy Enforcement** section.
    * Select \`Prevent Destructive Changes\` policy set from the Project scope.
    * Click **Apply Changes**.
    * **Save Pipeline**.

> The new Policy Set is now active and will enforce the policy at pipeline runtime.

### Step 3: Push a Disallowed Change to Git

1.  In your configured Git repo, add a \`changeSet\` that violates the policy (attempting to drop a table):

  ```yaml
    - changeSet:
        id: 2025-05-21-drop-users-table
        author: harness-lab
        changes:
          - dropTable:
              tableName: users
  ```

2.  Commit and push the change to the monitored branch (main).
3.  Commit and push the change to the monitored branch by updating the commit comments and pushing the commit button.

#### Pipeline Observation

The pipeline will automatically trigger. Observe that:

* The policy is triggered as part of the pipeline execution.
* The execution fails before deployment.
* The pipeline halts with the error:
    > dropping data is not permitted

---

## Value Callouts

| Aspect | Description |
| :--- | :--- |
| **Guardrails, Not Roadblocks** | Policies surface issues early without slowing down developers who follow best practices. |
| **Standardized Governance** | Approval workflows and checks are consistent across teams, environments, and databases. |
| **Policy-as-Code** | Governance is defined in code and version-controlled—just like everything else. |
| **Risk Reduction** | Disallowed or unsafe changes are caught before they affect production. |
| **Scalable Compliance** | Teams can move fast while meeting security and audit requirements at scale. |


## Lab 4: Orchestrate Changes Across Multiple Environments

## Key Outcomes

* **Single pipeline** for multi-env DB deployments
* **Environment-specific guardrails**
* **Reduced handoffs** and manual coordination

---

## Overview

In this lab, the user promotes a database change through multiple environments — **dev, staging, and production** — using a single orchestrated pipeline. Each stage applies the change to a different target database with environment-specific configurations, policies, and approval steps. 

As changes progress, the user can view which schema versions are deployed in each environment directly in the Harness UI — removing the need to manually track or document status.

---

## Walkthrough

### Step 1: Add QA Stage (DB2)

1.  In your existing pipeline from Lab 1, click **Add Stage** and choose **Custom Stage**, enter a name (`Deploy QA`).
2.  Click **Add Step Group**, enter a name (`DB`), then enable the **Containerized Stage**.
3.  Select the Kubernetes cluster (`DBDevOps`) where the step should run.
4.  Inside the stage, click **Add Step** and select **DBSchemaApply** (under DB DevOps).
5.  Name the step: `Deploy Database Schema - QA`.
6.  In the step configuration:
    * Schema: `DB`
    * Database Instance: `DB2`
7.  Click **Apply Changes**.

### Step 2: Add Production Stage (DB3)

1.  In your existing pipeline from Lab 1, click **Add Stage** and choose **Custom Stage**, enter a name (`Deploy Prod`).
2.  Click **Add Step Group**, enter a name (`DB`), then enable the **Containerized Stage**.
3.  Select the Kubernetes cluster (`DBDevOps`) where the step should run.
4.  Inside the stage, click **Add Step** and select **Apply Schema**.
5.  Name the step: `Deploy Database Schema - Prod`.
6.  In the step configuration:
    * Schema: `DB`
    * Database Instance: `DB3`
7.  Click **Apply Changes**.

### Step 3: Save and Run the Pipeline

1.  Click **Save** to finalize the multi-stage pipeline.
2.  In git, remove the table drop change, and the broken change from lab 2, then commit to kick off the pipeline.
3.  Observe the deployment of the previously committed schema change through:
    * Stage 1: DB1 (Dev)
    * Stage 2: DB2 (QA)
    * Stage 3: DB3 (Production)
4.  Verify successful execution in all stages.

### Step 4: View Schema Overview

1.  From the left-hand nav, go to **Overview**.
2.  Observe the green checkmarks next to Dev, QA, and Production, indicating: 
    * Where the change has been applied
    * That each deployment completed successfully
3.  This provides visibility across all environments from a single pane of glass.

---

## Value Callouts

| Feature | Description |
| :--- | :--- |
| **Unified Workflow** | One pipeline governs the full lifecycle of a database change from dev to prod. |
| **Environment-Specific Control** | Each stage can have its own policies, approvers, and rollback settings. |
| **Schema Visibility** | The Harness UI shows which schema changes have been applied where, so teams always know the current state across environments. |
| **Reduced Toil** | No need to manually coordinate between environments or hand off to DBAs. |
| **Production Readiness by Design** | Staged rollouts and approvals ensure only validated changes reach production. |
