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
  
          
### Getting Started Tutorial 
This is a quite comprehensive example that will touch nearly all the components of ecopad we will get into contact with.
The aim is to simulate a realistic workflow without the threat of messing up a production system.
If you have not installed  ecopad on your local machine yet 
start with the installation part of this README. This is neccessary to continue.
It would also be helpful to understand submodules (read https://git-scm.com/book/en/v2/Git-Tools-Submodules rather now then later) 
It will also help to know the reason why WE use them, which is explained in this 
README under the topic "Git setup explained"

Assuming that you have succesfully completed the installation a version of ecopad containing
a minimal example is already running.
To see the example in action point your browser to http://localhost/ecopad_portal/ 
Log in with the credentials you provided during the installation process and then hit the botton 
`send to API` and watch what happens.

#### Goals
Assume that we want to add a new feature to the website you are looking at for which a new model has to be run. The repository contains an example in Fortran as a prototype.
We will go about this in several steps
1. You will create  a functionally identical copy and integrate it into ecopad. This will walk you through the 
directory and repository structure and familiarize you with the (quite elaborate) git mechanics.

1. You will change the functionality a tiny bit, and make some intentional changes to different parts of the code. 

We will make these changes available in this repo and its submodules so that every developer can check them on his local machine. 

#### Step 1 

Where to start?  We know that we want to make the results of 
a model written in Fortran accessible via a webpage. So we could either start at a model or the webpage.
If we were forced to reverse engineer the whole project we would probably start at the website and work our way through its javascript code. We will not have to go so deep, but we should at least look at it to know where to come back later.
We use the developer tools of the browser to do so.
I will describe here how to see it using  `chrome`.
- to open the developer tools in my version I have to go to `more tools` -> `Developer tools` 
  This will divide the window. There should be a rider `Sources` showing the contents of `demo.js`
  and also the console 
- hit the `send to API` button(on the example website)  again and watch the little status window and the console geting busy with outputs .
- read the outputs! Please! ;-) to get a picture of what is going on. 
  It seems that 
  - the button triggers code that sends `json` data to another url
  - receives a result url in return 
  - pesters this result url (every 5 seconds)  with requests until it returns "SUCCESS"

There is some good news here:
- Our (ecopad specific) code is actually  pretty short.
- The real work of starting jobs and tracking their status is done by something else (the `cybercommons` framework)
- We communicate with this framework via urls. (technically `cybercommons` provides what is called a `REST-api`)
The last point is actually VERY useful. We can hit those urls that our website code is talking to directly with our browser. This peels away one layer of complexity and enables us to remote control cybercommons without OUR frontend website (because it provides one itself).
Although it is not an alternative it is great for testing out all the other parts. (Our model and the task queue about which we will talk later)
So lets do it and 
- hit the first url you saw our website talking to.
  http://localhost/api/queue/run/ecopadq.tasks.tasks.test/
  We could use this page to actually trigger the same computation that our button 'sent to API' starts.
  We could reverse engineer our javascript code to see how many and which arguments it actually transmits and put those manually in the "args" field. But it becomes a bit easier to guess if you see the receiving end of this code.
  Our `test` task is not the only one. 
  To see the other(s) 
- remove the last bit of the url and hit http://localhost/api/queue/
- remove even more and go to http://localhost/api
  The interesting links here are the last one under "User Profile" and the first two under "Queue"
  The "Tasks History" link suggests that cybercommons does some kind of bookkeeping for us, who triggered which   task and when... It needs a database to do this, so we will not be surprised if we find configuration for it later.

Our next question is where those possible tasks are defined. How does (our very much stripped down version of) ecopad know which tasks to offer us in the queue and how to access them?
Without help we would start reading the cybercommons documentation and find that it basically is a web front end for a widely used package called `celery` which does the actual scheduling and monitoring. Reading through the 'celery' documentation we would find that `celery` has a client server structure and a 'client' can set up multiple 'workers' to which it distributes the task code by means of a queue repository. 
Thats our clue. Although in our present configuration 'client' and 'worker' live on the same mashine they still use a (github) repository to store the code for the tasks.
Fortunately you already cloned this repository when you used the `git clone --recurse-submodule` command.
It is on the toplevel of your `ecopad` directory.
 

#### Create a new task in ecopadq

This section will guide you how to create a new task, which is very essential to further development of the platform.
Tasks are defined in a python package which lives in the subrepository: [ecopadq](https://github.com/ou-ecolab/ecopadq). 
Every time we restart cybercommons it will download this package from github, install it  and make the tasks available in on the api website 
(on your local machine under http://localhost/api/queue/)

We enter the appropriate subdirectory of ecopad (It has already been checked out recursively for us) and also create a temporary test branch of the subrepo.
This is important since the fact that the celery worker will download it's tasks from github forces us to commit and push our changes even before we know that they will work.(This is cumbersome but rooted in the client server structure of celery. We could scale to many workers on different machines and for this general situation we need some kind of code distribution) 
Additionally we will most likely need several commits until we get everything to work. To keep the resulting mess out of the commit history of the main branch we will record it in the testbranch it and will `rewrite` this chapter it before we commit it to the main branch.

I suggest that you call your test branch {yourname} to make the name unique and avoid conflicts with somebody elses test branch.
(assuming we are in `ecopad`)
```
cd ecopadq
git remote -v # just to see where we are..
git checkout -b {yourname}
```
Now open the file:
```
ecopadq/tasks/tasks.py
```

Every function in the file `tasks.py` with the decorator ``app.@task()`` is recognized by cybercommons . 
You may create a new function in the file 
- Identify the code of the function for the 'test' example.
- copy, paste and rename the function to the name you want to use (for instance `{yourName}Example`) 

To make these changes available to be used by cybercommons we have to commit and push them. 
```bash
git commit -m 'added an example task to  run fortran code in a docker container'
git push
```

To tell cybercommons to use the new version of its tasks queue we have to (temporarily) change its config 
(Assuming we are in the ecopad folder)
```bash
cd cybercommons/dc_config
```
in the file `cybercom_config.env` change the line
```
CELERY_SOURCE=git+https://github.com/ou-ecolab/ecopadq

```
to 
```
CELERY_SOURCE=git+https://github.com/ou-ecolab/ecopadq@{yourname}
```
(Don't write the {} the only indicate that you should write your name there)
(The `@{yourname}` at the end is the branch that celery will use.)  

Now we restart the cybercommons application.

```bash
cd cybercommons
make stop
make run
```

The next step is to see if our changes are reflected on the api website. To check point your browser to `http://localhost/api/queue/`
and see a new url under "Tasks" ` "http://localhost/api/queue/run/ecopadq.tasks.tasks.{YourName}Example/"` 
So far so good. Now click on it!

     - change commit and push our changes to this branch until we achieve our desired feature.
     - rewrite history for this branch by squashing our experimental commits into one and write a commit message for the combined commit
      - locally checkout the main branch and merge the temporary branch into it (this will be only the working commit resulting from our rebase with the nice commit message)
      - push (to the remote branch)
      - remove the temporary branch locally and remotely.
      - change the cybercommons config back to the main branch [ecopadq](https://github.com/ou-ecolab/ecopadq) 
   
Intermediate Summary:
We have created a new task for ecopad. We can reach it from the REST-api and pretend (for the moment)
that it does something different than the `test` example.

Note: 
- that the fact that we use 'ssh' for the 'test' and your new task (to perform it in a docker container that is   connected via an internal network)  is by no means necessary for all tasks. 
  Look at the `add` function (also check it out via the api ) which is completely written in `python`. 
  We use containers only because this is the use case for the models in the present ecopad. 
- that our new task is available through the REST-api of cybercommons but not yet on OUR website.
  We will have to connect it.

#### create a new docker container 

Now we create a new repository  under the ou-ecolab organisation (you can do this on the website https://github.com/ou-ecolab (Hit the green New button, name it as you want and add a README.md so that the repo is not empty.) 
I will call it `{YourName}Example` here.  

This will be eventually the home of the changed example.

The first question is where this new repository should live in the directory structure of ecopad. 
The example on the website can be found in: 

`cybercommons/dc_config/images/local_fortran_example`

Change to this directory 
`cd cybercommons/dc_config/images/local_fortran_example`
and issue the following command:
`git remote -v` 
This tells you the url of your remote companion repository on github.
Now go one level up in the directory structure and issue the same command
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
You can build and run it using the `docker-compose` command but it is not yet integrated with 
the website. 


Tasks:
1. Go back to the last subsection and change the code in the task queue to use YOUR new fortran container.
1. Open the `docker-compose.yml` again and look at the information concerning your new container.
   Make sure that you understand what these instructions mean. If you are unfamiliar wiht docker go
   throug the minimal tutorial https://docs.docker.com/get-started/ which provides you with the basic 
   tools to understand how to deal with containers. This will be very helpful for debugging.
1. If you haven't done so read this chapter of the git book: 
   https://git-scm.com/book/en/v2/Git-Tools-Submodules  
   This is also a good place of reference to deal with other questions concerning git, much better than
   looking up unrelated problems on stack overflow. 



#### Add your example into the website

The HTML of our webpage is found in `cybercommons/web/ecopad_portal/` (assuming you are in `ecopad`) and is
also  (you guessed it) a submodule. 
Convince yourself!
```bash
cd cybercommons/web/ecopad_portal
git remote -v
```
The procedure probably starts to look familiar.

- create a test branch
  ```bash
  git checkout -b {yourname}
  ```
- copy and paste the relevant HTML in index.html (make a new rider) and functions in demo.js to connect to 
  the new task in the queue.
- check with the browser until you are happy.(often reloading the page with Ctrl-F5 to erase the cache)
  This should in our special case of copying not involve any changes to the ecopadq, but for a real example 
  probably would. You would likely use the REST-api directly to make sure that you send the information your 
  task (in ecopadq) expects.
  Obviously ecopad has to run 
- refactor (remove all duplicated code...) while testing.
- check in your changes into your testbranch

#### Integrate all the changes into a commit of ecopad with the correct version of the subrepos 

We mentioned the submodule structure.
			ecopad

	ecopadq							cybercommons
			
					ecopad_portal	local_fortran_exiample 	{YourName}Example

The repos with subrepos are ecopad and cybercommons. 
The ecopad repository tracks (in addition to it's files, mainly this README) the commit hashes of **main** branch of cybercommons and the **master** branch of ecopadq. The point is that it does not track our test branches. 
In the same way the cybercommons repo tracks (in addition to it's own files)  the commit hashes of the **master** branches of ecopad_portal the **main** branch of local_fortran_example 
and whatever is the current gitbuh default branch for {YourName}Example.
The purpose of tracking the independent repositories as submodules is to allow them to have their own commits but to be able to pick out a particular combination of versions that we know works (have tested to work) with the other parts.
When we want to make such a commit to 'cybercommons' and finally 'ecopad' we first have to merge the changes in the submodule branches into their respective master/main branches.

We start from the bottom up. The only subrepo of cybercommons that has any branches is probably `ecopad_portal`
Assuming that you have checked in your changes you can switch to the master branch, pull the latest changes (in case somebody changed something in the meantime) and merge your testbranch.
(assuming you are in ecopad)
```bash
cd cybercommons/web/ecopad_portal/
git checkout master
git pull
git merge {yourname}
```
Then we go up and check in the changes to cybercommons.
Always use `git status` to make sure you don't commit unintentional changes.
E.g. make sure that you reverted the changes in the config file  `cybercommons/dc_config/cybercom_config.env`
back to the original ecopadq.
```bash
cd ../..
git status 
git add web/ecopad_portal
git add dc_config/images/{YourName}Example
git commit -m 'your commit message'
git push
```

Now we go up again and check in the version of `cybercommons` and `ecopadq` that we know work well together
and described the accomplishment of this commit without replecation of the details we expressed already in the commit messages of the submodules. Eventually we want to record this changes in the main branch but for now we check them into a test branch (If we start with this practice we have a better chance to keep the main branch clean)

```bash
cd ..
git status 
git checkout -b {yourname}
git add cybercommons ecopadq
git commit -m "Integrated a new example model into the website."
git push
```


In the future we will hopefully have some automated tests on github that are triggered by a push (to whatever branch). If those tests succeed we would merge the testbranch into the master.
To check the test branch manually you could stop your running ecopad, rename the containing folder it and check out the version you just commited. This is what a CI tool would do. In case you forget to commit some important file or change it will brake (in contrast to your local version wich will run on YOUR machine, since you have
the files you forgot to commit...)

```bash
cd cybercommons
make stop 
cd ..
mv ecopad ecopad.bak
git clone --recurse-submodules https://github.com/ou-ecolab/ecopad.git
git checkout {yourname}
cd ecopad/cybercommons
make run
```
If this is successfull you can merge the testbranch into the master.

```bash
git checkout master
git pull #in case somebody else worked on it
git merge {yourname}
```



Note that we do not have to do this for every commit of one of the subrepos, only if we want to make those changes available automatically via a single pull command in 
