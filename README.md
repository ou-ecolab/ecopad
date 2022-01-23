useful links: https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History

# ecopad
This Repository is intended as an umbrella for the site specific ecopad installation of Yiqi Luo's ecolab.
It contains the different repositories that constitute the ecopad code base as [submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules). 

# Installation on a single computer:
To clone it use 
```bash
       git clone --recurse-submodules https://github.com/ou-ecolab/ecopad.git
 ```
This will check out the subrepositories (in coordinated compatible versions) as subfolders. 
```bash
       cd ecopad/cybercommonds
```
follow the [installations instrtuctions of the cybercommons package:](https://github.com/ou-ecolab/cybercommons) and write down the username and password that you type in since you will later need it to use the api.

# Development: Extending or changing ecopad 
## Git setup explained
### Special Project Requirements
The ecopad project has some requirement that can **not** be met by a **single** repository: 
   
1. The project combines several codebases that live in different repositories.
   We solve this by the use of [Git-submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules).
  
1. The cybercommons infrastructure uses the repositories for additional tasks apart from version control, 
   namely to transfer the code for the tasks to the workers. 
   We will use temporary test branches to address this to avoid cluttering our version history.


Concerning 1.:
  - Why is this necessary? 
      - Some of the infrastructure (cybercommons) is shared with other projects which are not related to ecopad.
      - The system is usually distributed across several machines with different roles that require 
        only parts of the code.          
  - Challanges:
      - A new feature (like a new graph on the website) can involve changes in several different repositories.
      - These changes have to be coordinated. E.g. the last commit in the website might only work together with the second last
        commit of the TECO repo but not with the latest...
    
  - Solution:
   
    To achieve this coordination is the purpose behind this repository (ecopad) which uses [Git-submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules). In short a commit to ecopad records the state of all the subrepos ( as opposed to the actual code changes which are tracked in the different constituting sbrepos.)
       - Consequences for the workflow:
            - As the examples show this adds another step to our workflow: After we have worked on the different repositories, checked in the code changes, and tested that they work well together, we also commit to ecopad. 
       - Benefits
         - The changes in the subrepos can be combined to a single commit in ecopad which can be documented by a single commit message explaining the single purpose of those commits.   
         - If we clone (with --recurse-subrepos) or pull from ecopad on a development machine 
           we automatically get the subrepos in the versions compatible to each other with one command
         - We can roll back to working combinations of subrepo commits
         - If we deploy a new version of ecopad to our servers (including the workers for the models)  and thus have to checkout different repositories to different machines) we can be
           sure that we get commits of the different subrepos that are compatible with each other and work together. 

Concerning 2.: 
   - Challange:
     
     Since the celery workers receive their tasks by installing them from a remote git-hub repository, we are forced to commit
     and push all the experimental changes to the repository that contains our tasks (in our case [ecopadq](https://github.com/ou-ecolab/ecopadq).) before we even know that they will work.
     If we would directly push them to the main branch of this repository, we would create a bunch of intermediate commits, that would neither work nor have a decent commit message. Although we could later (after our final commit has achieved our goal) combine our little commits to one (by using git rebase and squash) and write a descriptive summary commit message, this practice of 'rewriting history' is actually strongly discouraged for all publicly shared repositories. 
   - Solution: 
          
### Getting Started Tutorial 
This is a quite comprehensive example that will touch nearly all components of ecopad.
The padagogical aim is to simulate a realistic workflow.
If you have not installed  ecopad on your local machine yet 
start with the installation part of this README. This is neccessary to continue.
Assuming that you have succesfully completed the installation a version of ecopad containing
a minimal example is already running.
To see the example in action point your browser to http://localhost/ecopad_portal/ 
Log in with the credentials you provided during the installation process and then hit the botton 
`send to API` and watch what happens.

#### Goals
Assume that we want to add a new feature to the website you are looking at for which a new model has to be run.  The repository contains an example in Fortran as a prototype.
We will go about this in several steps
1. You will create  a functionally identical copy and integrate it into ecopad. This will walk you through the 
directory and repository structure and  familiarize you with the git mechanics.

2. You will change the functionality a tiny bit, and make some intentional changes to different parts of the code. 

We will make these changes available in this repo and its submodules so that every developer can check them on his local machine. 

#### Step 1 

Now we create a new repository  under the ou-ecolab organisation (you can do this on the website https://github.com/ou-ecolab (Hit the green New button, name it as you want and add a README.md so that the repo is not empty.) 
I will call it `{YourName}Example` here.  

The first question is where this new repository should live in the directory structure of ecopad. 
The example on the website can be found in: 

`cybercommons/dc_config/images/local_fortran_example`

Change to this directory 
`cd cybercommons/dc_config/images/local_fortran_example`
and issue the following command:
`git remote -v` 
This tells you the url of your remote companion repository on github.
No go one level up in the directory structure and issue the same command
```bash
cd ..
git remote -v
```
This tells you a different url. Actually `local_fortran_example` is not only a
subfolder, it's a `submodule` of the `cybercommons` repository. In short it
means that the `cybercommons` repository does two things:
1. It ignores the files underneath the `local_fortran_example` folder.
2. It tracks the commit hashes of the `local_fortran_example` repository.  When
   we commit changes to `cybercommons` it will also remember the revision nubmer of
   `local_fortran_example` when we committed those changes.

You made use of
this functionality already when you cloned ecopad with the 
`--recurse submodules` flag, because the `cybercommons` repo is a submodule of the `ecopad` repo.
Check this out by changing  to the top level directory and issuing the same command again. 
```bash 
 cd ../../..
 git remote -v
```
So we have a recursive submodule structure:
`ecopad` is the toplevel repository 
`cybercommons` is a submodule of `ecopad`
`local_fortran_example` is a submodule of `cybercommons`
Your repository will also become a submodule of `cybercommons`
Go back to the cybercommons folder and do it:
```bash
cd cybercommons/dc_config/images
git submodule add https://github.com/ou-ecolab/{YourName}Example
```
To understand submodules read https://git-scm.com/book/en/v2/Git-Tools-Submodules now or later.
The reason why WE use them, is explained in this README under the topic Git setup explained.

Now copy the files from the `local_fortran_example` directory to `{YourName}Example` and take a look at them with an editor.
Then add them to your repo command and commit your changes and push them to github.
```bash
cd {YourName}Example
git add dockerfile entrypoint.sh start_sshd.sh test.f90
git status
git commit -m "your commit message should describe what you just achieved. Don't mention files (git already knows) but describe succinctly what intentions where behind your changes" 
git push
```

Before we try out your new docker container we look at its configuration.
This is stored in `cybercommons/docker-compose.yml`
Open it with an editor, find the section that deals with the `local_fortran_example` copy and paste it and 
in your new section change every occurence of `local_fortran_example` to `{YourName}Example`.
Now we can test your new container. 
(You have to be in the `cybercommons` directory for the following commands to work)
```bash
make shell
docker-compose build {YourName}Example
docker-compose run {YourName}Example '/bin/bash'
```
This will drop you into a shell inside the container
From the inspection of the file `test.f90` you will get an idea that the program takes 3 commandline arguments
for amplitude, phase and the resultfile path. You can check this out by (inside the container shell)
```bash
./test 1 2 "test.csv"
```
which will create a file `test.csv` (or whatever name you chose) which you should inspect with `cat test.csv`.
You can get out of the container shell by pressing `Ctrl-d`.
You are now ready to commit to the `cybercommons` repository.
```bash
git status
git add docker-compose.yml
git add {YourName}Example
git commit -m "Your commit message"
```
Short summary:
You have created and configured  a docker container thet is a prototype for a model.
You can build and run it using the `docker-compose` command but it is not yet integrated in any way with 
the website. It is also not clear yet which role the other containers play in the process.
These will be our next goals.

Task:
1. Open the `docker-compose.yml` again and look at the information concerning your new container.
   Make sure that you understand what these instructions mean. If you are unfamiliar wiht docker go
   throug the minimal tutorial https://docs.docker.com/get-started/ which provides you with the basic 
   tools to understand how to deal with containers. This will be very helpful for debugging.
1. If you haven't done so read this chapter of the git book: 
   https://git-scm.com/book/en/v2/Git-Tools-Submodules  
   This is also a good place of reference to deal with other questions concerning git, much better than
   looking up unrelated problems on stack overflow. 



We will first create a test branch of the ecopad repository so that non of our changes will affect the commit history of the main branch unless we want them to.
To make path descriptions easier I assume from now on that you changed into the directory where you cloned the repository.


#### Create a new task in ecopadq

This section will guide you how to create a new task, which is very essential to further develop the platform.
Tasks are defined in a python package which lives in the subrepository: [ecopadq](https://github.com/ou-ecolab/ecopadq). 
Every time we restart cybercommons it will download this package from github, install it  and make the tasks available in on the api website 
(on your local machine under http://localhost/api/queue/)

We enter the appropriate subdirectory of ecopad (It has already been checked out recursively for us) and also create a temporary test branch of the subrepo. This is important since the fact that cybercommons will download its task from from github forces us to commit and push our changes even before we know that they will work. Additionally we will most likely need several commits until we get everything to work. To keep the resulting mess out of the commit history of the main branch we record it in the testbranch it and will `rewrite` this chapter it before we commit it to the main branch.

(assuming we are in `ecopad`)
```
cd ecopadq
git checkout -b test
```
Now open the file:
```
ecopadq/tasks/tasks.py
```

Every function create in the file tasks.py with the decorator ``app.@task()`` is recognized by cybercommons . 
You may create a new function in the file 
Add the following code.
```Python
@app.task() 
def test(pars):
    task_id = str(test.request.id)
    input_a = pars["test1"]
    input_b = pars["test2"]
    docker_opts = None
    docker_cmd = "./test.o {0} {1}".format(input_a, input_b)
    result = docker_task(docker_name="test", docker_opts=None, docker_command=docker_cmd, id=task_id)
    return input_a + input_b
```
To make these changes available to be used by cybercommons we have to commit and push them. 
```bash
git commit -m 'added an example task to be run fortran conde in a docker container'
git push

To tell cybercommons to use the new version of its tasks queue we have to (temporarily) change its config 
(Assuming that you cloned ecopad into your home directory)
```bash
cd ~/ecopad/cybercommons/dc_config
```
in the file `cybercom_config.env` change the line
```
CELERY_SOURCE=git+https://github.com/ou-ecolab/ecopadq

```
to 
```
CELERY_SOURCE=git+https://github.com/ou-ecolab/ecopadq@test
```
(The `@test` at the end is the branch that celery (a part of cybercommons) will use.)  

Now we restart the cybercommons application.

```bash
cd ~/ecopad/cybercommons
make stop
make run
```

The next step is to see if our changes are reflected on the api website. To check point your browser to `http://localhost/api/queue/`
and see a new url under "Tasks" ` "http://localhost/api/queue/run/ecopadq.tasks.tasks.test/"` 
So far so good. Now click on it!




     - change commit and push our changes to this branch until we achieve our desired feature.
     - rewrite history for this branch by squashing our experimental commits into one and write a commit message for the combined commit
      - locally checkout the main branch and merge the temporary branch into it (this will be only the working commit resulting from our rebase with the nice commit message)
      - push (to the remote branch)
      - remove the temporary branch locally and remotely.
      - change the cybercommons config back to the main branch [ecopadq](https://github.com/ou-ecolab/ecopadq) 
   

   - Benefits:

       We have only one intentianal change in the master branch that works and is described by a commit message that describes purpose and consequences of the change instead of possibly many back and forth changes of which only the last one works...
       As a result we can keep the master branch clean so that whenever we come back to a commit we know that the code works.
       
[here](./development.md)

