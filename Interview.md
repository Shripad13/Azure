# Access the blob from Virtual Machine
1. Scenario: How does the Azure DevOps CI/CD Pipeline look in your organization ?

In our organization we have microservices architecture based application and we implemented the CI using Azure pipelines and CD using the GitOps approach.
Continuous Integration (CI):

Dev --> Git --> Unit Test --> Static Code Analysis --> Build --> N2N Testing --> Docker image --> Push to artifactory (ACR/ECR/Dockerhub)


Developer commits code change on Azure repo/github, 
Triggers on code changes.
Clones code from repository.
Runs unit tests and static code analysis through SonarQube
Builds artifacts (e.g., compiled code, container images) and output is in Docker image.
Scanning the image using Snyk/Trivy.
Stores artifacts in Azure Pipelines artifacts for deployment. (Azure Container Registry)
Update script for newly created image in the K8s YAML manifest file or Helm chart


Continuous Delivery (CD):

Repo --> Build --> Push to ECR --> Update (shell) script) --> Repo(looks for Image & integrates to GitOps/ArgoCD) --> GitOps --> K8s

Git repositories as the single source of truth.
COntinuous reconcilation of ArgoCD to Git and push image to K8s CLuster
If someone changes the k8s cluster then ArgoCD detects as Drift
GitOps does not allow manual change to k8s cluster
 
 Git --<-- GitOps(ArgoCD) -->-- K8s

CD Triggers on successful CI completion or manually.
ArgoCD picks up the new image and deploys into AKS Cluster
Deploys artifacts to designated environments (staging, production).
Runs environment-specific tests (e.g., integration, acceptance).
Approvals or gates can be implemented before deployment.
Optionally, rolls back deployments if issues arise.

2. Securing Sensitive Information in Pipelines:
Scenario: You need to securely store API keys and other secrets used in your pipeline tasks. How would you ensure their protection while maintaining pipeline functionality?

Answer: Explain using Azure Key Vault to store secrets and access them using managed identities or service connections with minimal privileges. Emphasize avoiding hardcoding secrets in the pipeline script.

3. Integrating Azure Container Registry (ACR) with Pipelines:
Scenario: Your application uses Docker containers. How would you integrate ACR with Azure Pipelines for building, pushing, and deploying container images?

Answer: Describe the process of configuring Docker tasks in the pipeline to build images, authenticate with ACR using service connections, push images to the registry, and deploy them to specific environments.

stages:
  stage:
    jobs:
Use connection store    

4. Debugging Pipeline Failures:
Scenario: Your pipeline consistently fails at a specific stage. How would you approach troubleshooting and identifying the root cause of the issue?

Answer: Highlight utilizing built-in debugging tools like logs, pipeline diagnostics, and Azure Monitor, alongside manual code review and environment checks. Mention potential causes like resource constraints, task configuration errors, or infrastructure issues.

5. Handling Code Merges and Rollbacks in Pipelines:
Scenario: You discover a critical bug in the recently deployed production environment. How would you leverage Azure Pipelines for a rollback and ensure safe merging of a fix?

Answer: Explain using deployment environments and conditional triggers to target specific environments. Discuss leveraging branching strategies and continuous deployment practices to revert changes and integrate a fix seamlessly.

6. Utilizing Azure Runners for Self-Hosted Environments:
Scenario: Your company has specific infrastructure requirements and needs to run pipelines on self-hosted machines. How would you leverage Azure Runners for this purpose?

Answer: Discuss configuring and managing self-hosted runners, ensuring security considerations like network isolation and access control. Mention using the appropriate runner OS and tools based on your project needs.


7. Implementing Role-Based Access Control (RBAC) in Pipelines:
Scenario: Your team has various roles with different access needs. How would you configure RBAC within Azure Pipelines to ensure users have appropriate permissions?

Answer: Explain leveraging built-in roles and custom definitions to grant access to pipelines, repositories, and resources. Highlight the importance of least privilege and separation of duties principles.

8. Automating Infrastructure Provisioning with Pipelines:
Scenario: You want to automate infrastructure provisioning and deployment alongside your application code. How would you integrate infrastructure as code (IaC) tools like Terraform with Azure Pipelines?

Answer: Discuss using tasks like Azure Resource Manager or Terraform tasks to manage infrastructure creation and deletion within the pipeline workflow. Mention benefits like faster deployments and improved consistency.

9. Maintaining Pipeline Security Throughout the CI/CD Process:
Scenario: How would you ensure overall security within your Azure Pipelines throughout the CI/CD process, from code building to deployment?

Answer: Discuss a holistic approach, including secure code practices, vulnerability scanning, container image scanning, service principal usage with least privilege, and regular pipeline audits.


10. What are Azure DevOps service connections?

"A service connection provides Azure DevOps pipelines with an authenticated way to access external services such as Azure subscriptions, container registries or other platforms.

 I prefer a dedicated identity with only the permissions required for the pipeline."

11. Azure RBAC - Authorization
Azure RBAC controls who can perform which actions on which Azure resources.

Conceptually, an RBAC assignment consists of a security principal, a role definition and a scope.

For example, instead of giving someone Owner access to an entire subscription, I might give the required team Contributor or a more specific custom role at the appropriate resource-group scope.

I always try to follow least privilege and assign access at the lowest practical scope."

10. Azure Policy vs RBAC

RBAC controls who can perform an action.
Azure Policy controls what configurations or resources are allowed or required.

11. Key Vault - Secrets
"I prefer using managed identity wherever supported.

The application or Azure resource gets an identity, and that identity is granted only the required Key Vault permissions.

The application then accesses the secret without requiring us to embed credentials in source code or pipeline YAML.

Access is controlled through appropriate authorization and monitored through Azure logging."

12. Managed Identity - Authentication
    "Managed identity provides an Azure-managed identity for a resource so that it can authenticate to Azure services without us having to store credentials.

There are two common types — system-assigned and user-assigned.

System-assigned identity is tied to the lifecycle of the resource, while 
user-assigned identity is a separate Azure resource that can be associated with multiple supported resources.

I prefer managed identity when supported because it reduces credential management and supports a stronger least-privilege model."

13. Azure Policy scenario - Governance
"I would implement an Azure Policy requiring the mandatory tags.

Depending on the requirement, the policy can audit existing resources, deny non-compliant resource creation or modify resources where supported.

I would first use audit mode to understand the existing compliance state before moving to deny mode, because immediately enforcing deny could unexpectedly block existing deployment processes."

12. How do you retrieve a secret in YAML?
"I don't hardcode the secret in YAML.

pulling secrets directly from Azure Key Vault into your pipeline job using the **AzureKeyVault@2** task
 retrieve the required secret at runtime and make it available only to the required task.

Ensure your Azure Resource Manager Service Connection has Get and List permissions on your Key Vault access policy or Azure RBAC


"If it's a deployment-time secret, I can integrate the pipeline with Key Vault. If it's required by the application at runtime, I'd preferably make the application retrieve it securely using managed identity where the architecture supports it."

13. How do you secure Azure DevOps?

 requires a defense-in-depth approach covering identity, pipeline security, source code protection, and audit governance.

1. Identity & Access (The Perimeter)
• Centralised Auth: First, I always link Azure DevOps to Microsoft Entra ID rather than using personal Microsoft accounts. This ensures that when an employee leaves the company, their access is revoked instantly.
• Least Privilege: I apply strict Conditional Access policies (like enforcing MFA) and strictly enforce the Principle of Least Privilege, making sure users are scoped properly and never blindly thrown into the 'Project Administrators' group.


2. Pipeline Security (The Execution)
• Project Isolation: By default, I turn on settings to limit the job authorization scope to the current project. This prevents a compromised pipeline in Project A from modifying resources or code in Project B.
• Secret Management: I completely eliminate long-lived passwords and secrets. For cloud deployments, I use Workload Identity Federation for Azure Service Connections, which leverages short-lived, automated OIDC tokens. For application secrets, I map them dynamically from Azure Key Vault rather than hardcoding them in pipeline variables.
• Deployment Gates: I enforce Approvals and Checks on Environments and Service Connections so code cannot touch production without manual sign-off or passing compliance checks.


3. Repository & Code Security (The Supply Chain)
• Branch Policies: I enforce strict branch policies on main branches. Nobody pushes code directly; it must go through a Pull Request (PR) with mandatory peer reviews and a passing build validation pipeline.

4. Governance & Auditing (The Trail)
• Finally, for monitoring, I configure Azure DevOps Audit Streams to forward all organization logs into an Azure Log Analytics Workspace. This lets us track high-risk changes, like who modified permissions or deleted a repository, in real-time."


1.  how do you write in approvals in Azure DevOps pipelines YAML file?
Step 1: Configure the Approval in the UI
Step 2: Reference the Environment in Your YAML File
Once the UI check is created, lock down a specific stage or job in your YAML pipeline by targeting that environment using the environment property.
When the pipeline reaches this stage, it will automatically pause and wait for the designated users to click "Approve".

trigger:
- main

stages:
- stage: BuildStage
  jobs:
  - job: BuildJob
    steps:
    - script: echo "Building the application..."

- stage: DeployToProd
  dependsOn: BuildStage
  jobs:
  - deployment: DeployJob
    pool:
      vmImage: 'ubuntu-latest'
    # This environment name MUST perfectly match the name created in the Azure DevOps UI
    environment: 'Production' 
    strategy:
      runOnce:
        deploy:
          steps:
          - script: echo "Deploying to Production because approval was granted!"



1. What is the fundamental hierarchy of an Azure DevOps YAML pipeline?
• Pipeline: The top-level workflow definition file (azure-pipelines.yml).
• Stage: A logical boundary in the pipeline (e.g., Build, Test, Deploy) that lets you group jobs and set environment-level gates.
• Job: An execution unit that runs on a single agent or as an agentless task. Jobs can run sequentially or in parallel.
• Step: A linear sequence of operations inside a job, which can be either a Script (Bash, PowerShell, CLI) or a Task (pre-packaged building block like PublishBuildArtifacts@1 or Docker@2).


2. What is the difference between a task, a script, and a template?
• Task: A pre-packaged, versioned script or action provided by Microsoft or the Marketplace (e.g., AzureWebApp@1, DotNetCoreCLI@2). It abstracts underlying commands and handles input validation cleanly.
• Script: Raw shell, Bash, or PowerShell commands (script:, bash:, powershell:) used when you need custom logic or fine-grained CLI control that doesn't have a dedicated task.
• Template: A reusable YAML snippet (template: my-steps.yml) used to DRY (Don't Repeat Yourself) pipeline code across multiple repositories or builds.

3. How do you pass data or variables from one task or job to another?
  Q..  How do you set a variable in one task and read it in a subsequent task—or across different jobs?
  Across Jobs (Job A to Job B): You must map the output variable from the source job:

# Job A sets it
- script: echo "##vso[task.setvariable variable=myVar;isOutput=true]Hello"
  name: setvarStep

# Job B reads it
steps:
- script: echo $(jobA.setvarStep.myVar)
  dependsOn: JobA
  condition: succeeded()

4. How do you handle secrets securely inside pipeline tasks?

• Never hardcode secrets in YAML files.
• Define them in Variable Groups linked to Azure Key Vault, or mark them as secret: true in the pipeline variables UI.
• To map a secret variable into a task or script, pass it explicitly via the env: 
# mapping:
steps:
- script: echo "Running secure command"
  env:
    MY_SECRET_TOKEN: $(MySecretVariableFromVault)

5. What happens when a task fails midway through a job, and how do you customize that behavior?

• By default, if a task fails, all subsequent tasks in that job are skipped, and the job fails.
• You can override this behavior using the condition: property with built-in functions like always(), failed(), or succeededOrFailed():

- script: echo "This runs even if the previous task failed"
  condition: always()


6. Scenario-Based: "A build task takes 20 minutes to run. How would you troubleshoot and optimize it?"

• Analyze Timestamps: Use the Azure DevOps pipeline run UI to inspect the duration of each individual task (e.g., package restore vs. test execution).
• Enable Caching: Use the Cache@2 task to cache heavy dependencies (like ~/.m2 for Maven, node_modules for Node.js, or NuGet packages) across runs.
• Parallelize Jobs: Split unit tests, integration tests, and linting into separate parallel jobs across multiple agent slots instead of running them serially in a single job.
• Upgrade Agents: Move from shared/standard Microsoft-hosted agents to higher-spec private/self-hosted agents or scale-set agents if disk I/O or CPU is the bottleneck.


7. How to run the parallel jobs in YAML?
   
To run unit tests, integration tests, and linting in parallel, you define them as separate jobs within the same stage and configure them to have no dependencies on each other. 

  By default, jobs inside a stage run concurrently if your Azure DevOps organization has enough parallel agent slots available.   



trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

stages:
- stage: QualityGates
  displayName: 'Code Quality and Testing'
  jobs:

  # Job 1: Runs Linting
  - job: RunLinting
    displayName: 'Code Linting'
    steps:
    - script: |
        echo "Running ESLint / Code analysis..."
        # npm run lint
      displayName: 'Execute Linter'

  # Job 2: Runs Unit Tests (Runs in parallel with Linting)
  - job: RunUnitTests
    displayName: 'Unit Testing'
    steps:
    - script: |
        echo "Running Unit Tests..."
        # npm run test:unit
      displayName: 'Execute Unit Tests'

  # Job 3: Runs Integration Tests (Runs in parallel with Linting and Unit Tests)
  - job: RunIntegrationTests
    displayName: 'Integration Testing'
    steps:
    - script: |
        echo "Running Integration Tests..."
        # npm run test:integration
      displayName: 'Execute Integration Tests'


# lets i have project A inside that repository X and artefact generates in project A in repository X , how can i write in Azure DevOps pipeline YAML to use in Project B repository Y?

you must define Project A's pipeline as a pipeline resource in your Project B YAML file

Here is how you can write the YAML pipeline for Project B, followed by the mandatory security permissions you need to enable in Azure DevOps for it to work.


Step 1: Write the YAML in Project B (Repository Y)

Add this structure to your azure-pipelines.yml file inside Project B. This configuration establishes a reference to Project A and uses the download task to fetch the generated artifact.

trigger:
  - main  # Triggers when code is pushed to Project B's repository Y

# 1. Define the upstream pipeline resource from Project A
resources:
  pipelines:
    - pipeline: UpstreamArtifactResource # Local alias identifier for this resource
      project: Project A                 # EXACT name of the source Azure DevOps Project
      source: Name-Of-Pipeline-In-ProjectA # EXACT name of the pipeline that produces the artifact
      trigger: true                      # Optional: Set to true if Project B should run automatically whenever Project A finishes

pool:
  vmImage: 'ubuntu-latest'

jobs:
  - job: UseArtifact
    displayName: 'Download and Use Artifact from Project A'
    steps:
      # 2. Download the artifact using the resource alias
      - download: UpstreamArtifactResource
        artifact: drop  # Replace with the exact name of the artifact published in Project A
        displayName: 'Download Project A Artifact'

      # 3. Verify or use the downloaded artifact
      - script: |
          echo "Listing downloaded files:"
          ls -R $(Pipeline.Workspace)/UpstreamArtifactResource
        displayName: 'List Downloaded Artifact Files'


When you use the - download keyword:
• The artifact is automatically downloaded into the standard workspace directory:
$(Pipeline.Workspace)/<pipeline-resource-alias>/<artifact-name>
• Following the example above, your files will be available at:
$(Pipeline.Workspace)/UpstreamArtifactResource/drop


Step 2: Fix the Security Permissions (Mandatory)
By default, Azure DevOps isolates project scopes for security. If you run the pipeline in Project B now, it will fail with an "Artifact not found" or "Access Denied" error. You must grant Project B permission to read from Project A


1. Disable "Limit Job Authorization Scope"

1. Go to Project Settings in Project B.
2. Click on Settings under the Pipelines menu section.
3. Scroll down to Limit job authorization scope to current project for non-release pipelines and toggle it OFF.
(Note: If this option is locked, your Organization Administrator must turn it off at the Organization Settings level).

2. Grant the Build Service Account Access to Project A

1. Navigate to Project A -> Project Settings.
    Go to Pipelines -> Settings.
2. Under Limit job authorization scope..., ensure it is toggled OFF here as well so Project A can share data out.
3. If you encounter tight project security restrictions, go to Project A -> Project Settings -> Permissions -> Users. Find the user named Project B Build Service (<Your-Org-Name>) and explicitly grant it View project-level information and Read permissions for the pipelines.