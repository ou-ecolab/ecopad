FURTHER DEVELOPMENT
=====================


Further development can be of two types:-
   * Adding a new task to the existing model
    
   * Adding a new model


Note :- It's a very bad practice to modify existing working codes in github master branch and push it back to the master node again.

    1. First create new branch ecopadq.Push codes to the new branch.This is done for version control. 

    2. Create a new task with @task() method decorator

    3. Within celery folder update requirements.txt to include ecopadq test branch instead of master

    4. Restart system
    
    5. Check whether new or updated tasks works fine or not
    
    6. If everything goes well merge branch to master.

Adding a new task to the existing model
--------------------------------------------

Inorder to add a new task ,goto https://github.com/ou-ecolab/ecopadq/tree/master/ecopadq/tasks . Select the new branch and update the tasks.py and restart the system . If it works merge it to the master branch else try again.
 

Adding a new model
---------------------

Inorder to add a new model,we first need to build the model and also a docker file which will contain all the path information of the inputs required for the model and also install all the dependency softwares required to run the model in the proper environment.Then modify the tasks.py in https://github.com/ou-ecolab/ecopadq/tree/master/ecopadq/tasks and include the appropriate tasks and restart the system again. 
