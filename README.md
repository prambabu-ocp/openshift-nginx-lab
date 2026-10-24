Step 1 — Create your Git repository
Create a repository on GitHub. For your first lab, a public repository is simplest because Jenkins can clone it without Git credentials. Never put your OpenShift token in Git.

Repository name: openshift-nginx-lab

Create this structure:
openshift-nginx-lab/
Jenkinsfile

k8s/
app.yaml

File A: k8s/app.yaml
This file defines the Deployment, Service, and Route as code. Save the following as k8s/app.yaml:

This matches your working setup: Bitnami Nginx listens on port 8080, and the Service sends traffic to that port. OpenShift will assign a hostname to the Route.

Note: This YAML assumes the resources are already compatible with your OpenShift Sandbox permissions. The pipeline will perform a server-side dry run to catch manifest or permission errors before applying changes.
File B: Jenkinsfile
Create a file named exactly Jenkinsfile in the repository root.

This pipeline has separate stages for checkout, validation, authentication, deployment, rollout verification, and an HTTP smoke test.

The smoke test assumes the route is reachable from your Windows PC over HTTP. If your network or Sandbox ingress blocks the request, the deployment might be healthy even though this final test fails.

Step 1.4 — Connect Jenkins to Git
In Jenkins:
1. Open openshift-nginx-deploy.
2. Click Configure.
3. Under Pipeline, change Definition to Pipeline script from SCM.
4. Set SCM to Git.
5. Configure the fields:
Field	Value
Repository URL	https://github.com/YOUR_GITHUB_USERNAME/openshift-nginx-lab.git
Credentials	None for a public repository
Branch Specifier	*/main
Script Path	Jenkinsfile

6. Click Save.
7. Click Build Now and open Console Output.
Jenkins needs the Git plugin and access to Git from its Windows service account. If checkout fails, inspect the first error in Console Output.
Step 2 — Add CI/CD checks
Your new pipeline now includes these checks:
1. Source checkout
CI
Jenkins retrieves the committed Jenkinsfile and OpenShift YAML from Git.

2. Manifest validation
CI
Client-side and server-side dry runs check the manifest before it is applied. Server-side validation also checks the cluster API and permissions.

3. Deployment and rollout
CD
oc apply reconciles the resources from Git, and oc rollout status waits for the Nginx Deployment to become ready.

4. Smoke test
CD
PowerShell sends an HTTP request to the OpenShift Route and fails the build if it cannot obtain an HTTP 200 response.

<img width="885" height="209" alt="image" src="https://github.com/user-attachments/assets/2ebba8b9-ae95-4893-958f-2f4e02adf35c" />


<img width="653" height="228" alt="image" src="https://github.com/user-attachments/assets/4c6ed441-fcb7-4b54-bac8-7bcc818e3040" />
