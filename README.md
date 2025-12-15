# Lab 1: Deploy a Change

## Key Outcomes
- No friction for developers  
- Automated changes  

## Overview
In this lab, the user sets up a pipeline integrating a schema source, a target database, change application steps, and built-in change management.  
The user then pushes a new database changelog to Git (e.g., adding a column). This triggers the pipeline and creates a pull request review step for the database change, just like any other code change.

---

## Walkthrough

### Step 1: Create a Deployment Pipeline
1. In the Harness UI, go to **Pipelines**.  
2. Click **Create a Pipeline**, enter a name (**Deploy DB Schema**), and click **Start**.  
3. Click **Add Stage** and choose **Custom Stage**, enter a name (**Deploy Dev**).  
4. Click **Add Step Group**, enter a name (**DB**), then enable **Containerized Execution**.  
5. Select the Kubernetes cluster (**DBDevOps**) where the step should run.

---

### Step 2: Add the Schema Deployment Step
1. Inside your stage, click **Add Step**.  
2. Search for and select **Apply Schema** (under **DB DevOps**), enter a name (**Apply Change**).  
3. In the step configuration:
   - **Schema**: DB  
   - **Database Instance**: DB1  
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
1. Navigate to your pipeline and click **Triggers**, then **New Trigger**.  
2. Choose **Harness** as the trigger type.  
3. Enter a name (**Update**).  
4. Select the repository (**db_changes**) used in your DB Schema definition.  
5. Select **Event (Push)**.  

**Under Conditions:**
- Set the branch name(s) (**main**) to monitor.  
- Under **Changed Files**, enter the path to your changelog file (`changelog.yaml`).  

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

1. Familiar Workflow – Developers stay in Git.
2. Single Source of Truth – All changes are versioned together.
3. Lower Learning Curve – GitOps-aligned workflows.
4. Accelerated Velocity – No manual DB scripts or tickets.
5. Compliance by Design – Automated, policy-driven changes.


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

1. Automatic Rollbacks: Backout plans are pre-validated and ready to run, reducing mean time to recovery (MTTR).
2. No Manual Intervention: Developers do not need to SSH into environments or locate old scripts—rollback is built into the pipeline.
3. Resilience as Default: Pipelines are designed to fail gracefully, keeping environments stable and deploy-ready.


