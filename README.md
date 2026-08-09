# Jenkins Shared Library & Parameterised Pipelines

This project is a hands-on lab that covers two common real-world Jenkins skills:

1. **Building a reusable Jenkins Shared Library** so multiple pipelines can share the same CI/CD code instead of copy-pasting it.
2. **Creating a parameterised pipeline** that can deploy to different environments (Dev, QA, Staging, Production) using a single Jenkinsfile.

---

## Prerequisites

Before starting, make sure you have:

- A running Jenkins server (local or on a server/EC2 instance)
- A GitHub account (to host the shared library and sample project repos)
- Docker installed on the Jenkins agent/node (for the build/push steps)
- Basic familiarity with Git and Jenkins pipeline syntax (helpful but not required)

---

## Task 1: Reusable Jenkins Shared Library

### What is a Shared Library?

Think of a Shared Library as a "toolbox" of common pipeline functions (like Git checkout, build, and Docker push) that you write **once** and reuse in **any number of Jenkinsfiles**. This avoids duplicating the same pipeline code in every project.

### Step 1: Create the Shared Library Repository

1. Create a new GitHub repository, for example `jenkins-shared-library`.
2. Inside it, create the following folder structure. This is the standard layout Jenkins expects:

```
jenkins-shared-library/
└── vars/
    ├── gitCheckout.groovy
    ├── mavenBuild.groovy
    ├── dockerBuildImage.groovy
    └── dockerPushImage.groovy
```

> The `vars/` folder holds "global variables" — each `.groovy` file becomes a callable function you can use directly in a Jenkinsfile.

<img src="screenshots/shared-library-repo-structure.png" />

### Step 2: Write the Reusable Functions

Each file in `vars/` defines one reusable step. Here are simple examples:

**`vars/gitCheckout.groovy`** — reusable Git checkout step

```groovy
def call(String repoUrl, String branch = 'main') {
    git branch: branch, url: repoUrl
}
```

**`vars/mavenBuild.groovy`** — reusable Maven build step

```groovy
def call() {
    sh 'mvn clean package'
}
```

**`vars/dockerBuildImage.groovy`** — reusable Docker image build step

```groovy
def call(String imageName, String tag = 'latest') {
    sh "docker build -t ${imageName}:${tag} ."
}
```

**`vars/dockerPushImage.groovy`** — reusable Docker image push step

```groovy
def call(String imageName, String tag = 'latest') {
    sh "docker push ${imageName}:${tag}"
}
```

Commit and push these files to your `jenkins-shared-library` repository.

<img src="screenshots/shared-library-groovy-files.png" />

### Step 3: Configure the Library in Jenkins

1. Go to **Manage Jenkins → System**.
2. Scroll down to **Global Pipeline Libraries**.
3. Click **Add** and fill in:
   - **Name**: `jenkins-shared-library` (this is the name you'll reference with `@Library`)
   - **Default version**: `main` (your branch name)
   - **Retrieval method**: Modern SCM → Git
   - **Project Repository**: the URL of your shared library repo
4. Click **Save**.

<img src="screenshots/global-pipeline-libraries-config.png" />

Inside your ec2-unbuntu create a folder and you can generate a basic Maven application with:

```bash
mvn archetype:generate \
  -DgroupId=com.example \
  -DartifactId=demo-app \
  -DarchetypeArtifactId=maven-archetype-quickstart \
  -DarchetypeVersion=1.5 \
  -DinteractiveMode=false
```

Then move the generated pom.xml and src into the repository root if needed.


### Step 4: Use the Library in a Jenkinsfile

In your actual project repository, create a `Jenkinsfile` that pulls in the shared library with `@Library` and calls the reusable functions:

```groovy
@Library('jenkins-shared-library') _

pipeline {
    agent any

    parameters {
        string(name: 'REPO_URL', defaultValue: 'https://github.com/your-username/your-app.git', description: 'Git repository to build')
        string(name: 'REPO_BRANCH', defaultValue: 'main', description: 'Branch to checkout')
        string(name: 'IMAGE_NAME', defaultValue: 'your-app', description: 'Docker image name')
    }

    stages {
        stage('Checkout') {
            steps {
                gitCheckout(params.REPO_URL, params.REPO_BRANCH)
            }
        }
        stage('Build') {
            steps {
                mavenBuild()
            }
        }
        stage('Docker Build') {
            steps {
                dockerBuildImage(params.IMAGE_NAME, "${env.BUILD_NUMBER}")
            }
        }
        stage('Docker Push') {
            steps {
                dockerPushImage(params.IMAGE_NAME, "${env.BUILD_NUMBER}")
            }
        }
    }
}
```

> The `@Library('jenkins-shared-library') _` line at the top tells Jenkins to load the library you configured in Step 3. The underscore `_` is required syntax when you're not importing a specific class.


### Step 5: Prove It Works Across Multiple Projects

To demonstrate reusability, use the **same shared library** in a second, different project repository — just repeat Step 4 with a different `Jenkinsfile` (pointing to a different app repo). Both pipelines should build successfully, calling the exact same underlying functions without any duplicated code.

<img src="screenshots/multiple-pipelines-using-shared-library.png" />

---

## Task 2: Parameterised Pipelines for Multi-Environment Deployments

### Why Parameters?

Instead of creating four separate Jenkinsfiles (`Jenkinsfile-dev`, `Jenkinsfile-qa`, `Jenkinsfile-staging`, `Jenkinsfile-prod`), you can create **one** Jenkinsfile and let the user pick the target environment at build time using a dropdown parameter.

### Step 1: Add a Choice Parameter

Add a `parameters` block to your pipeline with an `Environment` choice parameter:

```groovy
pipeline {
    agent any

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['Dev', 'QA', 'Staging', 'Production'],
            description: 'Select the environment to deploy to'
        )
    }

    stages {
        stage('Show Selected Environment') {
            steps {
                echo "Deploying to environment: ${params.ENVIRONMENT}"
            }
        }
        stage('Deploy') {
            steps {
                script {
                    echo "Running deployment steps for ${params.ENVIRONMENT}..."
                    // Add environment-specific deployment logic here
                }
            }
        }
    }
}
```

### Step 2: Run the Pipeline and Select a Parameter

1. Go to your pipeline job in Jenkins.
2. Click **Build with Parameters** (this option appears automatically once a `parameters` block exists).
3. Select an environment from the **ENVIRONMENT** dropdown (e.g., `QA`).
4. Click **Build**.

<img src="screenshots/build-with-parameters-dropdown.png" />

### Step 3: Verify the Console Output

Open the build's **Console Output**. You should see the selected environment printed clearly, confirming the parameter was passed correctly into the pipeline.

<img src="screenshots/console-output-selected-environment.png" />

---

## Project Structure

```
.
├── jenkins-shared-library/        # Shared library repo (Task 1)
│   └── vars/
│       ├── gitCheckout.groovy
│       ├── mavenBuild.groovy
│       ├── dockerBuildImage.groovy
│       └── dockerPushImage.groovy
├── project-a/
│   └── Jenkinsfile                # Uses shared library
├── project-b/
│   └── Jenkinsfile                # Also uses shared library (proves reusability)
└── parameterised-pipeline/
    └── Jenkinsfile                # Task 2 — multi-environment deployment
```

---

## Key Takeaways

- Shared Libraries eliminate duplicate pipeline code across projects — write once, use everywhere.
- The `@Library` annotation is how a Jenkinsfile "imports" the shared library.
- Parameterised pipelines let one Jenkinsfile serve multiple environments, avoiding pipeline sprawl.
- Always echo/log selected parameters early in the pipeline — it makes debugging and auditing much easier.

---

## Notes

- Replace all placeholder repo URLs (`https://github.com/your-username/...`) with your actual repository links.
- Screenshot filenames above are placeholders — replace them with your actual captured screenshots, keeping the same filenames or updating the `<img src="">` paths accordingly.
