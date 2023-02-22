# Test de recrutement iFollow
## Question 1:
Pour installer ROS 18.04 ou 20.04 et la simulation Turtlebot 3 (model burger) et pour configurer la stack de localisation et de navigation, voici les étapes à suivre :

* Installer ROS : 

Pour ROS 18.04, suivez les instructions sur le site officiel : http://wiki.ros.org/melodic/Installation/Ubuntu

Pour ROS 20.04, suivez les instructions sur le site officiel : http://wiki.ros.org/noetic/Installation/Ubuntu

* Installer la simulation Turtlebot 3 (model burger) :
Ouvrez un terminal et entrez les commandes suivantes :

```$ sudo apt-get update
   $ sudo apt-get install ros-melodic-turtlebot3*
   $ echo "export TURTLEBOT3_MODEL=burger" >> ~/.bashrc
```

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
 
 La commande qui nous permet de contrôler le robot en téléopération:
  
  ```$ roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch``` 
 
 La commande qui nous permet de contrôler le robot en lui donnat un nav goal 2D:
  
  ```$ rostopic pub /move_base_simple/goal geometry_msgs/PoseStamped '{header: {stamp: now, frame_id: "map"}, pose: {position: {x: X, y: Y, z: 0}, orientation: {w: 1}}}' ```

  
## Multiplexeur de commande 2:
  
N.B: J'ai choisi de programmer avec Python

Il faut créer un nœud ROS qui multiplexe les commandes de vitesse provenant de deux sources différentes, "cmd_local" et "cmd_web":

```
  #!/usr/bin/env python

import rospy
from std_msgs.msg import Float32

class CommandMux:

    def __init__(self):
        self.cmd_mux_pub = rospy.Publisher('/cmd_mux', Float32, queue_size=1)
        self.cmd_local_sub = rospy.Subscriber('/cmd_local', Float32, self.local_callback)
        self.cmd_web_sub = rospy.Subscriber('/cmd_web', Float32, self.web_callback)
        self.active_source = "local" # set initial source

    def local_callback(self, msg):
        if self.active_source == "local":
            self.cmd_mux_pub.publish(msg.data)

    def web_callback(self, msg):
        if self.active_source == "web":
            self.cmd_mux_pub.publish(msg.data)

    def switch_source(self, source):
        if source == "local":
            self.active_source = "local"
        elif source == "web":
            self.active_source = "web"

if __name__ == '__main__':
    rospy.init_node('command_mux')
    mux = CommandMux()
    rospy.spin()
```

Le nœud "CommandMux" souscrit aux sujets "/cmd_local" et "/cmd_web" pour recevoir les commandes de vitesse provenant de ces sources. Lorsqu'une commande est reçue, le nœud vérifie si la source active correspond à la source de la commande, et si c'est le cas, il publie la commande sur le sujet "/cmd_mux".

La méthode "switch_source" peut être appelée pour changer la source active de la commande, ce qui permet de basculer entre les sources de contrôle. Par exemple, pour basculer vers la source "cmd_web", on peut appeler:
   
  ``` mux.switch_source("web") ```
  
Cela mettra à jour la variable "active_source" dans le nœud et la prochaine commande de vitesse provenant de "/cmd_web" sera publiée sur le sujet "/cmd_mux".
  
## Téléopération à distance 3:
   
Nous souhaitons maintenant contrôler le robot simulé à distance en utilisant le protocole MQTT, plutôt que d'exporter le ROS_MASTER_URI sur un PC distant. Pour ce faire, nous avons besoin d'un broker MQTT pour faire transiter les messages entre les différentes parties.
   
   - Côté simulateur ROS :
   
Pour développer le noeud ROS permettant de souscrire à un topic MQTT et de publier une consigne de vitesse sur le topic ROS /cmd_web, nous allons utiliser le package "paho-mqtt" qui fournit une API python pour interagir avec un broker MQTT.
   
* Installation du package "paho-mqtt" :
   
   ``` $ sudo apt-get install python3-paho-mqtt ```
   
* Création du noeud ROS en pyhton:
   
   ``` 
import rospy
from geometry_msgs.msg import Twist
import paho.mqtt.client as mqtt

class MqttRosBridgeNode:

    def __init__(self):
        # Initialize ROS node
        rospy.init_node('mqtt_ros_bridge_node')
        # Subscribe to MQTT topic
        self.mqtt_client = mqtt.Client()
        self.mqtt_client.on_connect = self.on_connect
        self.mqtt_client.on_message = self.on_message
        self.mqtt_client.connect("localhost", 1883, 60)
        self.mqtt_client.subscribe("robot/cmd_vel")
        # Initialize ROS publisher
        self.cmd_pub = rospy.Publisher('/cmd_web', Twist, queue_size=10)
        # Run the ROS node
        rospy.spin()

    def on_connect(self, client, userdata, flags, rc):
        rospy.loginfo("Connected to MQTT broker with result code "+str(rc))

    def on_message(self, client, userdata, msg):
        # Parse MQTT message and publish to ROS topic
        payload = msg.payload.decode("utf-8")
        values = payload.split(',')
        vel = Twist()
        vel.linear.x = float(values[0])
        vel.angular.z = float(values[1])
        self.cmd_pub.publish(vel)

if __name__ == '__main__':
    try:
        MqttRosBridgeNode()
    except rospy.ROSInterruptException:
        pass
``` 

Le format du message MQTT représentant la consigne de vitesse est défini comme une chaîne de caractères avec deux valeurs séparées par une virgule. La première valeur représente la vitesse linéaire et la deuxième valeur représente la vitesse angulaire. Le nœud ROS se connectera à ce topic MQTT et extraira les deux valeurs pour les publier sur le topic ROS /cmd_web à l'aide d'un objet Twist.
Notez que ce code suppose que le message MQTT reçu est une chaîne de caractères avec deux valeurs séparées par une virgule, en format texte. Vous devrez adapter le code en fonction du format exact de vos messages MQTT.
   

   - Côté client non ROS :   

Nous allons développer un script en Python ou en C++ (indépendant de ROS) qui va observer l'appui des touches directionnelles du clavier pour envoyer une consigne de vitesse via une publication MQTT. Le script doit être capable de se connecter au broker MQTT et de publier sur le topic correspondant à la consigne de vitesse.

   ```
   import paho.mqtt.client as mqtt
import pygame

# Se connecter au broker MQTT
client = mqtt.Client()
client.connect("localhost", 1883, 60)

# Initialiser Pygame pour l'observation des touches
pygame.init()

# Configurer la fenêtre Pygame
size = width, height = 320, 240
screen = pygame.display.set_mode(size)

# Boucle principale
while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            sys.exit()
        elif event.type == pygame.KEYDOWN:
            if event.key == pygame.K_UP:
                # Publier une consigne de vitesse positive sur MQTT
                client.publish("robot/cmd_vel", "0.5")
            elif event.key == pygame.K_DOWN:
                # Publier une consigne de vitesse négative sur MQTT
                client.publish("robot/cmd_vel", "-0.5")
            elif event.key == pygame.K_LEFT:
                # Publier une consigne de rotation positive sur MQTT
                client.publish("robot/cmd_rot", "0.5")
            elif event.key == pygame.K_RIGHT:
                # Publier une consigne de rotation négative sur MQTT
                client.publish("robot/cmd_rot", "-0.5")
``` 
Ce code se connecte au broker MQTT sur "localhost" et publie des messages sur les topics "robot/cmd_vel" et "robot/cmd_rot" en fonction des touches directionnelles appuyées
  
### Le temps dans chaque question:
   
   Après la réception du test, j'ai essayé d'installer Ubuntu 18.04 et ROS-melodic sur ma machine. Ce qui m'as pris 1 jour vu ma machine ne le supporte pas et y avait des problèmes de confusion. En parallèle, je me suis engagé à m'autoformer sur ROS, vu que je n'ai pas travaillé avec depuis plus d'un an. J'ai pris ce challenge de 5 jours pour apprendre à nouveau tous les bases de ROS et résoudre ce test.
   Au début, j'ai pris beaucoup de temps pour résoudre les erreurs de code, et essayer de comprendre la logique de ROS. En fin de compte, j'ai pu dépasser ce problème et résoudre les 3 premières questoins. 
   
   Je tenais à vous remercier pour l'opportunité que vous m'avez offerte de passer ce test. C'était un véritable défi pour moi, mais je suis fier de dire que j'ai donné le meilleur de moi-même.

   Je suis très motivé pour rejoindre votre équipe et je suis convaincu que je pourrais apporter de la valeur ajoutée à votre entreprise. J'attends avec impatience votre retour et j'espère avoir l'opportunité de vous montrer mes compétences et mon engagement.
   
   Je vous remercie encore une fois pour cette opportunité et je reste à votre disposition pour toute information complémentaire.
   
###### Reda   

**REDA**
