# Helium.io
A true Kubernetes Native CICD pipeline tool.

![helium](docs/images/helium-small.png)

Helium is light, pure and the safest known ~~substance in the plant~~  **kubernetes native devops toolkit.** It helps building and deploying application into kubernates backed environment.

Helium is written in kotlin and welcomes contributors from JVM community, the entire core codebase is written just within single class file of less than 5kb.
##

### Why Helium?

#### Helium is minimal, super simple and pure without a learning curve.
Build pipelines are not the final product which you or your clients are looking for,  they want the underlying applications as a deliverables and therefore, a pipeline must be simple, small, easy and just enough to serve the need for build and deploy.
Keeping these in mind Helium enables developers to focus on the real application, rather than spending their time and efforts learning, building and maintaining the pipeline.

With helium, there are...
* No Custom CRD's, No Operators to install, so no learning curve, it runs just like any other container.
* No complicated yaml schema, very intuitive, enough to build CICD.
* Single yaml pipline file promote simplicity and maintainability
* Out of the box support for every kubernetes yaml tags, including volumes, secrets etc within the steps.
* Pipeline can be kept along with your source git.
* Out of the box support for git webhooks.
* Promote reusability with simplicity.

##

#### The simplest pipeline.

    pipeline:  
        version: helium/v1alpha1  
        name: testpipeline
        tasks:  
          - name: simple-task  
            steps:  
              - name: simple-step  
                image: alpine:latest  
                command: ["sh", "-c"]  
                args: ["cat /etc/os-release"]



#### tasks:
We can have multiple `tasks` within a `pipeline`, each `task` would run independently, and they would not share any state or data.

A  `task` must have a `name`  and list of `steps`  needs to be performed.

> Tasks are converted into pods in the Kubernetes cluster, failing one task (pod) does not impact other running tasks.

#### steps:
We can have multiple `steps` within each `task`, steps runs sequentially order, and they share the data available in `/workspace` directory.

A  `step` must have a `name` and `image` however, `image` can be excluded when using helium `templats`

When a `step` fails within a `task`, helium stops executing that complete `task`.

> steps are converted into initiContainers within a pod.

##
#### Git Clone example

Helium support cloning multiple git repository 

    tasks:  
      - name: simple-task  
        git: 
          - name: my-app
            clone: https://github.com/bansmani/gradle-springboot-sample-app
        steps:  
          - name: build-source
            image: adoptopenjdk:11-jdk-openj9
            command: [ "sh", "-c" ]
            args: ["cd /workspace/my-app && chmod a+x gradlew && ./gradlew clean build"]


We can pass all types of arguments which is supported by git command. 

E.g. git `--branch` also accepts git `tags`
 
     git:
        - name: my-app 
          clone: http://mygitrepo --branch development


Using a secure git clone. 

        git:
            - name: my-app
              clone: git@github.com:bansmani/gradle-springboot-sample-app.git
              secret: my-ssh-key-secret

##

#### Reusing Pipeline

Re-usability is designed with simplicity within helium, it uses templating for reusability. 
the template file can be kept again with the same codebase, or can be kept on remote host 
or even another git repository. 

#### Template and vars
Template support variable substitution (`vars`), which could be defined at the `step` level or at the `task` level 

> `step` level variable overrides `task` level variables 

    tasks:
        - name: build-task
          template: path-to-template/template.yaml
          git: 
            - name: my-app 
              clone: https://mygiturl
          steps:
            - name: build-source
              template: gradle-build
              vars: 
                - name: sourceroot
                  value: my-app

Meanwhile in the helium template file 

        helium-template:
            gradle-build:
                image: adoptopenjdk:11-jdk-openj9
                command: [ "sh", "-c" ]
                args: ["cd /workspace/${sourceroot} && chmod a+x gradlew && ./gradlew clean build"]

final output will be 

    tasks:
        - name: build-task
          template: path-to-template/template.yaml
          git: 
            - name: my-app 
              clone: https://mygiturl
          steps:
            - name: build-source
              template: gradle-build
              image: adoptopenjdk:11-jdk-openj9
              command: [ "sh", "-c" ]
              args: ["cd /workspace/my-app && chmod a+x gradlew && ./gradlew clean build"]


Task level vars

    tasks:
        - name: build-task
          template: path-to-template/template.yaml
          vars: 
            - name: myvar
              value: Manish
          steps:
            - name: print-fname
              template: echo-step

            - name: print-lname
              template: echo-step
              vars: 
                - name: myvar
                  value: Bansal

Template file

    helium-template:
        gradle-build:
            image: alpine
            command: [ "sh", "-c" ]
            args: ["echo ${myvar}"]


and the output would be 

    [TASK: build-task STEP: print-fname] Manish
    [TASK: build-task STEP: print-lname] Bansal


### Finally a completed pipeline 
* [complete build and deploy pipeline!](testdata/complete-pipeline-example-01.yaml)
  * building gradle based source code
  * building and pushing the image using kaniko 
  * deploying the image using helm  
* [helium template example](testdata/helium-template.yaml)
    
