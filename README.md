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

