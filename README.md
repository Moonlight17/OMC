# Jenkins Pipeline Documentation

## Pipeline Steps

### 1. Agent Specification
- **Agent**: Specifies that the pipeline will run on an agent labeled `agent1`.

### 2. Stages Overview
- The pipeline contains the following stages:
  - **Check Basic Curl Tool**
  - **Clone Curl Repository**
  - **Configurate**
  - **Run Tests and Install**
  - **Result Stage**

### 3. Stage Details

#### Check Basic Curl Tool
- **Objective**: Verifies that `curl` is installed on the agent node.

#### Clone Curl Repository
- **Agent**: Executes on the master node.
- **Objective**: Clones the `curl` repository from GitHub.
- **Stash**: Saves the cloned repository as `curl-repo` to share with other stages.

#### Configurate
- **Objective**: Prepares and configures the codebase for building `curl`.
- **Commands**:
  - Unstashes `curl-repo`.
  - Runs `autoreconf -fi` to generate configuration scripts.
  - Configures `curl` with static linking and without SSL.
  - Cleans previous builds and initiates a fresh build.

#### Run Tests and Install
- **Objective**: Installs the built `curl` tool on the system.

#### Result Stage
- **Objective**: Validates the installed version of `curl` and isolates the binary.
- **Commands**:
  - Runs `./src/curl --version` to check the version.
  - Creates an isolated directory, copies the binary into it, and verifies the version from the isolated binary file.

### 4. Post Actions

#### Success
- **Action**: Archives the built `curl` executable as an artifact.
- **Output**: Displays a success message indicating that the build succeeded and the artifact was archived.

#### Failure
- **Action**: Notifies about the build failure via a Telegram message.
- **Output**: Displays an error message and sends a custom Telegram notification including the job name and build number.





---

# Docker Compose and Dockerfile Documentation

This documentation outlines the configuration and purpose of each component in the `docker-compose.yaml` file and the two Dockerfiles (`Dockerfilemaster` and `Dockerfileagent`) used for setting up Jenkins and Jenkins agent services.

## Docker Compose File: `docker-compose.yaml`

### Services

#### 1. Jenkins (Master Node)
- **Build**: Uses `Dockerfilemaster` to build the Jenkins master image.
- **Container Name**: `jenkins`
- **Networks**:
  - `public-network`: Connects to the external network for internet access.
  - `jenkins-network`: Internal network for communication between Jenkins and its agent.
- **Ports**:
  - `8080:8080`: Exposes Jenkins' web UI on port `8080`.
  - `50000:50000`: Exposes Jenkins' agent port for communication with agents.

#### 2. Jenkins Agent
- **Build**: Uses `Dockerfileagent` to build the Jenkins agent image.
- **Container Name**: `jenkins-agent`
- **Depends On**:
  - **jenkins**: Ensures the agent only starts after Jenkins is running.
- **Networks**:
  - `jenkins-network`: Internal network to communicate with Jenkins.
- **Environment Variables**:
  - **JENKINS_URL**: Specifies the Jenkins URL for the agent to connect.
  - **JENKINS_AGENT_SSH_PORT**: Port for SSH connection.
  - **JENKINS_SECRET**: Secret key for agent registration.
  - **JENKINS_AGENT_NAME**: Name of the agent.

### Networks

- **public-network**: External network for Jenkins to access the internet.
- **jenkins-network**: Internal, isolated bridge network for secure communication between Jenkins and the agent.

### Volumes

- **jenkins_home**: Stores Jenkins data and configurations.
- **agent_workdir**: Stores the Jenkins agent's workspace.

## Dockerfile: `Dockerfilemaster` (Jenkins Master)

- **Base Image**: `jenkins/jenkins:lts` - Uses Jenkins Long Term Support (LTS) image for stability.
- **User**: Switches to `root` to perform installations.
- **RUN Command**:
  - Installs `sudo`.
  - Allows the `jenkins` user to run `apt` commands without a password.
  - Cleans up cache after installation.
- **Exposed Ports**:
  - `8080`: Jenkins web UI port.
  - `50000`: Agent communication port.
- **User**: Switches back to `jenkins` user after installation for security.

## Dockerfile: `Dockerfileagent` (Jenkins Agent)

- **Base Image**: `jenkins/inbound-agent:latest` - Uses Jenkins inbound agent image.
- **User**: Switches to `root` for installation and configuration tasks.
- **RUN Command**:
  - Updates packages and installs necessary tools and dependencies for building `curl`.
  - Adds the `jenkins` user to the `sudo` group, allowing passwordless `make` command execution.
  - Cleans up cache after installation.
- **User**: Switches back to `jenkins` user for security.
