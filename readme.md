# Building a Secure, Automated Supply Chain - Mid Atlantic Summit 2019

In this lab you will integrate Docker Enterprise in to your development pipeline. You will push an image to the Docker Trusted Registry (DTR). DTR will scan your image for vulnerabilities so they can be fixed before your application is deployed. This helps you build more secure apps!

> **Difficulty**: Beginner
>
> **Time**: Approximately 30 minutes
>
> **Table of Contents**:
>
> * [Who am i](#who-am-i)
> * [Document conventions](#document-conventions)
> * [Abbreviations](#abbreviations)
> * [Prerequisites](#prerequisites)
> * [Understanding the Play With Docker Interface](#understanding-the-play-with-docker-interface)
>   * [1. Console Access](#1-console-access)
>   * [2. Access to your Universal Control Plane (UCP) and Docker Trusted Registry (DTR) servers](#2-access-to-your-universal-control-plane-ucp-and-docker-trusted-registry-dtr-servers)
>   * [3. Session Information](#3-session-information)
> * [Introduction](#introduction)
> * [Task 1 Accessing PWD](#task-1---accessing-pwd)
>   * [Task 1.1 Set Up Environment Variables](#task-11---set-up-environment-variables)
> * [Task 2 Enable Docker Image Scanning](#task-2---enable-docker-image-scanning)
> * [Task 3 Create Jenkins User and Organization](#task-3---create-jenkins-user-and-organization)
>   * [Task 3.1 Create Jenkins Organization](#task-31---create-jenkins-organization)
>   * [Task 3.2 Create Jenkins User](#task-32---create-jenkins-user)
> * [Task 4: Create DTR Repositories](#task-4---create-dtr-repositories)
>   * [Task 4.1 Create Promotion Policy - Private to Public](#task-41---create-promotion-policy---private-to-public)
> * [Task 5 Pull / Tag / Push Docker Image](#task-5---pull--tag--push-docker-image)
> * [Task 6 Review Scan Results](#task-6---review-scan-results)
>   * [Task 6.1 Hide Vulnerabilities](#task-61---hide-vulnerabilities)
> * [Task 7 Extend with Image Mirroring](#task-7---extend-with-image-mirroring)
> * [Task 8 Docker Content Trust / Image Signing](#task-8---docker-content-trust--image-signing)
> * [Task 9 Automate with Jenkins](#task-9---automate-with-jenkins)
>   * [Task 9.1 Deploy Jenkins](#task-91---deploy-jenkins)
>   * [Task 9.2 Plumb Jenkins](#task-92---plumb-jenkins)
>   * [Task 9.3 Webhooks](#task-93---webhooks)
> * [Conclusion](#conclusion)

## Who am i

* Github : [https://github.com/clemenko](https://github.com/clemenko)
* Twitter : [@clemenko](https://twitter.com/clemenko)
* Email : [clemenko@docker.com](mailto:clemenko@docker.com)

## Document conventions

When you encounter a phrase in between `<` and `>`  you are meant to substitute in a different value.
We are going to leverage the power of [Play With Docker](http://play-with-docker.com).

## Abbreviations

The following abbreviations are used in this document:

* UCP = Universal Control Plane
* DTR = Docker Trusted Registry
* DCT = Docker Content Trust
* CVE = Common Vulnerabilities and Exposures
* PWD = Play With Docker

## Prerequisites

This lab requires an instance of Docker Enterprise. Docker Enterprise includes Docker Universal Control Plane and Docker Trusted Registry. This lab provides Docker Enterprise.

## Understanding the Play With Docker Interface

![pwd screen](img/pwd_screen.jpg)

This workshop is only available to people in a pre-arranged workshop. That may happen through a [Docker Meetup](https://events.docker.com/chapters/), a conference workshop that is being led by someone who has made these arrangements, or special arrangements between Docker and your company. The workshop leader will provide you with the URL to a workshop environment that includes [Docker Enterprise](https://www.docker.com/enterprise-edition). The environment will be based on [Play with Docker](https://labs.play-with-docker.com/).

If none of these apply to you, contact your local [Docker Meetup Chapter](https://events.docker.com/chapters/) and ask if there are any scheduled workshops. In the meantime, you may be interested in the labs available through the [Play with Docker Classroom](https://training.play-with-docker.com/alacart/).

There are three main components to the Play With Docker (PWD) interface.

### 1. Console Access

Play with Docker provides access to the 3 Docker Enterprise hosts in your Cluster. These machines are:

* A Linux-based Docker Enterprise 2.2 (UCP 3.1.6 & DTR 2.6.5 & 18.09.2)  Manager node
* Three Linux-based Docker Enterprise 2.2 (18.09.3) Worker nodes

By clicking a name on the left, the console window will be connected to that node.

### 2. Access to your Universal Control Plane (UCP) and Docker Trusted Registry (DTR) servers

Additionally, the PWD screen provides you with a one-click access to the Universal Control Plane (UCP)
web-based management interface as well as the Docker Trusted Registry (DTR) web-based management interface. Clicking on either the `UCP` or `DTR` button will bring up the respective server web interface in a new tab.

### 3. Session Information

Throughout the lab you will be asked to provide either hostnames or login credentials that are unique to your environment. These are displayed for you at the bottom of the screen.

**Note:**  There are a limited number of lab connections available for the day. You can use the same session all day by simply keeping your browser connection to the PWD environment open between sessions. This will help us get as many people connected as possible, and prevent you needing to get new credentials and hostnames in every lab. However, if you do lose your connection between sessions simply go to the PWD URL again and you will be given a new session.

## Introduction

This workshop is designed to demonstrate the power of Docker Secrets, Image Promotion, Scanning Engine, and Content Trust. We will walk through creating a few secrets. Deploying a stack that uses the secret. Then we will create a Docker Trusted Registry repository where we can create a promotion policy. The promotion policy leverages the output from Image Scanning result. This is the foundation of creating a Secure Supply Chain. You can read more about  secure supply chains for our [Secure Supply Chain reference architecture](https://success.docker.com/article/secure-supply-chain).

## Task 1 - Accessing PWD

1. Navigate in your web browser to the URL the workshop organizer provided to you. **Chrome is advised!**

2. Fill out the form, and click `submit`. You will then be redirected to the PWD environment. It may take a minute or so to provision out your PWD environment.

### Task 1.1 - Set Up Environment Variables

We are going to use `worker3` for **ALL** our command line work. Click on `worker3` to activate the shell.

![pwd screen](img/pwd_screen.jpg)

Now we need to setup a few variables. We need to create `DTR_URL` and `DTR_USERNAME`. But the easiest way is to clone the Workshop Repo and run script.

```bash
git clone https://github.com/clemenko/fedsummit_19.git
```

Once cloned, now we can run the `var_setup.sh` script.

```bash
source fedsummit_19/scripts/var_setup.sh
```

Now your PWD environment variables are setup. We will use the variables for some scripting.

## Task 2 - Enable Docker Image Scanning

Before we create the repositories, let's start with enabling the Docker Image Scanning engine. Good thing there is a script for that.

```bash
./fedsummit_19/scripts/enable_scanning.sh
```

## Task 3 - Create Jenkins User and Organization

In order to setup our automation we need to create an organization and a user account for Jenkins. We are going to create a user named `jenkins` in the organization `ci`.

### Task 3.1 - Create Jenkins Organization

1. From the `PWD` main page click on `DTR`.
  ![orgs](img/orgs_1.jpg)

2. Once in `DTR` navigate to `Organizations` on the left.
3. Now click `New organization`.
4. Type in `ci` and click `Save`.
  ![new_org](img/new_org.jpg)

Now we should see the organization named `ci`.

![new_org2](img/orgs_2.jpg)

### Task 3.2 - Create Jenkins User

While remaining in DTR we can create the user from here.

1. Click on the organization `ci`.
2. Click `Add user`.
3. Make sure you click the radio button `New`. Add a new user name `jenkins`. Set a simple password that you can remember. Maybe `admin1234`?

    ![newuser](img/new_user.jpg)

4. Now change the permissions for the `jenkins` account to `Org Owner`.

    ![admin](img/org_admin.jpg)

5. Now we need to set the password for the Jenkins user on `worker3`. Replace your password for the `<PASSWORD>` below.

    ```bash
    #export DTR_TOKEN=<PASSWORD>
    export DTR_TOKEN=admin1234
    ```

## Task 4 - Create DTR Repositories

We now need to access Docker Trusted Registry to setup two repositories.

We have an easy way with a script or the hard way by using the GUI.

Either way we need to create two repositories, `summit19_build` and `summit19`. `summit19_build` will be used for the private version of the image. `summit19` will be the public version once an CVE scan is complete.

**Easy Way:**

Since we used `git clone` to copy the repository to `worker3` for this workshop, there is a script from that will create the DTR repositories.

```bash
./fedsummit_19/scripts/create_repos.sh
```

This script will also create the Promotion Policy we will discuss further.

**Hard Way:**

1. Navigate to `Repositories` on the left menu and click `New repository`.
2. Create that looks like `ci`/`summit19_build`. Make sure you click `Private`. Do not click `Create` yet!
3. Click `Show advanced settings` and then click `On Push` under `SCAN ON PUSH`.  This will ensure that the CVE scan will start right after every push to this repository.  And turn on `IMMUTABILITY`. Then click `Create`.
  ![new_repo](img/new_repo.jpg)

4. Repeat this for creating the `ci`/`summit19` `Public` repository with `SCAN ON PUSH` set to `On Push` and `IMMUTABILITY` turned `Off`.

5. We should have two repositories now.
  ![new_repo](img/repo_list.jpg)

### Task 4.1 - Review Promotion Policy - Private to Public

With the two repositories setup we can now define the promotion policy. No we can review the policy for the `ci`/`summit19` repository.

1. Navigate to the `ci`/`summit19_build` repository. Click `Promotions` and click "eye" on the right of the policy that was created from the scripts in a earlier task.
  ![create](img/review_policy.jpg)

    Notice there are a lot of options for Criteria for triggering promotions. This allows for a lot of customization. When we push any image to `ci`/`summit19_build` it will get scanned. Based on that scan report we could see the image moved to `ci`/`summit19`. Lets push a few images to see if it worked.

## Task 5 - Pull / Tag / Push Docker Image

Lets pull, tag, and push a few images to YOUR DTR.

In order to push and pull images to DTR we will need to take advantage of PWD's Console Access.

1. Navigate back to the PWD tab in your browser.
2. Click on `worker3`.
3. In the console we should already have a variable called `DTR_URL`. Lets check.

    ```bash
    ./fedsummit_19/scripts/pull_tag_push.sh
    ```

## Task 6 - Review Scan Results

Lets take a good look at the scan results from the images. Please keep in mind this will take a few minutes to complete.

1. Navigate to DTR --> `Repositories` --> `ci/summit19_build` --> `Tags`.

    Don't worry if you see images in a `Scanning...` or `Pending` state. Please click to another tab and click back.

    ![list](img/image_list.jpg)

2. Take a look at the details to see exactly what piece of the image is vulnerable.

     Click `View details` for an image that has vulnerabilities. How about `flask`? There are two views for the scanning results, **Layers** and **Components**. The **Layers** view shows which layer of the image had the vulnerable binary. This is extremely useful when diagnosing where the vulnerability is in the Dockerfile.

     ![list](img/image_view.jpg)

    Now lets look at the Components view. The vulnerable binary is displayed, along with all the other contents of the layer, when you click the layer itself. In this example there are a few potentially vulnerable binaries:

    ![list](img/image_comp.jpg)

    Now we have a chance to review each vulnerability by clicking the CVE itself, example `CVE-2019-3822`. This will direct you to Mitre's site for CVEs.

    ![list](img/mitre.jpg)

    Now that we know what is in the image. We should probably act upon it.

### Task 6.1 - Hide Vulnerabilities

If we find that they CVE is a false positive. Meaning that it might be disputed, or from OS that you are not using. If this is the case we can simply `Hide` the vulnerability. This will not remove the fact that the CVE was found.

Click `Show layers affected` and then you can click `Hide` for the one critical CVE.

![hide](img/cve_hide.jpg)

If we click back to `Tags` we can now see that the image does not have a critical vulnerability.

![critical](img/cve_no_critical.jpg)

## Task 7 - Extend with Image Mirroring

Docker Trusted Registry allows you to create mirroring policies for a repository. When an image gets pushed to a repository and meets a certain criteria, DTR automatically pushes it to repository in another DTR deployment or Docker Hub.

This not only allows you to mirror images but also allows you to create image promotion pipelines that span multiple DTR deployments and data centers. Let's set one up. How about we mirror an image to [hub.docker.com](https://hub.docker.com)?

1. Go to [hub.docker.com](https://hub.docker.com) and create an login and repository.

2. Navigate to `Repositories` --> `ci`/`summit19` --> `MIRRORS` --> `New mirror`.
   Change the `REGISTRY TYPE` to `Docker Hub` and fill out the relevant information like:

   ![mirror1](img/mirror.jpg)

3. Click `Connect` and scroll down.
4. Next create a `tag name` Trigger that is equal to `promoted`
5. Leave the `%n` tag renaming the same.
6. Click `Save & Apply`.
    ![mirror2](img/mirror2.jpg)

Since we already had an image that had the tag `promoted` we should see that the image was pushed to [hub.docker.com](https://hub.docker.com). In fact we can click on the [hub](https://hub.docker.com) repository name to see if the image push was successful.

![more](img/mirror3.jpg)

## Task 8 - Docker Content Trust / Image Signing

Docker Content Trust/Notary provides a cryptographic signature for each image. The signature provides security so that the image requested is the image you get. Read [Notary's Architecture](https://docs.docker.com/notary/service_architecture/) to learn more about how Notary is secure. Since Docker Enterprise is "Secure by Default," Docker Trusted Registry comes with the Notary server out of the box.

We can create policy enforcement within Universal Control Plane (UCP) such that **ONLY** signed images from the `ci` team will be allowed to run. Since this workshop is about DTR and Secure Supply Chain we will skip that step.

Let's sign your first Docker image?

1. Right now you should have a promoted image `$DTR_URL/ci/summit19:flask`. We need to tag it with a new `signed` tag.

   ```bash
   docker pull $DTR_URL/ci/summit19:flask
   docker tag $DTR_URL/ci/summit19:flask $DTR_URL/ci/summit19:signed
   ```

2. Now lets use the Trust command... It will ask you for a BUNCH of passwords. Do yourself a favor in this workshop and use `admin1234`. We can even use a variable for this.

    ```bash
    export DOCKER_CONTENT_TRUST_ROOT_PASSPHRASE="admin1234"
    export DOCKER_CONTENT_TRUST_REPOSITORY_PASSPHRASE="admin1234"
    docker trust sign $DTR_URL/ci/summit19:signed
    ```

    Here is an example output:

    ```bash
    [worker3] (local) root@10.20.0.11 ~
    $     docker trust sign $DTR_URL/ci/summit19:signed
    Created signer: jenkins
    Finished initializing signed repository for ip172-18-0-7-bjmlj2h6u0dg00a2j7n0.direct.beta-hybrid.play-with-docker.com/ci/summit19:signed
    Signing and pushing trust data for local image ip172-18-0-7-bjmlj2h6u0dg00a2j7n0.direct.beta-hybrid.play-with-docker.com/ci/summit19:signed, may overwrite remote trust data
    The push refers to repository [ip172-18-0-7-bjmlj2h6u0dg00a2j7n0.direct.beta-hybrid.play-with-docker.com/ci/summit19]
    79d8c9b0c7eb: Layer already exists
    73b1d8895770: Layer already exists
    758b3ad582e9: Layer already exists
    f1b5933fe4b5: Layer already exists
    signed: digest: sha256:dfd465db73b8fd3d4d64175f11dc38725913b49a2e59d82a70c75a2892f621a2 size: 1156
    Signing and pushing trust metadata
    Successfully signed ip172-18-0-7-bjmlj2h6u0dg00a2j7n0.direct.beta-hybrid.play-with-docker.com/ci/summit19:signed
    ```

    Again please use the same password. It will simplify this part of the workshop.

3. And we can confirm the signature has been applied by inspecting the image:

    ```bash
    docker trust inspect $DTR_URL/ci/summit19:signed
    ```

    Here is the example output:

      ```bash
    [worker3] (local) root@10.20.0.11 ~
    $ docker trust inspect $DTR_URL/ci/summit19:signed
    [
        {
            "Name": "ip172-18-0-7-bjmlj2h6u0dg00a2j7n0.direct.beta-hybrid.play-with-docker.com/ci/summit19:signed",
            "SignedTags": [
                {
                    "SignedTag": "signed",
                    "Digest": "dfd465db73b8fd3d4d64175f11dc38725913b49a2e59d82a70c75a2892f621a2",
                    "Signers": [
                        "jenkins"
                    ]
                }
            ],
            "Signers": [
                {
                    "Name": "jenkins",
                    "Keys": [
                        {
                            "ID": "2e78226bb7144f8af1d416984e20fa688de714570a1841395dd5b55950cb65d0"
                        }
                    ]
                }
            ],
            "AdministrativeKeys": [
                {
                    "Name": "Root",
                    "Keys": [
                        {
                            "ID": "1330c69fa43a6ab40e286876efdb31c0de2aa3fb6b84b9f74fcc3a1b77bb55ad"
                        }
                    ]
                },
                {
                    "Name": "Repository",
                    "Keys": [
                        {
                            "ID": "c2265b8fde8433dd01986815a879af71dcbb9607197c65dbd6fd96b3e02a74a8"
                        }
                    ]
                }
            ]
        }
    ]
      ```

4. Back in DTR, Navigate to `Repositories` --> `ci`/`summit19` --> `Tags` and you will now see the new `signed` tag with the text `Signed` under the `Signed` column:

    ![promoted](img/promoted_signed.jpg)

5. If you were to enable Docker Content Trust in UCP then you would need to upload the public certificate used to sign the image. As we did not perform the `docker trust signer add` command before step 2 above then a public certificate is automatically generated but is not associated to a user in UCP. This means when UCP tries to verify the signature on a signed image to a user it will fail and therefor not meet UCP's Content Trust policy.

    To resolve this issue you can upload the base64 encoded public certificate in `~/.docker/trust/tuf/$DTR_URL/ci/summit19/metadata/targets.json` - the certificate is located in the structure `.signed.delegations.keys` with the key value of `public`.

    For example, use the command `cat ~/.docker/trust/tuf/$DTR_URL/ci/summit19/metadata/targets.json | jq '.signed.delegations.keys' | grep public` to extract the certificate.

## Task 9 - Automate with Jenkins

In order to automate we need to deploy Jenkins. If you want I can point you to a few Docker Compose yamls. OR we have the easy way. The easy, aka script, deploys Jenkins quickly.

### Task 9.1 - Deploy Jenkins

1. Take a look at the script. Also notice the script will check variables, and then runs `docker run`.

    ```bash
    cat ./fedsummit_19/scripts/jenkins.sh
    ```

2. Then install Jenkins.

    ```bash
    ./fedsummit_19/scripts/jenkins.sh
    ```

3. Pay attention to the url AND Jenkins password. It will look like :

    ```bash
    [worker3] (local) root@10.20.0.11 ~
    $ ./fedsummit_19/scripts/jenkins.sh
    =========================================================================================================

      Jenkins URL : http://ip172-18-0-8-bjmlj2h6u0dg00a2j7n0.direct.beta-hybrid.play-with-docker.com:8080

    =========================================================================================================
      Waiting for Jenkins to start.....................
    =========================================================================================================

      Jenkins Setup Password = 9a64091d98d5460b8675de99ebd0b8f2

    =========================================================================================================
    ```

4. Now navigate to `http://$DOCS_URL:8080` by clicking on the url in the terminal. Let's start the setup of Jenkins and enter the password. It may take a minute or two for the `Unlock Jenkins` page to load. Be patient.
  ![token](img/jenkins_token.jpg)

5. One the password is entered click grey "X" in the upper right hand corner. This will cancel out of the installer.
  ![plugins](img/jenkins_plugins.jpg)

6. And we are done installing Jenkins. Click `Start using Jenkins`
  ![finish](img/jenkins_finish.jpg)

### Task 9.2 - Plumb Jenkins

Now that we have Jenkins setup and running we need to add an "item".

1. Click on `New item` in the upper left.
  ![newitem](img/jenkins_newitem.jpg)

2. Enter a name like `ci_summit19`, click `Freestyle project` and then click `OK`.
  ![time](img/jenkins_item.jpg)

3. Let's scroll down to the `Build` section. We will come back to the `Build Triggers` section in a bit. Now click `Add build step` --> `Execute shell`.
  ![build](img/jenkins_build.jpg)

4. You will now see a text box. Past the following build script into the text box.

    **Please replace the <DTR_URL> with your URL! `echo $DTR_URL` <-- `worker3`**

    ```bash
    DTR_USERNAME=admin
    DTR_URL=<DTR_URL>

    docker login -u admin -p admin1234 $DTR_URL

    docker pull clemenko/summit19:alpine

    docker tag clemenko/summit19:alpine $DTR_URL/ci/summit19_build:jenkins_$BUILD_NUMBER

    docker push $DTR_URL/ci/summit19_build:jenkins_$BUILD_NUMBER

    docker rmi clemenko/summit19:alpine $DTR_URL/ci/summit19_build:jenkins_$BUILD_NUMBER
    ```

    It will look very similar to:
    ![build2](img/jenkins_build2.jpg)

      Now scroll down and click `Save`.

5. Now let's run the build. Click `Build now`.
  ![now](img/jenkins_buildnow.jpg)

6. You can watch the output of the `Build` by clicking on the task number in the `Build History` and then selecting `Build Output`
  ![history](img/jenkins_bhistory.jpg)

7. The console output will show you all the details from the script execution.
  ![output](img/jenkins_output.jpg)

8. Review the `ci`/`summit19` repository in DTR. You should now see a bunch of tags that have been promoted.
  ![supply](img/automated_supply.jpg)

### Task 9.3 - Webhooks

Now that we have Jenkins setup we can extend with webhooks. In Jenkins speak a webhook is simply a build trigger. Let's configure one.

1. Navigate to Jenkins and click on the project/item called `ci_summit19` and click on `Configure` on the left hand side.

2. Then scroll down to `Build Triggers`. Check the checkbox for `Trigger builds remotely` and enter a Token of `summit19_rocks`.  Scroll down and click `Save`.
  ![trigger](img/jenkins_triggers.jpg)

3. Now in your browser goto YOUR `http://$DOCS_URL:8080/job/ci_summit19/build?token=summit19_rocks`

    It should look like: `http://ip172-18-0-8-bjmlj2h6u0dg00a2j7n0.direct.beta-hybrid.play-with-docker.com:8080/job/ci_summit19/build?token=summit19_rocks`

4. Check DTR to verify the images were pushed. Then log into `https://hub.docker.com` to see if your images were mirrored.

    ![hub_mirror](img/hub_mirror.jpg)

## Conclusion

In this workshop we were able use the tools that are included with Docker Trusted Registry to build a basic Automated Secure Supply Chain. Hopefully with this foundation you can build your own organizations Secure Supply Chain!
