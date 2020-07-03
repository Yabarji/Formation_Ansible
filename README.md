Formation Ansible

Présentation

Exportation au format Markdown
exploitation dans un éditeur de texte
rendu dans un logiciel tel que CuteMarkEd (Windows), ReText(Linux), Serveur Codimd...

Solution Ansible, permet :
le déploiement
la configuration
de nombreuses autres actions sur plusieurs machines "à la fois"

Autres solutions similaires à Ansible
chef
puppet
cfengine
...

Parmi les avantages d'Ansible
très peu de pré-requis
ne nécessite aucun agent sur les postes clients

Les pré-requis 
mécanisme de communication SSH par défaut, possibilité d'autres mécanismes (WinRm pour Windows, Docker pour Docker, ...)
par défaut, l'interpréteur Python pour les postes Linux ou PowerShell pour les postes Windows 

Préparation de la plate-forme
Composition de la plate-forme
un poste maître "ansible" pour piloter ansible
postes clients
1 poste Debian (node-1)
1 poste CentOS (node-2)

Les images Debian et CentOS sont accessibles sur http://192.168.31.38/ansible/.

Préparation du poste client Debian/Ubuntu : node-1
Opérations à effectuer :
L'installer en tant que VM en lui donnant le nom "node-1"
     root: azerty
Vérifier les paramètres réseau : "Accès par pont"
La démarrer
Une fois démarrée :
se connecter en root
noter son adresse IP (192.168.31.50)
installation du package "sudo" (apt install sudo)
sur l'image Debian(nettoyage)
groupdel stagiaire (pb image)
vi /etc/hosts pour remplacer node-2 par debian
création du compte "stagiaire"
useradd -m -s /bin/bash stagiaire
création d'un mot de passe pour stagiaire
                passwd stagiaire
configuration sudo pour le compte stagiaire. Ajouter la ligne suivante avec la commande visudo :
                stagiaire ALL=(ALL) NOPASSWD:ALL
Vérifier la configuration "visudo" en se connectant en tant que "stagiaire" et en effectuant la commande suivante :
sudo whoami

(stagiaire/azerty  -   root/azerty)
                    Debian                  CentOS (ssh root@IP)
Yassen        192.168.31.46    192.168.31.52
Julio            192.168.31.39     192.168.31.53
Laury           192.168.31.47     
Mohammed 192.168.31.48    192.168.31.54
Safia            192.168.31.49    192.168.31.55
Yolande                                  192.168.31.51

Préparation du poste ansible
Il peut s'agir, soit
du poste Ubuntu
d'une VM de votre choix 

En tant que root
Installation du package python3-pip : apt install python3-pip
En tant que stagiaire
pip3 install ansible
création d'une paire de clés SSH privée/publique : ssh-keygen
transfert de la clé publique sur le compte stagiaire du node installé : ssh-copy-id 192.168.31.50
vérification par : ssh 192.168.31.50 hostname
 
 Création du répertoire ansible pour la formation.
 
Premiers pas avec Ansible

Ansible est destiné à effectuer un certain nombre d'opérations sur des postes clients qu'il convient donc de lui définir : on appelle cela l'inventaire.

Création d'un inventaire de départ : fichier "hosts"

Pour des questions d'organisation, on préfère mettre le fichier d'inventaire dans un répertoire, ici nommé "inventory".
On teste l'inventaire par : ansible-inventory -i inventory --list ou --graph

Fichier de configuration d'Ansible
Ansible cherche l'un des trois fichiers suivants (dans l'ordre donné)
ansible.cfg dans le répertoire courant
$HOME/.ansible.cfg à la racine du répertoire personnel du compte
/etc/ansible.cfg

Premier exemple de configuration :
[defaults]
inventory = inventaire

Commande Ad-hoc
Une commande Ad-Hoc est une commande ansible faisant appel à un module à l'aide de l'option "-m".
Exemple n°1
ansible -m ping all

Exemple n°2
ansible -m raw -a "date" all
debian | CHANGED | rc=0 >>
lundi 29 juin 2020, 14:27:19 (UTC+0200)
Shared connection to 192.168.31.50 closed.
Le module "raw" est utile dans la mesure où il ne nécessite pas la présence de python sur le poste client.

Exemple n°3
ansible -m copy -a "src=data.txt dest=/tmp/data.txt" all
La première exécution donne lieu au transfert du fichier -> d'où l'état CHANGED en jaune
La seconde, le fichier étant présent et identique, ne nécessite pas le transfert -> état SUCCESS en vert
Ce principe s'appelle l'idempotence.

Nouvelle version du fichier de configuration

[defaults]
inventory          = inventaire
interpreter_python = auto_silent

Exemple n°4
ansible -m lineinfile -a "path=/tmp/data.txt line='ligne 5'" all

Exemple n°5
ansible -m lineinfile -a "path=/etc/hosts line='192.168.31.38 master'" all
Cet exemple échoue pour des motifs de droits d'accès. Il faudrait être en 'root'.
Ce problème est réglé à l'aide de l'option --become ou de la directive become dans le fichier de configuration.

Nouvelle version du fichier de configuration

[defaults]
inventory          = inventaire
interpreter_python = auto_silent

[privilege_escalation]
become             = yes
become_user        = root

Poste Ansible   ---------------------------------------   Postes clients
Stagiaire                          ---->                              Stagiaire -> root (sudo)

Cas de Rachid
Wilda                              ---->                               Stagiaire -> root (sudo)
Ajout de la ligne suivante dans ansible.cfg (section [defaults])
remote_user = stagiaire
NB: l'envoi de la clé de wilda sur le compte stagiaire des postes clients doit être fait par : ssh-copy-id stagiaire@IP

Type                        Hostname          IP                         Compte     Distribution
Poste Ansible          ansible              192.168.31.38      stagiaire    Debian-10.4
Poste 1 Debian       debian               192.168.31.50      stagiaire    Debian-10.4
Poste 2 CentOS      srv1                   192.168.31.105   stagiaire     CentOS-7.6

Atelier sur les commandes Ad-Hoc

Consigne : Installation d'un serveur Web Apache, en s'assurant qu'il soit actif.
Nom du package
Debian : apache2
CentOS: httpd
Nom du service
Debian : apache2
CentOS : httpd

Module d'installation de package
Debian : apt
CentOS: yum
Module générique : package

Debian
ansible -m package -a "name=apache2 state=present" all
CentOS
ansible -m package -a "name=httpd state=present" all

Module de gestion de service pour forcer l'activation et le démarrage du service
systemd
service

Debian
ansible -m service -a "name=apache2 state=started enabled=yes" all
CentOS
ansible -m service -a "name=httpd state=started enabled=yes" all

Exemple d'un fichier index.html (à créer dans le répertoire files/)
<!DOCTYPE html>
<html>
  <head>
    <title>Accueil</title>
  </head>
  <body>
    <h2>Accueil du site</h2>
  </body>
</html>



Module URI
Permet des requêtes HTTP et HTTPS.
ansible -m uri -a "url=http://192.168.31.50/index.html" all
Pour obtenir le contenu, il faut utiliser l'argument "return_content" :
ansible -m uri -a "url=http://192.168.31.50/index.html return_content=yes" all
    

Les playbooks Ansible
Inconvénients des commandes Ad-Hoc :
non persistentes ! (figurent juste dans l'historique shell...)
indépendantes les unes des autres (on ne peut récupérer le résultat de l'une pour conditionner l'exécution d'une autre)
...

Le recours aux playbook est bien plus confortable.
Un playbook est :
une sorte de scénario
un fichier texte
écrit en YAML
très utile car regroupe l'ensemble des actions que l'on souhaite effectuer sur tout ou partie des postes du parc
...

Le langage YAML (Yet Another Markup Language)
Le YAML :
sert essentiellement à décrire des structures de données
est extrêmement strict
se structure grâce à l'indentation. Le non respect de l'indentation voue le playbook à l'échec.

http://sweetohm.net/article/introduction-yaml.html

Il faut utiliser un éditeur compatible YAML ou configurable en ce sens
Visual Studio Code (Microsoft)
vim (à condition de l'avoir configuré comme il faut).

Configuration VIM à mettre dans le fichier $HOME/.vimrc
set autoindent
set smartindent
set number
set formatoptions+=ro
au! BufNewFile,BufReadPost *.{yam,yml} set filetype=yaml
autocmd FileType yaml setlocal ai ts=2 sts=2 sw=2 expandtab cursorcolumn
highlight CursorColumn cterm=none ctermbg=16

Un playbook est une structure YAML :
    C'est un tableau d'actes, chaque acte étant un hashage avec des clés imposées et des clés libres.
    La seule clé imposée est "hosts".
    
    La clé libre "tasks", décrivant l'ensemble des tâches de l'acte, est un tableau de tâches, chaque tâche étant un hashage appelant un module.

Le premier Playbook
---
- name: Acte I du premier Playbook
  hosts: all

  tasks:
  - name:
    debug:
      msg: "Hola el Mundo"

Notion d'acte
Un acte est un ensemble d'actions effectuées sur un ensemble de postes.

Micro-atelier
Consigne : Constituer le playbook playbook-ping.yml avec une seule tâche appelant le module "ping" sur l'ensemble de clients.
---
- name: Ping des clients
  hosts: all

  tasks:
  - name: Ping des clients
    ping:

Ajout du poste client CentOS

Opérations à effectuer sur le poste CentOS
se connecter en tant que root
création du compte stagiaire
     useradd -m -s /bin/bash stagiaire
affectation d'un mot de passe au compte stagiaire
     passwd stagiaire
configuration sudo pour le compte stagiaire
     Avec la commande visudo, ajout de la ligne suivante :
         stagiaire  ALL=(ALL) NOPASSWD:ALL
    
    Si besoin de nano, l'installer par la commande : yum install nano
    Pour utiliser nano avec visudo :
        EDITOR=nano visudo

Opérations à effectuer sur le poste Ansible
Envoyer la clé publique du compte stagiaire local sur le compte stagiaire du poste CentOS
     ssh-copy-id 192.168.31.105
Tester la bonne configuration
      ssh 192.168.31.105 whoami
Inscrire le nouveau client dans l'inventaire
Tester à l'aide du playbook ping.

Atelier récapitulatif
Consigne : Mise en oeuvre d'un site Web
Installation du serveur Apache
Activation et démarrage du service
Copie du fichier index.html dans /var/www/html

Nom du package
Debian : apache2
CentOS: httpd
Nom du service
Debian : apache2
CentOS : httpd

Module d'installation de package
Debian : apt
CentOS: yum
Module générique : package

Sur les CentOS, il faut désactiver le service firewalld.

Module de gestion de service pour forcer l'activation et le démarrage du service
systemd
service

Pour le moment, il est nécessaire de séparer les actions sur chaque distribution au sein de deux actes différents.

---
- name: Installation sur les machines Debian
  hosts: Debian_hosts

  tasks:
  - name: Installation du package apache2
    package:
      name: apache2
      state: present

  - name: Activation et démarrage du service apache2
    systemd:
      name: apache2
      state: started
      enabled: yes

- name: Installation sur les machines CentOS
  hosts: CentOS_hosts

  tasks:
  - name: Installation du package httpd
    package:
      name: httpd
      state: present

  - name: Activation et démarrage du service httpd
    systemd:
      name: httpd
      state: started
      enabled: yes

  - name: Exception HTTP sur le firewall
    firewalld:
      permanent: yes
      service: http
      immediate: yes
      state: enabled

- name: Copie du fichier index.html
  hosts: all

  tasks:
  - name: Copie du fichier index.html
    copy:
      src: index.html
      dest: /var/www/html/index.html

Options utiles de la commande ansible-playbook
--list-tasks
--list-hosts
--list-tags

Fichier de rejeu : fichier "retry"
Un fichier de rejeu contient la liste des machines sur lesquelles le playbook a échoué.
Cette fonctionnalité doit être activée par la ligne de configuration suivante :
retry_files_enabled   = yes
On peut décider de la localisation des fichiers de rejeu par la ligne suivante :
retry_files_save_path = retry.d
NB: le répertoire sera créé si besoin est.

Une fois l'erreur corrigée, le playbook est ré-exécuté par la commande :
ansible-playbook nom_du_playbook -l @fichier_de_rejeu

Ansible et les variables

Variables d'inventaires
Variables présentes dans un fichier d'inventaire. Par exemple :
[salle_12]
debian    ansible_host=192.168.31.50
srv1      ansible_host=192.168.31.105

[Debian_hosts]
debian    ansible_host=192.168.31.50

[CentOS_hosts]
srv1      ansible_host=192.168.31.105
srv2      ansible_host=192.168.31.106
srv3      ansible_host=192.168.31.107
srv4      ansible_host=192.168.31.108

[CentOS_host:vars]
ansible_python_interpreter=/usr/bin/python3

Inconvénient : une machine présente plusieurs impose autant de présence de ces variables associées (ansible_host)

Variables de groupes et variables d'hôtes
Ces variables sont mise dans des fichiers portant le nom d'un hôte ou d'un groupe, avec l'extension yml, dans le répertoire host_vars/ ou group_vars/, selon les cas.

Variables d'actes (playbook)
Se sont des variables définies directement dans un acte, dans la clé "vars".
---
- name: Ansible et les variables
  hosts: all

  vars:
    couleur: mauve

  tasks:
  - name: Affichage de la variable ansible_host
    debug:
      var: couleur

NB: une variable d'acte n'est accessible qu'au sein de l'acte dans lequel elle est définie.

Extra-variables
Variables définies par l'option -e de la commande ansible-playbook

Possibilité de définir une valeur par défaut lors de l'utilisation d'une variable pouvant ne pas être définie :
---
- name: Ansible et les variables
  hosts: all

  tasks:
  - name: Affichage de la variable couleur
    debug:
      var: couleur|default("cramoisi")
                  ^^^^^^^^^^^^^^^^^^^^
Il est possible de faire à un fichier plutôt que de multiplier les options -e var=valeur :
ansible-playbook playbook-extra-vars.yml -e @extra.yml
                                            ^^^^^^^^^^
On peut également faire appel au module "include_vars" :
---
- name: Ansible et les variables
  hosts: all

  tasks:
  - name:
    include_vars:
      file: extra.yml

  - name: Affichage de la variable couleur
    debug:
      var: couleur

Priorité des variables
Selon une priorité croissante :
Variables de groupes dans l'inventaire
Variables de groupes dans le répertoire group_vars/
Variables d'hôtes dans l'inventaire
Variables d'hôtes dans le répertoire host_vars/
Variables chargées par le module include_vars

Atelier sur les variables
Objectif : simplification du playbook-web.yml afin de n'avoir qu'un seul acte (sans la tâche firewalld)
Choisir un nom de variable explicite
Déterminer le(s) fichier(s) dans lesquels définir la variable.

Déclaration des variables
group_vars/
├── all.yml                    apache_document_root: /var/www/html
├── CentOS_hosts.yml           apache_package_name: httpd
├                              apache_service_name: httpd
├── Debian_hosts.yml           apache_package_name: apache2
├                              apache_service_name: apache2

On peut vérifier par : ansible-inventory --graph

Playbook-web-2.yml
---
- name: Mise en oeuvre d'un site web
  hosts: all

  tasks:
  - name: Installation du serveur apache
    package:
      name: "{{apache_package_name}}"
      state: present

  - name: Activation et démarrage du service apache
    systemd:
      name: "{{apache_service_name}}"
      state: started
      enabled: yes

  - name: Copie du fichier index.html
    copy:
      src: index.html
      dest: "{{apache_document_root}}"

Variables et des conditions

Comment conditionner l'exécution d'une tâche ?
Cela est possible avec la clause "when".

---
- name: Clause when
  hosts: all

  tasks:
  - name: Tâche subordonnée à l'existence de la variable "go"
    debug:
      msg: "Allez zou !"
    when: go|default('no')|bool

Ansible et les faits (Facts)

Les "facts" sont des variables générées par Ansible fournissant un grand nombre d'informations sur chaque node.
Les "facts" sont récoltés par le module "setup".

Commande Ad-Hoc :
ansible -m setup debian

Possibilité de filtrage pour rechercher des facts :
ansible -m setup -a "filter=*mount*" debian

Dans un playbook, les faits sont par défaut automatiquement récoltés.
Cette récolte est conditionnée par la valeur par défaut "implicit" de la directive de configuration "gathering".
Pour désactiver la récolte : mettre "gathering" à "explicit", dans ce cas la récolte ne se fera qu'à condition d'utiliser la clause gather_facts: yes au niveau de chaque acte, dans un playbook.

Mise en cache des faits

Solutions de mise en cache

$ ansible-doc -t cache -l
jsonfile  JSON formatted files
memcached Use memcached DB for cache
memory    RAM backed, non persistent
mongodb   Use MongoDB for caching
pickle    Pickle formatted files
redis     Use Redis DB for cache
yaml      YAML formatted files

Exemple de mise en cache sur fichiers JSON :

[defaults]
inventory             = inventaire
interpreter_python    = auto_silent

gathering             = smart
fact_caching          = jsonfile
fact_caching_connection = facts.json

retry_files_enabled   = yes
retry_files_save_path = retry.d

[privilege_escalation]
become                = yes
become_user           = root

Module lineinfile

Utilisation de module lineinfile pour remplacer une ligne :

---
- name: Module lineinfile
  hosts: all

  tasks:
  - name: Ajout d'une ligne dans le fichier /tmp/data.txt
    lineinfile:
      path: /tmp/data.txt
      line: "Prénom: {{prenom}}"
      regexp: "^Prénom"
      create: yes

Les handlers

Un handler est une tâche dont l'exécution est subordonnée au status "CHANGED" de la tâche qui sollicite le handler.
---
- name: Module lineinfile
  hosts: all

  vars:
    prenom: Zoé

  tasks:
  - name: Ajout d'une ligne dans le fichier /tmp/data.txt
    lineinfile:
      path: /tmp/data.txt
      line: "Prénom: {{prenom}}"
      regexp: "^Prénom"
      create: yes
    notify: affiche_info

  handlers:
  - name: Indiquer que la tâche a conduit à un changement
    debug:
      msg: "Fichier modifié !"
    listen: affiche_info

Par défaut, les handlers sollicités ne s'exécuteront qu'à la fin de l'acte.
Il est possible de déclencher l'exécution des handlers à la suite de la tâche avec la directive :

- meta: flush_handlers

---
- name: Module lineinfile
  hosts: all

  vars:
    prenom: Yassen

  tasks:
  - name: Ajout d'une ligne dans le fichier /tmp/data.txt
    lineinfile:
      path: /tmp/data.txt
      line: "Prénom: {{prenom}}"
      regexp: "^Prénom"
      create: yes
    notify: affiche_info

  - meta: flush_handlers        # <<<------

  - name: Bête tâche
    debug:
      msg: "Bête tâche"

  handlers:
  - name: Indiquer que la tâche a conduit à un changement
    debug:
      msg: "Fichier modifié !"
    listen: affiche_info

Adaptation du playbook-web-3.yml
Fichiers de configuration
$ head group_vars/*
==> all.yml <==
apache_document_root: /var/www/html
apache_port: 86

==> CentOS_hosts.yml <==
ansible_python_interpreter: /usr/bin/python
apache_package_name: httpd
apache_service_name: httpd
apache_port_file: /etc/httpd/conf/httpd.conf

==> Debian_hosts.yml <==
apache_package_name: apache2
apache_service_name: apache2
apache_port_file: /etc/apache2/ports.conf

Playbook-web-3.yml
---
- name: Mise en oeuvre d'un site web
  hosts: all

  tasks:
  - name: Installation du serveur apache
    package:
      name: "{{apache_package_name}}"
      state: present

  - name: Activation et démarrage du service apache
    systemd:
      name: "{{apache_service_name}}"
      state: started
      enabled: yes

  - name: Configuration du port d'écoute d'Apache
    lineinfile:
      path: "{{apache_port_file}}"
      line: "Listen {{apache_port}}"
      regexp: "^Listen"
    notify: restart_apache

  - name: Exception HTTP sur le firewall
    firewalld:
      permanent: yes
      port: "{{apache_port}}/tcp"
      immediate: yes
      state: enabled
    when: ansible_distribution == 'CentOS'

  - name: Copie du fichier index.html
    copy:
      src: index.html
      dest: "{{apache_document_root}}"

  handlers:
  - name: Redémarrage du service Apache
    systemd:
      name: "{{apache_service_name}}"
      state: restarted
    listen: restart_apache

Les Templates Ansible

Un Template est un modèle de fichier qui sera traité par le moteur de template Jinja2 en vue d'effectuer un certain nombre de substitutions pour générer un fichier de sortie qui sera déposé sur les clients.
Les templates sont stockés dans le répertoire templates/.

Exemple simple d'un template (fichier templates/data.txt.j2) :

hostname: {{ansible_hostname}}

Exemple de playbook :

---
- name: Les Templates Jinja2
  hosts: all

  tasks:
  - name: Template
    template:
      src: data.txt.j2
      dest: /tmp/data.txt

Syntaxe Jinja2

Référence : https://jinja.palletsprojects.com/en/2.11.x/templates/

Possibilité d'utiliser :
des commentaires
des tests
des boucles

Exemple :
{# Blablabla #}
{% if ansible_distribution == "Debian" -%}
        Distribution Debian
{% elif ansible_distribution == "CentOS" -%}
        Distribution CentOS
{% else %}
        Distribution non supportée
{% endif %}

Liste des stagiaires:
{% for stagiaire in stagiaires %}
        {{ stagiaire }}
{% endfor %}

Informations sur bob:
{% for cle, valeur in bob.items() %}
        {{cle}}: {{valeur}}
{% endfor %}

Playbook associé
---
- name: Les Templates Jinja2
  hosts: all

  vars:
    stagiaires:
      - Alexandre
      - Hamza
      - Julio
      - Laury
      - Mohammed
      - Moussa
      - Rachid
      - Safia
      - Yassen
      - Yolande

    logins:
      - login: bob
        home: /home/bob
      - login: curt
        home: /home/curt

    bob:
      prenom: bob
      nom: léponge
      lieu: au fond de l'eau

  tasks:
  - name: Template
    template:
      src: data-2.txt.j2
      dest: /tmp/data.txt

Comment afficher un template au lieu de l'envoyer sur les clients (test) ?
Recours à la fonction lookup('template','nom du fichier template') :
---
- name: Les Templates Jinja2
  hosts: all

  vars:
    ...

  tasks:
  - name: Template
    debug:
      msg: "{{lookup('template','data-2.txt.j2').splitlines()}}"

Atelier sur les templates
Objectif : Création, dans le site web, du fichier /var/www/html/node.html contenant les infos suivantes :
    Accueil hostname
    
    Architecture
    Distribution
    Adresse IPv4
    Liste des montages :
        périphérique       point_de_montage

Variables "facts" utiles :
ansible_architecture
ansible_distribution
ansible_default_ipv4_address (??? à vérifier)
Tableau ansible_mounts -> clés device et mount

Solution (1 parmi d'autres)
Tâche à ajouter dans le playbook
  - name: Fichier "templaté" node.html
    template:
      src: node.html.j2
      dest: "{{apache_document_root}}/node.html"

Template node.html.j2 à créer dans le répertoire templates/

<!DOCTYPE html>
<html>
  <head>
    <title>{{ansible_hostname}}</title>
  </head>
  <body>
    <h2>Accueil {{ansible_hostname}}</h2>
        Architecture: {{ansible_architecture}}<br/>
        Distribution: {{ansible_distribution}}<br/>
        Adresse IPv4: {{ansible_default_ipv4.address}}<br/>
        <ul>
        {% for montage in ansible_mounts %}
                <li>{{montage.device}} mounté sur {{montage.mount}}</li>
        {% endfor %}
        </ul>
  </body>
</html>

Les boucles Ansible

Boucle simple
---
- name: Ansible et les boucles
  hosts: debian

  vars:
    couleurs:
      - vert
      - marron tiède
      - gris scintillant
      - noir éblouissant

  tasks:
  - name:
    debug:
      msg: "Couleur {{idx + 1}} : {{couleur}}"
    loop: "{{couleurs}}"
    loop_control:
      index_var: idx
      loop_var: couleur

Boucles imbriquées
---
- name: Ansible et les boucles
  hosts: debian

  vars:
    gentsdelamer:
      - prenom: bob
        nom: léponge
        sports:
          - plongée
          - natation
      - prenom: patrick
        nom: létoile
        sports:
          - escalade
          - danse

  tasks:
  - name:
    debug:
      msg: "{{item.0.prenom}} {{item.1}}"
    loop: "{{gentsdelamer|subelements('sports')}}"

Remarques
item.0 référence l'item de la boucle externe
item.1 référence l'item de la boucle interne
ainsi de suite pour les imbrications supplémentaires éventuelles...

Atelier sur les boucles

Objectifs: Création des groupes et des comptes définis dans le fichier logins.yml
Dans un premier temps (sauf ceux qui sont joueurs) ne pas créer les groupes...

Fichier logins.yml
logins:
  - login: bob
    home: /home/bob
    shell: /bin/bash
    password: azerty
    groupes:
      - admin
      - pedagogie
  - login: curt
    home: /home/curt
    shell: /bin/bash
    password: azerty
    groupes:
      - infos
      - admin

Module de création des groupes : group
Module de création des comptes : user

Génération d'un mot de passe chiffré à partir d'un mot de passe en clair :
---
- name: Génération d'un mot de passe chiffré
  hosts: debian

  vars:
    password: secret

  tasks:
  - name: Chiffrement du password
    debug:
      msg: "{{password | password_hash('sha512', 'abc1213')}}"

Playbook attendu
---
- name: TP sur les boucles
  hosts: all

  tasks:
  - name:
    include_vars:
      file: logins.yml

  - name: Création des groupes
    group:
      name: "{{item.1}}"
    loop: "{{logins|subelements('groupes')}}"

  - name: Création des comptes
    user:
      name: "{{item.login}}"
      home: "{{item.home}}"
      password: "{{item.password|password_hash('sha512','123456')}}"
      shell: "{{item.shell}}"
      groups: "{{item.groupes}}"
    loop: "{{logins}}"


Les rôles Ansible

Un rôle Ansible permet :
de masquer la complexité des tâches et des handlers d'un playbook ;
la réutilisation des fonctionnalités d'un playbook.

Un rôle est une nouvelle clause, au même titre que tasks et handlers.
Une fois le rôle créé ou récupéré d'Internet, son utilisation se réduit aux seules lignes suivantes :

Utilisation d'un rôle existant

---
- name: Utilisation d'un rôle
  hosts: all
  
  roles:
      - role: nom_du_role_1
      - role: nom_du_role_2
      - ...

Création d'un rôle
A un rôle correspond une arborescence située, par défaut, dans le répertoire roles/, à créer si nécessaire.
L'arborescence d'un rôle est créée par la commande suivante :

$ ansible-galaxy init roles/demo
- Role roles/demo was created successfully
$ tree roles
roles
└── demo
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── README.md
    ├── tasks
    │   └── main.yml
    ├── templates
    ├── tests
    │   ├── inventory
    │   └── test.yml
    └── vars
        └── main.yml

Atelier sur les rôles
Objectif : création d'un rôle pour installer un serveur Web.
L'idée est de reprendre le playbook playbook-web-4.yml.
Les variables apache_* qui étaient dépendantes des groupes d'inventaires, doivent dorénavant être dépendantes de la distribution.
Actions à entreprendre :
création de la tâche d'inclusion des fichiers de variables/distribution dans le fichier tasks/main.yml :
- name: Inclusion des variables
  include_vars:
    file: "vars/{{ansible_distribution}}.yml"
transférer les tâches du playbook-web-4.yml dans le fichier tasks/main.yml
transférer le handler du même playbook dans le fichier handlers/main.yml
Création des fichiers vars/Debian.yml et vars/CentOS.yml contenant les variables précédemment définies dans les fichiers group_vars/Debian.yml et group_vars/CentOS.yml les mettant en commentaires dans ces derniers.
Transfert de la variable apache_port du fichier group_vars/all.yml dans le fichier defaults/main.yml.

Il ne reste plus qu'à tester le rôle avec le playbook suivant :
---
- name: Test du rôle apache
  hosts: all

  roles:
  - role: apache
    apache_port: 82

Utilisation de rôles existants

Recherche et infos sur les rôles disponibles :
soit avec ansible-galaxy
soit avec un navigateur

Atelier sur les rôles Galaxy
Objectif:
Installation du rôle geerlingguy.mysql
ansible-galaxy install geerlingguy.mysql -p roles
Création du playbook-role-mariadb
pour installer mariadb avec le mot de passe 'secret' pour l'administrateur
Vérification sur les nodes
ss -tln
systemctl status mariadb
pour y créer les deux bases db1 et db2
On se connecte à mariadb par : mysql -u root -p
Exécuter la commande : SHOW DATABASES;  # Attention au point-virgule final
pour y créer l'utilisateur bob avec le mot de passe 'azerty' et les droits SELECT, INSERT sur la base db1
En étant connecté à mariadb, exécuter la commande : SHOW GRANTS FOR bob@localhost ou bob@'%'

Première version du playbook-role-mysql.yml :

---
- name: Mise en oeuvre d'une serveur MariaDB avec le rôle Geerlingguy.mysql
  hosts: all

  roles:
    - role: geerlingguy.mysql
      mysql_root_password: secret

Chiffrement des données 


Les TAGS



Démonstration Ansible/Docker (si le temps le permet...)


Finalités/objectifs/intérêts de quelques outils

Docker/Podman
Solution de containerisation --> Micro-service

Kubernetes
Orchestrateur de containers (aujourd'hui indépendant de Docker)

Vagrant
Utilitaire de provisionnement de VM de type VirtualBox/VMWare/Hyper-V

Vagrant peut appeler directement un playbook Ansible :

Vagrant.configure("2") do |config|  
  ...
# 
# Run Ansible from the Vagrant Host 
#
config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
   end
end

Terraform
Puppet
Ansible
Solutions de configuration/déploiement de systèmes/VM/Containers
L'avantage de Ansible est qu'il ne nécessite aucun agent sur les postes clients.

FOG
Outils de déploiement d'images.

Thèmes à consolider/maîtriser

Linux : Utilisation+Administration
Réseaux : Fondamentaux (Adresses, routage, DHCP, DNS, quelques services essentiels : web, messagerie, NTP, ...)

Ansible -> Gestion d'un parc (physique, virtuel, container)
Git/Gitlab/Github

Un gros plus :
Docker
Virtualisation

Kubernetes/Puppet/Vagrant/Openshift


