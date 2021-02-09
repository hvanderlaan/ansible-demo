![Ansible](https://getvectorlogo.com/wp-content/uploads/2019/01/red-hat-ansible-vector-logo.png)<!-- .element height="100%" width="100%"style="border: 0; background: None; box-shadow: None" -->

Welkom.

---
## Agenda

- Introductie
- Wat je Ansible
- Wanneer gebruik je Ansible
- Ansible best practices
- Ansible en Linux/Unix
- Ansible en Windows
- Ansible en Virtualisatie / Containerisatie
- Vragen ?

---
## Introductie

![Harald van der Laan](https://www.haraldvdl.nl/img/avatar-harald.jpg) <!-- .element height="30%" width="30%"style="border: 0; background: None; box-shadow: None" align="left" -->

Harald van der Laan, Opensource consultant @ AnylinQ. 
- Ruim 16 jaar ervaring in IT omgevingen.
- IOT, Automation en Security
- Hobby fotograaf

---
## Wat is Ansible

Ansible is een deployment en configuration management tool. Doormiddel van playpooks en rollen kan je repetative handelingen uitvoeren.

--
### Deployment tool

Ansible is bij uitstek een tool om een baseline op te zetten. Hierdoor zijn de systemen die je uitrolt altijd identiek.

--
### Configuration management tool

Ansible kan ook gebruikt worden om configuraties op systemen hetzelfde te houden. Echter door de manier waarop Ansible werkt (daemon loos) is het verstandig om hiervoor Ansible Tower / AWX te gebruiken.

---
## Wanneer gebruik je Ansible

Ansible gebruik je als je vaak hetzelfde moet doen, of op plekken waar je gebruik maakt van `cluster-ssh` of andere emulatoren dir op meerdere systemen hetzelfde uitvoeren.

Ansible kan ook gebruikt worden om er voor te zorgen dat configyratie bestanden overal hetzelfde zijn of deze niet veranderen. Hiervoor moet je wel een soort van scheduling gaan gebruiken (bijv: Ansible Tower / AWX).

---
## Ansible best practices

--
### Installatie van Ansible

Ansible is het makkelijkst te installeren op een Linux systeem. Om er voor te zorgen dat je de laatste versie hent kan je dit met pip doen.

```bash
$ yum install python3-pip
$ pip3 install --upgrade pip
$ pip3 install --upgrade ansible
# Als pip problemen heeft met installeren moet men sudi gebruikeb
$ sudo -H pip3 install --upgrade ansible
```

--
### Directorie structuur van Ansible

Om Ansible overzochtelijk te houden zijn een aantal best practices verzonnen. Ook voor de inrichting en de directorie structuur van de Ansible werkdirectorie.

```ini
.
├── ansible.cfg
├── inventory
│       ├── production
│       └── web
├── group_vars
├── playbooks
|   ├── deploy-web-service.yml
|   └── templates
|       └── index.html.j2
├── roles
|   ├── baseline
|   └── weekly-patching
└── site.yaml
```

--
### Ansible modules

Ansible heeft bij installatie al een groot aantal modules. Dit zijn standaard `commando's` die je kunt gebruiken om acties uiy te voeren op een systeem. Als je Ansible gebruikt maak dan ook gebruiik van de standaard modules die meegeleverd worden. Hierdoor worden je playbooks idempotent. En zal Ansible ze niet meer uitvoeren als dit niet nodig is.

--
## Ansible inventory

De Ansible inventory is een bestand of script die een lijst met systemen aan Ansible geeft. Deze lijst is een lijst met losse systemen of groepen van systemen waar je Ansible playbooks op kan uitvoeren.

```ini
[appservers]
redhat.example.com
debian.example.com

[vmhosts]
esxi01.example.com
vcenter.example.com

[nederland:children]
linux
vmhosts
```
--
### Ansible playbooks

Ansible playbooks zijn bestanden die Ansible gebruikt om taken uit te voeren. De playbooks worden in `YAML` geschreven. In een playbook staat minimaal de systemen waar het uitgevoerd op moet worden, en de taken of rollen die uitgevoerd moeten worden.

Voorbeeld playbook:
```ini
- hosts: all           # draai op all systemen in de inventory
  gather_facts: true   # verzamel all ansible variabelen

  tasks:               # Taak definitie
    - name: maak gebruikers aan        # naam vab de taak
      user:                            # module naam
        name: haraldvanderlaan         # module optie
        home: /home/haraldvanderlaan   # module optie
        shell: /usr/bin/bash           # module optie
```
--
### Ansible rollen

Om te voorkomen dat je playbooks of taken op meerder plaatsen gaat aanmakan kan je ook rollen maken. Een rol is een set van playbooks die je in een ander playbook kan gebruiken. Hierbij moet je denken aan een rol die users installeerd of een bepaalde applicatie installeerd.

```ini
- hosts: appservers
  gather_facts: true

  roles:
    - {role: httpd, when: ansible_distribution == "RedHat" }
    - {role: apache, when: ansible_distribution == "Debian" }
    - {role: iis, when: ansible_distribution == "Windows" }
```

--
### Ansible rollen: Directorie structuur

Zoals ansible ook zijn eigen directoie structuur heeft, hebben rollen dar ook.

```ini
testrole/
├── defaults         ├── tasks
│   └── main.yml     │   └── main.yml
├── files            ├── templates
├── handlers         ├── tests
│   └── main.yml     │   ├── inventory
├── meta             │   └── test.yml
│   └── main.yml     └── vars
└── README.md            └── main.yml
```

--
### Gebruik van variabelen

Ansible kan gebruiken maken van variabelen, deze mogen in de inventory, playbooks, apparte variabelen bestand staan. Maar kunnen ook opde commandline worden meegegeven. Daarnaast heeft Ansible ook nog eigen variabelen die gebruikt kunnen worden in een playbook of een role. Deze variabelen kannen geladen worden door `gather_facts: true` in je playbook te zetten.

Een aantal voorbeelden van Ansible variabelen
- ansible_distribution
- ansible_fqdn
- ansible.ens0p3.ipv4
--
### Beveiliging van playbooks en variabelen

Met Ansible kan je veel verschillende dingen beheren, dit brengt echter wel een beveiligings risico met zich mee. Gevoellige invormatie, zoals gebruikersnamen, wachtworden, certificaten, kunnen overal in de playbooks of rollen staan. En dan hebben we het nog niet eens over de credentials die gebruikt worden door Ansible zelf om in te kunnen loggen.

Om dit toch allemaal veilig te kunnen gebruiken kan je ansible-vault gebruiken.

--
### Ansible-vault

Ansible-vault is een tool voor ansible om playbooks, variabelen bestanden en inventories te kunnen encrypten. Hierdoor is de inhoud van het bestand niet meer leesbaar zonder een wachtwoord. Ansible kan deze bestanden `on-the-fly` decrypten en het daarna gebruiken

```bash
$ ansible-vault encrypt inventory/prod
$ ansible-playbook -i inventory/prod site.yaml --ask-vault-pass
```

--
### Syntax checking van je playbooks

Ondank YAML een redelijk makkelijk te leren taal is, maak je toch redelijk snel foutjes in de code. Je kan natuurlijk gewoon het ansible commando draaien. Om er later achter te komen dat er een fout in zit. Maar gelukkig heeft Ansible ook een syntax checker. 

***Let op:*** De fout melding van Ansible zijn niet altijd even duidelijk.

---
## Ansible en Linux/Unix

Ansible en Linux/Unix gaan erg goed samen. Ansible bruikt een `ssh connectie` met een unprivileged user account. Ansible kan gebruik maken van `sudo` of `su` om extra privileges te krijgen. Als je geen public en private ssh key's gebruikt moet er op de Ansible server een extra python module worden geinstallerd (sshpass). De voorkeur is dan ook om ssh-key's the gebruiken.

--
### Linux playbook voorbeeld

```ini
- hosts: all
  gather_facts: true
  become: true

  tasks:
    - name: instaleer laatste updates met yum
      yum:
        name: '*'
        state: latest
      when: ansible_distribution == 'RedHat'

    - name: zet SELinux aan op RedHat
      selinux:
        policy: targeted
        status: enforcing
      when: ansible_distribution == 'RedHat'
```


---
## Ansible en Windows

Met Ansible kan je ook Windows systemen beheren. Dit kan ook met ssh, maar de meesten gebruiken `winrm`. Ook zijn er een aantal standaard windows modules voor Ansible. Hierbij moet je denk aan win_users, win_msi. Ook kan Ansible powershell commando's/script plaatsen en draaien.

Als je Winrm gebruikt oet je op de Ansible server pywinrm installeren. Ook moet je op het Windows systeem winrm configureren

https://docs.ansible.com/ansible/latest/user_guide/windows_setup.html

--
### Windows playbook voorbeeld

```ini
- hosts: windows
  gather_facts: true
  become: true

  tasks:
    - name: maak gebruiker Harald van der Laan aan
      win_user:
        name: haraldvanderlaan
        fullname: "Harald van der Laan"
        password: LekkerVeiligInPlainText
        groups:
          - Users
        state: present
```
---
## Ansible en virtualisatie / containerisatie

Buiten heb beheer van een operating system kan Ansible ook worden ingezet als deployment tool. Denk hierbij aan het aanmaken van een nieuwe virtuele server of het deployen van een container. Voor meerder virtualisatie producten en container producten zijn standaard modules.
--
### Ansible en VMWare

Ansible kan heel goed VMWare taken uitvoeren zoals het aanmaken van een nieuwe server, het starten/stoppen en snapshots maken of verwijderen. Zo kan je een rol maken die alle systemen upgrade, maar eerst een snapshot maakt. Als dan alles goed is gegaan de snapshot verwijderd of terug zet als het fout is gegaan.

Voor het beheer van VMWare is de python module pyvmomi nodig.

https://docs.ansible.com/ansible/latest/scenario_guides/guide_vmware.html

--
### Ansible en Docker

Ook container platformen zoals docker kunnen door Ansible worden gebruikt. Van het uitrollen van docker swarm tot het starten/stoppen of updaten van een container kan door Ansible worden gedaan.

voor het beheren van docker omgevingen is de python3 docker module nodig op de Ansible server

https://docs.ansible.com/ansible/latest/scenario_guides/guide_docker.html

---
## Vragen ?

![questions](https://base.imgix.net/files/base/ebm/machinedesign/image/2019/04/machinedesign_16431_promoaskingquestions_917203022_0.png?auto=format&fit=max&w=1200)

---
## Fork me on github

![Fork me on github](https://cdn.slidesharecdn.com/ss_thumbnails/forkme-131226180611-phpapp02-thumbnail-4.jpg?cb=1388081215)

https://github.com/hvanderlaan/ansible-demo.git

---
## Ansible Tower / AWX

In deze kennissessie heb ik het een aanral keer over Ansible Tower en AWX gehad.
Daarom een kleine demo van AWX.