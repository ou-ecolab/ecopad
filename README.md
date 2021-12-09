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
      - Use a shortlive temporay branch to test and combine our changes.
        ```bash
        cd ecopad/ecopadq/ecopadq/tasks
        ```
     
        We give an example for a chnage in our task list.
        The steps are the following (and actually much quicker than this description suggests):
        - create a temporary test-branch of [ecopadq](https://github.com/ou-ecolab/ecopadq) (with a name like tmp_test_diagram clearly indicating it's short live span and discouraging other people from checking it out) 
         - temporarily change the cybercommons configuration to use this branch.
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

