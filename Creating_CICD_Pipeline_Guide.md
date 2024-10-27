
# Creating a CI/CD Pipeline with AWS CodePipeline, CodeBuild, and CodeDeploy

This guide explains how to set up a CI/CD pipeline using AWS CodePipeline, CodeBuild, and CodeDeploy with GitHub as the source repository.

---

### Prerequisites

1. **Create and Upload the Source Code to a Repository**  
   - Ensure your source code is in a repository supported by CodeBuild and CodePipeline, such as **CodeCommit**, **Amazon S3**, **Bitbucket**, or **GitHub**.
   - Your source code should include:
     - A **buildspec.yml** file for CodeBuild.
     - An **appspec.yml** file for CodeDeploy.
     - A **app.py** file for to run in application.

---

### Step 1: Create the Pipeline

1. **Create a New Pipeline**
   - Go to **AWS CodePipeline** and create a new pipeline.
   - Name your pipeline, e.g., **MyAppPipeline**.

2. **Pipeline Configuration**
   - **Execution Mode**: Select the **Superseded** default option.
   - **Service Role**: Choose to create a **new service role** to allow necessary permissions.
   
3. **Source Stage**
   - **Source Provider**: Select **GitHub** and connect to your GitHub account.
   - **Repository**: Choose your repository and default branch.
   - **Output Artifact**: Use the **CodePipeline Default** format.
   - **Trigger**: Set the trigger event to **Push**.
   - **Filter**: Include branches such as `main`.

4. **Build Stage**
   - **Build Provider**: Select **AWS CodeBuild**.
   - **Project Name**: Use **MyAppBuild** (created in Step 2).
   - **Build Type**: Choose **Single build** for simple applications. For more complex applications (e.g., with backend API, web frontend), consider using **Batch build**.
   - **Input Artifact**: Select **SourceArtifact** as the input artifact.

5. **Deploy Stage**
   - **Deploy Provider**: Select **AWS CodeDeploy**.
   - **Input Artifact**: Use the **BuildArtifact** generated by CodeBuild.
   - **Application Name**: Specify **MyAppDeploy** (created in Step 3).
   - **Deployment Group**: Use **MyAppDeployGroup**.

6. **Review and Create**
   - Review your configurations and create the pipeline.

---

### Step 2: Create a Build Project in CodeBuild

1. **Create a New Build Project**
   - Go to **AWS CodeBuild** and create a project named **MyAppBuild**.
   - Set the source to **GitHub** and connect your GitHub account.

2. **Environment Configuration**
   - Configure the environment as needed, and create a new service role if prompted.
   
3. **BuildSpec File**
   - Select the option to use a **buildspec.yml** file. Ensure this file is in your repository.

4. **Artifacts**
   - No need to select an artifact manually. CodePipeline will automatically create an artifact after the first build execution.

---

### Step 3: Create an Application in CodeDeploy

1. **Create an Application**
   - In **AWS CodeDeploy**, create a new application named **MyAppDeploy**.
   - **Compute Platform**: Choose **EC2** as the platform.

2. **Deployment Group**
   - Create a deployment group named **MyAppDeployGroup**.
   - Assign a **service role** with the necessary permissions for CodeDeploy.

3. **Deployment Type**
   - Select **In-place Deployment** for the deployment type.

4. **Environment Configuration**
   - Choose **Amazon EC2 Instances**.
   - **Tag Group**: Define a tag group for identifying instances:
     - **Key**: `Name`
     - **Value**: `CI-CD-test`
   
   > **Note**: Ensure any EC2 instances launched use these same tags.

5. **Additional Configuration**
   - **Load Balancing**: Enable if needed.
   - **Review and Create**: Complete the configuration to create your deployment group.

---

### Possible Issues and Solutions

- **Missing Source Files**: Ensure all required files, like `buildspec.yml` and `appspec.yml`, are present in the source repository.
  
- **CodeDeploy Not Creating Output Artifact**:
  - You may encounter the error:
    ```
    CodePipeline: Insufficient permissions Unable to access the artifact with Amazon S3 object key
    ```
  - To resolve this, ensure the following lines are included in your `buildspec.yml`:

    ```yaml
    artifacts:
      files:
        - '**/*'  # Adjust to include all necessary files
      discard-paths: no  # Maintains the folder structure
    ```

- **Lifecycle Hook Errors**:
  - Example error:
    ```
    Script does not exist at specified location: /opt/codedeploy-agent/deployment-root/...
    ```
  - Ensure scripts specified in lifecycle hooks exist at the correct paths and that the EC2 instances are running.

---

### Testing the Pipeline

1. **Make Changes in Local Code**:
   - Update `app.py` on your local machine with the following:
     ```python
     print("Hello, AWS CodePipeline!")
     ```

2. **Commit and Push**:
   - Use Git to commit and push the change:
     ```bash
     git add app.py
     git commit -m "Updating app.py"
     git push origin main
     ```

3. **Observe Pipeline Execution**:
   - **AWS CodePipeline** will detect the new commit and automatically trigger the pipeline stages (Source, Build, Deploy).
   - You can view real-time progress in the CodePipeline console.

---

### References and Resources

- [CodePipeline Documentation](https://docs.aws.amazon.com/codepipeline/)
- [CodeBuild Documentation](https://docs.aws.amazon.com/codebuild/)
- [CodeDeploy Documentation](https://docs.aws.amazon.com/codedeploy/)

--- 