# CNTF - Custom UERANSIM Docker Image

## Purpose
This source code repository stores the configurations to create a custom UERANSIM docker image. This custom image includes scripts from Puppeteer (https://github.com/puppeteer/puppeteer), which enables the UERANSIM test-suite to simulate a UE performing normal user activities (e.g. watching YouTube, browsing websites, etc.) while connected to a 5g network.

## AWS - Elastic Container Registry (ECR)
* In the event that the custom image(s) is not stored in an existing image repository, please create a new AWS ECR repository for it to be uploaded & stored: 
    * Step 1 - Create an ECR repository: https://docs.aws.amazon.com/AmazonECR/latest/userguide/repository-create.html
    * Step 2 - In the ".gitlab-ci.yml" file, change line 5 to represent the repository registry URL for your AWS ECR repository

## Deployment
Prerequisites:

* *Please ensure that you have configured the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-quickstart.html) to authenticate to an AWS environment where you have adequate permissions to create an EKS cluster, security groups and IAM roles* 
* *Please ensure that the pipeline in the "CNTF-Main" repository has been successfully deployed, as this ensures that all necessary components are available to support the execution of scripts in this repository.*  
* *Please ensure that there is an existing image repository where these custom docker images can be stored.*

Steps:
1. [Mirror](https://docs.gitlab.com/ee/user/project/repository/mirror/) this repository OR connect it [externally](https://docs.gitlab.com/ee/ci/ci_cd_for_external_repos/) to Gitlab 
2. Perform a "Git clone" of this repository on your local machine
3. Set up a private Gitlab runner on the CNTF EKS cluster (***Note:*** *You only need to do this process once, this runner can be used by the other CNTF repositories you execute*):
    * In Gitlab, on the left side of the screen, hover over "settings" and select "CI/CD"
    * Next to "Runners" select "expand"
    * Unselect "Enable shared runners for this project"
    * Click "New project runner"
    * Under "Operating systems" select "Linux"
    * Fill out the "Tags" section and select "Run untagged jobs"
    * Scroll to the bottom and select "Create runner"
    * Copy and save the "runner token" listed under "Step 1"
    * Select "Go to runners page", you should now see your runner listed with a warning sign next to it under "Assigned project runners"
    * On your local terminal:
        * Install the helm gitlab repository: `helm repo add gitlab https://charts.gitlab.io`
        * intialize helm (for helm version 2): `helm init` 
        * create a namespace for your gitlab runner(s) in the cntf cluster: `kubectl create namespace <NAMESPACE (e.g. "gitlab-runners")>`
        * Install your created runner via helm: 
        `helm upgrade --install <RUNNER_NAME> -n <NAMESPACE> --set runnerRegistrationToken=<RUNNER_TOKEN> gitlabUrl=http://www.gitlab.com gitlab/gitlab-runner`
        * Check to see if your runner is working: `kubectl get pods -n <NAMESPACE>` (you should see "1/1" under "READY" and "Running" under "STATUS")
        * Give your runner cluster-wide permissions: `kubectl apply -f gitlab-runner-rbac.yaml`
    * In Gitlab, Under "Assigned project runners" you should now see that your runner has a green circle next to it, signaling a "ready" status
    * **How to re-use this runner for other CNTF repositories:**
        * Hover over "Settings" and select "CI/CD"
        * Under "Other available runners", find the runner you have created and select "Enable for this project"

4. Authenticate [Gitlab with AWS](https://docs.gitlab.com/ee/ci/cloud_deployment/)

5. Run the CI/CD pipeline:
    * On the left side of the screen click the drop-down arrow next to "Build" and select "Pipelines"
    * In the top right hand corner select "Run Pipeline"
    * In the drop-down under "Run for branch name or tag" select the appropriate branch name and click "Run Pipeline"
    * Once again, click the drop-down arrow next to "Build" and select "Pipelines", you should now see the pipeline being executed

## Pipeline Stages
Goal of each stage in the pipeline (refer to ".gitlab-ci.yml" for more details):
* "build" - builds the customer UERANSIM docker image and pushes it to an AWS ECR repository

## Project Structure:
```
└── README.md
|
|
└── .gitlab-ci.yml             contains configurations to run CI/CD pipeline
|
|
└── Dockerfile                 contains the configurations to create the custom ueransim docker image
|   
|
└── amazon-search.js           contains configurations to simulate a search on amazon.com and can be inserted into the dockerfile to modify the ueransim test-suite
|
|
└── devtools-search.js         contains configurations to simulate a search on dev-tools.ai and can be inserted into the dockerfile to modify the ueransim test-suite
|
|
└── youtube-search.js          contains configurations to simulate watching a video on youtube.com and can be inserted into the dockerfile to modify the ueransim test-suite
```

