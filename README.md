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
  
  ```$ sudo apt-get install ros-<distro>-navigation``` 

### reda

#### Reda

###### Reda   

**REDA**
