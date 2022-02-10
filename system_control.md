Troubleshooting
=================

After setting up the system,check the docker containers that are running.

      `# docker ps`
      
If all the six containers are not running,then there is a problem with the docker containers and check the log of the corresponding docker container which is failing to load up.To know which docker container has filed to load type:-

      `# docker ps -a`
      
Under STATUS you will find EXITED written on it.The CONTAINER_ID corresponding to that IMAGE will give us an indication of what went wrong.Type the following command. 
      
      `#docker logs CONTAINER_ID`
      
Sometimes checking the celery logs also can give the hint where the problem in our system is which is in ecopad/ecopad/celery/log folder.

      `#tail -f celery.log`
      
Everytime you make some changes in the system,don't forget to restart the system.For that go to the                       \< path_to_application/application_short_name/run \> folder and run the following commands.

      `#./docker_restart`
      
      `#./cybercom_up`
      
The ./docker_restart command kills all the docker containers and removes them from the system.

For Server Adminstration(Not Local Machine)
-------------------------------------------

In the sever under \< /home/ecopad/ecopad/run \> folder ,there is file named upgrade_tasks.If you have done only some coding changes and for that no new library is required,then it is not mandatory to restart the entire system,instead you can just run the upgrade_tasks file which will delete only the ecopadq folder and reinstall that folder again.
