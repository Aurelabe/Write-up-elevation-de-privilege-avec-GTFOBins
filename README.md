# Write-up: elevation de privilege avec GTFOBins

**Rapport sur l'élévation de privilège avec sudo et GTFOBins**

---

**Introduction :**

Ce rapport décrit une procédure d'élévation de privilèges sur une machine Linux en utilisant des commandes autorisées via `sudo` et l'exploitation de binaires spécifiques à l'aide de GTFOBins. L'objectif est de démontrer comment, même avec des restrictions de privilèges, il est possible d'acquérir des privilèges root en exploitant certains outils.

---

**Contexte et Pré-requis :**

Pour réaliser cette manipulation, plusieurs conditions doivent être remplies :

* Une machine fonctionnant sous un système d’exploitation Linux (Debian, Ubuntu, Kali, etc.).
* Un accès root ou un utilisateur ayant les privilèges nécessaires pour configurer l’environnement.
* Un terminal pour exécuter les commandes.

---

**Création d'un nouvel utilisateur et configuration des privilèges :**

La première étape de cette démonstration consistait à créer un nouvel utilisateur avec des privilèges limités. Cela a été fait en utilisant la commande `sudo adduser testuser`. L'utilisateur `testuser` a ensuite été configuré avec des privilèges restreints en modifiant le fichier des sudoers via la commande `visudo`. Une ligne spécifique a été ajoutée pour autoriser l'utilisateur à exécuter seulement les commandes `/usr/bin/find` et `/usr/bin/less` en tant que root, sans avoir besoin de saisir un mot de passe.

Le fichier sudoers pour l'utilisateur `testuser` a été modifié comme suit :

```
testuser ALL=(ALL) NOPASSWD: /usr/bin/find, /usr/bin/less
```

Cette configuration limite l'utilisateur `testuser` à l'exécution de ces deux commandes en mode root, tout en interdisant l'accès à toute autre commande avec des privilèges élevés.

---

**Test des restrictions imposées :**

Une fois les privilèges configurés, il a été important de tester si les restrictions étaient effectivement appliquées. Après s’être connecté en tant que `testuser` via la commande `su - testuser`, les commandes autorisées ont été testées :

```
sudo /usr/bin/find
sudo /usr/bin/less
```

Ces deux commandes ont été exécutées sans problème, confirmant que l’utilisateur pouvait utiliser ces commandes avec des privilèges root. Cependant, lorsqu'une commande non autorisée, telle que `sudo /bin/bash`, a été exécutée, un message d'erreur a été affiché, confirmant que l’utilisateur n'était pas autorisé à exécuter cette commande :

```
Sorry, user testuser is not allowed to execute '/bin/bash' as root...
```

![image](https://github.com/user-attachments/assets/c5717ac4-b135-42cf-a5f1-fef1e50b3cc2)


Cette étape a permis de vérifier que les restrictions de privilèges étaient en place et fonctionnaient comme prévu.

---

**Exploitation des binaires avec GTFOBins :**

L'étape suivante a consisté à exploiter les commandes autorisées par `sudo` pour obtenir un shell avec des privilèges root, malgré les restrictions imposées. GTFOBins, une ressource en ligne dédiée aux binaires exploitables sous sudo, a permis d'identifier deux commandes autorisées susceptibles d'être utilisées pour contourner ces restrictions.

* **Option 1 : Exploitation avec `find`**
  La première méthode d'exploitation utilisée a été la commande `find`. Cette commande permet d'exécuter un shell root à l’aide de l'option `-exec` pour lancer une commande spécifique, ici `/bin/sh`. La commande suivante a permis d’obtenir un shell avec des privilèges root :

  ```
  sudo /usr/bin/find . -exec /bin/sh \; -quit
  ```

  Une fois cette commande exécutée, un shell root a été obtenu. Cela a été confirmé en utilisant la commande `whoami`, qui a retourné `root`.

  ![image](https://github.com/user-attachments/assets/5ff21e66-b9ae-4e2a-8b04-28ec5bc20b56)


* **Option 2 : Exploitation avec `less`**
  La deuxième méthode d'exploitation a impliqué la commande `less`, un programme permettant de visualiser des fichiers. En utilisant la commande suivante pour ouvrir le fichier `/etc/passwd` en mode `less` :

  ```
  sudo /usr/bin/less /etc/passwd
  ```

  Une fois le fichier ouvert, la commande `!sh` a été entrée dans l'interface de `less`, ce qui a permis de lancer un shell root. Cette méthode a également conduit à l'obtention d'un shell avec des privilèges root.

  ![image](https://github.com/user-attachments/assets/97a9297c-8a09-45cc-b1b7-532bfc13dc70)
