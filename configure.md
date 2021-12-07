Configuration
==============

It is not a good practice to run commands as root.So we create a Docker group and our username to that group.


Docker Configure
-----------------

1. Create a docker group

      `# sudo groupadd docker`

2. Add your username to the docker group.

      ` # sudo usermod -aG docker <user_name> ` 
      
3. Run this command so that you don't need to restart your shell for the change to occur.

      `# sg docker -c "bash"`



#### Allow ssh on port 22

The following commands allows port 22 to accept ssh connections in your local system which is required to connect with the dockers.

Note:- apt-get is an ubuntu command,if you are on a fedora system use yum.

       `# sudo apt-get update`
       
       `# sudo apt-get install openssh-server`
       
       `# sudo ufw allow 22`


#### Creating authentication keys

13. Now we need to create ssh keys which will create 3 files id_rsa,id_rsa.pub and known_hosts.From the id_rsa.pub file,we will again create another file known as authorizeed_keys.These keys are required to connect to the cybercommons platform.To create the keys type the following commands.

    `# ssh-keygen`
    
       Press Enter for the first command ,don't type any paraphase for second and third commands.After that type the following command
    
    `# cat id_rsa.pub >> authorized_keys`

       To run the system we need keys to communicate with the system.This is the reason why we create ssh keys.To learn more about ssh keys [click here](https://help.github.com/articles/generating-an-ssh-key/)

#### Configuring the cybercom_up file and running the system

14. Now go to  \< path_to_application_short_name/run \> and open the file cybercom_up and in the celery part mount env,.ssh and add the environmental variable host_data_dir in cybercom_up.
    
       `# vi cybercom_up`
    
       Change [[host_ip="to_your_local_machines_ip-address]]

       This is how the docker command of celery should exactly look like.

       
       `# docker run -d --name application_short_name_celery --link application_short_name_rabbitmq --link application_short_name_mongo -v /home/<username>/.ssh:/root/.ssh -v /home/<username>/<path_to_application_short_name>/celery/code:/code:z -v /home/<username>/<path_to_application_short_name>/celery/log:/log:z -v /home/<username>/<path_to_application_short_name>/data:/data:z -e "host_data_dir=/home/<username>/<path_to_application_short_name>/data" -e "docker_worker=$host_ip" -e "docker_username=$docker_username"  -e "C_FORCE_ROOT=true" -e "CELERY_CONCURRENCY=8" cybercom/celery`
       
       
       



15. Now the configuration is almost complete.Now when we run the cybercom_up file.Everthing should work perfectly fine.Type the           following set of commands so that the system restart the docker containers.
   
    `# ./cybercom_up`

    `# ./docker_restart`

    `# ./cybercom_up`
 
 After this open your browser and type 0.0.0.0/ecopad_portal.You should see the system running.
 
 If you are still facing problems [click here](https://github.com/ou-ecolab/ecopad_documentation/tree/master/system_control) for  assistance.
   
