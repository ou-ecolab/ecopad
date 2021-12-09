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
     If we would directly push them to the main branch of this repository, we would create a bunch of intermediate commits, that would neither work nor have a decent commit message. Although we could later (after our final commit has achieved our goal) combine our little commits to one (by using git rebase and squash) and write a descriptive summary commit message, this practice of 'rewriting history' is actually strongly discouraged for all repositories except our local one. 
   - Solution: 
          
### Fortran Task Tutorial 
This is a quite comprehensive example that will touch nearly all components of ecopad.
In order to reproduce some of the situations you will likely encounter while extending and maintaining the project, the tutorial  **deliberately contains intermediate (failing) code** along with
the corrections and some hints for debugging. 
The padagogical aim is to simulate a realistic workflow.
#### Assumend Technical Goal
Assume that we want to add a new feature to the ecopad website for which a new model has to be run, which is implemented in Fortran.
We will make these changes available in the repo so that every developer can check them on his local machine. 

If you have not cloned the ecopad repository on your local machine do so now  by:

 ```bash
 git clone --recurse-submodules https://github.com/ou-ecolab/ecopad.git
 ```
    
We will first create a test branch of the ecopad repository so that non of our changes will affect the commit history of the main branch unless we want them to.
```
cd ecopad
git checkout -b test 
```
    
Now we create a new repository  under the ou-ecolab organisation (you can do this on the website https://github.com/ou-ecolab (Hit the green New button, name it as you wan and add a README.md so that the repo is not empty.) 
I will name it `FotranExample` here.  
(Since we are in a branch that we can remove later, all traces of our example repository will be gone along with it.
So we do not have to worry about this and can present the same steps as if we were adding a repo for real).  

We integrate the new repo  as a submodule in the ecopad repository:
    
```bash
cd ~/ecopad
git submodule add https://github.com/ou-ecolab/FortranExample.git
cd FortranExample
```
Now create the `Dockerfile`
```
From ubuntu:18.04
RUN apt-get update
RUN apt-get install -y gfortran
```
Run the following command to create the new docker image:

```Bash
docker build -t test:latest .
```

### Run the container(image) and connect to it interactively

```Bash
docker run -it test:latest bash
```
In the container type:

```Bash
gfortran -v
```
Which will give you some verbose information about the gfortran compiler and prooves that it is installed. 

## Extend the container with some Fortran code

The following is a very simple Fortran code with the input from command line. In this example, you will learn how the code receives the arguments from the command line. Copy the code to an empty file (**e.g., test.f90**) in the same directory as the `Dockerfile`.

```Fortran
program main

    character(len=10) first_command_line_argument   !declaration
    character(len=10) second_command_line_argument  !declaration

    call getarg(1, first_command_line_argument)     !get argument
    call getarg(2, second_command_line_argument)    !get argument

    print *, first_command_line_argument            !print 
    print *, second_command_line_argument           !print

end program

Fortran utilizes the function ``getarg()`` to receive the arguments from command line. In the model, the paths to some crucial files are usually passed by command line arguments. Thus, it is important for us tu know how it works.
```
Open the `Dockerfile` and add the following lines 
```
COPY test.f90 /root/
WORKDIR /root
RUN gfortran -o test.o test.f90
```

Get out of the container (Ctr-D) 
Rebuild it:
```Bash
docker build -t test:latest .
```

Once finished, you may run the docker image (this time without interaction) with:

```Bash
docker run test ./test.o test1 test2
```

In the command, ``test`` is the docker image we justed created. ``./test.o`` is the executable file in the ``WORKDIR`` of the docker image.
``test1`` and ``test2`` are two command line arguments.

#### Create a new task in ecopadq

This section will guide you how to create a new task, which is very essential to further develop the platform.
Tasks are defined in a python package which lives in the subrepository: [ecopadq](https://github.com/ou-ecolab/ecopadq). 
Every time we restart cybercommons it will download this package from github and install this package and make the tasks available in on the api website 
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
and check if a new Task appeared in the "Tasks" list!
If so click on the new task.



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

