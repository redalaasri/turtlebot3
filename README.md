# Test de recrutement iFollow
## Question 1:
Pour installer ROS 18.04 ou 20.04 et la simulation Turtlebot 3 (model burger) et pour configurer la stack de localisation et de navigation, voici les étapes à suivre :

* Installer ROS : 
Pour ROS 18.04, suivez les instructions sur le site officiel : http://wiki.ros.org/melodic/Installation/Ubuntu
Pour ROS 20.04, suivez les instructions sur le site officiel : http://wiki.ros.org/noetic/Installation/Ubuntu
* Installer la simulation Turtlebot 3 (model burger) :
Ouvrez un terminal et entrez les commandes suivantes :

```$ sudo apt-get update```

```$ sudo apt-get install ros-melodic-turtlebot3```

```$ echo "export TURTLEBOT3_MODEL=burger" >> ~/.bashrc```

Remplacez <distro> par la version de ROS que vous avez installée (par exemple, melodic pour ROS 18.04 ou noetic pour ROS 20.04). Dans mon cas, j'ai installé Ubuntu 18.04 alors <distro> = melodic.
  
* Installer la stack de localisation et de navigation :

Entrez la commande suivante dans le terminal :  
  
  ```$ sudo apt-get install ros-melodic-navigation``` 
  
* Créer un environnement de travail:
  
  Entrer les commandes suivantes dans le terminal:

  
  ``` 
  $ mkdir -p ~/catkin_ws/src
  $ cd ~/catkin_ws/
  $ catkin_make
  ``` 

* Télécharger les packages nécessaires :

Entrez les commandes suivantes dans un terminal :
  
``` 
  $ cd ~/catkin_ws/src
$ git clone https://github.com/ROBOTIS-GIT/turtlebot3_simulations.git
$ git clone https://github.com/turtlebot/turtlebot.git
$ git clone https://github.com/turtlebot/turtlebot_msgs.git
$ git clone https://github.com/turtlebot/turtlebot_interactions.git
$ cd ~/catkin_ws/
$ catkin_make
```

  <img width="960" alt="image" src="https://user-images.githubusercontent.com/126022726/220484820-32c6fd4e-2bd1-4a46-b9e2-e36221df86ca.png">

 
* Contrôler le robot en téléopération (clavier) :

Ouvrez un autre terminal et entrez la commande suivante :
  ```$ roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch``` 
  ### reda

#### Reda
  

###### Reda   

**REDA**
