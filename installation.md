# 
#### Building the teco_spruce and teco_spruce_viz docker images

1. Git clone teco_spruce and teco_spruce_viz from github inside path-to-application/\<application_short_name\>
   

     `$ git clone https://github.com/ou-ecolab/teco_spruce`

     `$ git clone https://github.com/ou-ecolab/teco_spruce_viz`

       teco_spruce folder contains the actual fortran code which runs the teco_spruce model.This folder also contains a DockerFile using which we can build an image of teco_spruce.

       teco_spruce_viz contains the R code for the visualization of the generated graphs.This folder also contains a DockerFile using which we can build an image of teco_spruce_viz.
3. Goto \<application_short_name/teco_spruce\> and build the  teco_spruce image.

     `# docker build -t teco_spruce .`

4. Goto \<application_short_name/teco_spruce_viz\> folder and inside it build ecopad_r image.

     `# docker build -t ecopad_r .`

#### Creating the spruce_data folder

2. Goto \<application_short_name/data/local\> folder and create a folder  \<spruce_data\> and go inside that folder and copy some important which will be required during the course of the installation. 
   
     `# mkdir spruce_data`


      This folder will contain all the input files required to run  the teco_spruce model.
      
      
     `# cd spruce_data`

     `#  wget -r -np -nd --reject "index.html*" http://ecolab.cybercommons.org/misc/spruce_data/ wget -r -np -nd --reject "index.html*" http://ecolab.cybercommons.org/misc/spruce_data/ `
     
3. Create a folder inside \<spruce_data\> folder and name it \<Weathergenerate\> and move all the csv file from \<spruce_data\> to it.
     
     `# mkdir Weathergenerate`

     `# mv EMforcing* Weathergenerate/ `
     
     The Weathergenerate file is needed to build  the teco_spruce image.



#### Downloading the frontend contents

8. Git clone ecopad_portal from github inside \<application_short_name/data/static\> folder.
 
       `# git clone https://github.com/ou-ecolab/ecopad_portal`

       ecopad_portal contains the index.html along whith the api.js and template files which is the Grapical User Interface(GUI) of the system.

#### Some additional configurations 


10. Add pandas to the requirment.txt inside \<application_short_name/celery/code\>.

       Requirement.txt contains all the dependent libraries required to run run our system.Pandas is a python library which is used in the system to pull data from the spruce website.To learn more about pandas [click here](http://pandas.pydata.org/)

11. Inside \<application_short_name/celery/code\> create a folder  task_config and inside task_config create a file config.py. config.py contain the username and password required to pull data from spruce_data website.

      `# mkdir task_config`
      
      `# cd task_config`
      
      `# vi config.py`
      
      `#ftp_username="<username>"` 
      
      `#ftp_password="<password>" `
 `

12. Create a \<ecopad_tasks\> folder inside \<application_short_name/data/static\> .
    
       `# mkdir ecopad_tasks`


       ` All the outputs of simulation,data simulation or forecasting will be generated inside this folder.`
       
       

