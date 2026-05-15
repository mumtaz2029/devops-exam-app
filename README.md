I have Watched the video : https://youtu.be/BScNFDBdE7M
and learned how to deploy a 3-tier-application (based on docker compose intigrated with Jenkins)

I’ll put it into a clean copy-paste format without extra formatting.


TROUBLESHOOTING AND CHALLENGES FACED DURING 3-TIER APPLICATION DEPLOYMENT

1. SonarQube Version Warning During Deployment

Challenge Faced:
During deployment of the 3-tier application (Frontend, Backend, Database), the SonarQube container used for code quality analysis displayed a red warning banner:

“You’re running a version of SonarQube that is no longer active…”

This indicated that the deployed LTS (Long-Term Support) version was outdated and no longer actively maintained. As a result, the project risked:
- Unsupported security and quality analysis
- Missing updated coding rules
- Reduced CI/CD reliability

Troubleshooting Process:

Step 1: Identify the Root Cause
- Verified the SonarQube version through:
  - Administration → System Info
  - Docker container inspection commands
- Confirmed the container was running an outdated image.

Commands Used:
docker ps
docker exec -it sonar bash

Step 2: Attempted Service Restart
Initially, a container restart was attempted to refresh the service:

docker restart sonar

However, the warning banner still appeared after restart.

Step 3: Root Cause Analysis
After investigation, it was identified that the issue was not related to container state but to the outdated SonarQube image version itself.

Solution Implemented:

Stopped and removed the old container:
docker stop sonar
docker rm sonar

Pulled the latest SonarQube LTS image:
docker pull sonarqube:lts-community

Recreated the container with persistent volumes to preserve data:
docker run -d --name sonar \
-p 9000:9000 \
-v sonarqube_data:/opt/sonarqube/data \
-v sonarqube_extensions:/opt/sonarqube/extensions \
sonarqube:lts-community

Outcome:
- SonarQube upgraded successfully to the latest LTS version
- Red warning banner disappeared
- Code quality scans resumed normally
- CI/CD analysis became more reliable with updated quality and security rules

Impact on the Project:
This fix ensured:
- Reliable static code analysis
- Updated security and code quality checks
- Stable integration with Jenkins CI/CD pipeline
- Better maintainability of the DevOps workflow


2. Docker Authentication Failure in Jenkins Pipeline

Challenge Faced:
During the Docker image build and push stage, the Jenkins pipeline failed with:

ERROR: docker login failed

This prevented the application image from being pushed to Docker Hub.

Troubleshooting Process:

Step 1: Credential Verification
- Investigated Jenkins credentials configuration
- Found that an incorrect Docker Hub password had initially been entered.

Step 2: Manual Authentication Testing
Docker login was tested manually on the EC2 instance:

docker logout
docker login -u mumtaz2029

After entering the correct credentials, authentication succeeded:

Login Succeeded

Step 3: Jenkins Credential Update
The Docker Hub credentials stored in Jenkins were updated through:

Manage Jenkins → Credentials → Global Credentials

Updated:
- Docker Hub username
- Password / Access Token

Solution Implemented:
Correct Docker credentials were configured in Jenkins, allowing secure authentication to Docker Hub.

Outcome:
- Docker authentication issue resolved
- Image build completed successfully
- Docker image successfully pushed to Docker Hub


3. Python Dependency Failure During Docker Build

Challenge Faced:
The Docker image build failed while installing Python dependencies.

Error Observed:
cannot find ft2build.h
ERROR: Failed to build 'reportlab'

Root Cause:
The application was using:

FROM python:3.14-slim

Some required system libraries for reportlab were unavailable in the slim image.

Solution Implemented:
The Docker base image was updated to a stable version:

FROM python:3.10-slim

This ensured better compatibility for dependency installation.

Outcome:
- Docker image built successfully
- Python packages installed without errors
- Backend container became deployable


4. Docker Scout Integration Failure

Challenge Faced:
During image security analysis, the pipeline failed with:

docker: unknown command: docker scout

Root Cause:
Docker Scout worked in the EC2 terminal but was unavailable in the Jenkins runtime environment.

This happened because Jenkins was using a different Docker execution environment.

Solution Implemented:
The Docker Scout stage was temporarily disabled to allow deployment to continue successfully.

Outcome:
- CI/CD pipeline execution resumed
- Deployment stages continued without interruption
- Future enhancement planned for Jenkins-based Docker Scout integration


OVERALL PROJECT OUTCOME

Despite multiple deployment challenges, the 3-tier application CI/CD pipeline was successfully stabilized through systematic troubleshooting and root cause analysis.

Technologies Used:
- Jenkins for CI/CD
- Docker & Docker Compose for containerization
- SonarQube for code quality analysis
- Trivy for vulnerability scanning
- Docker Hub for image registry
- AWS EC2 for hosting

Key Learnings:
- Importance of dependency compatibility in Docker builds
- Credential management in Jenkins pipelines
- Version maintenance for DevOps tools
- Structured troubleshooting in CI/CD environments
- Root cause analysis before implementing fixes

