# Linux Server Configuration


Este é o projeto final do nanodegree de full stack web developer

Aqui vemos como configurar corretamente uma distribuição linux,  instalar e configurar um servidor web para servir uma aplicação web
- A distribuição linux é [Ubuntu](https://www.ubuntu.com/download/server) 16.04 LTS.
- A vps é [Amazon Lighsail](https://lightsail.aws.amazon.com/).
- A aplicação é  [Restaurant Menu App](https://github.com/LorenzoTomaz/Restaurant-Menu-App).
- O DB é [PostgreSQL](https://www.postgresql.org/).
- Minha máquina local roda Ubuntu 18.04

O endereço da aplicação é http://100.24.107.188.xip.io.

## Obtenha uma VPS

### 1: Compre uma Instância da Amazon Lightsail

- Faça o Login [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/resources) usando uma conta da amazon web services.
- Clique `Create instance`. 
- Escolha `Linux/Unix`, `OS Only` e  `Ubuntu 16.04 LTS`.
- Clique em `Create` para criar uma instância.


**Referência**
- [Como criar um server na Amazon Lightsail](https://serverpilot.io/community/articles/how-to-create-a-server-on-amazon-lightsail.html).


### 2: Conecte-se com a VPS com SSH

- No menu clique em `SSH keys` e baixe a chave privada.
- Mova o arquivo da chave privada nomeado `LightsailDefaultPrivateKey-*.pem` para o diretório `~/.ssh` e o renomeie como `lightsail_key.rsa`.
- No shell digite: `chmod 600 ~/.ssh/lightsail_key.rsa`.
- Digite: `ssh -i ~/.ssh/lightsail_key.rsa ubuntu@100.24.107.188` para se conectar a VPS 


## Atualizações e Segurança

### 3: Atualize pacotes

```
sudo apt-get update
sudo apt-get upgrade
```


### 4: Mude a porta SSH

- Edite o arquivo com `sudo nano /etc/ssh/sshd_config`.
- Mude o número da porta de `22` para `2200`.
- Salve o arquivo.
- reinicie o  SSH: `sudo service ssh restart`.

### 5: Configurando UFW

- Configure o firewall para permitir apenas as conexões SSH (porta 2200), HTTP (porta 80), and NTP (porta 123).
  ```
  sudo ufw status                 
  sudo ufw default deny incoming   
  sudo ufw default allow outgoing  
  sudo ufw allow 2200/tcp          
  sudo ufw allow www               
  sudo ufw allow 123/udp           
  sudo ufw deny 22                 
  ```

- No shell, digite `sudo ufw enable`. Voce verá aparecer a seguinte mensagem na tela:
  ```
  Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
  Firewall is active and enabled on system startup
  ```

- Veja a lista de permissões do ufw com o comando `sudo ufw status`. Voce verá aparecer a seguinte mensagem na tela:

  ```
  Status: active
  
  To                         Action      From
  --                         ------      ----
  2200/tcp                   ALLOW       Anywhere                  
  80/tcp                     ALLOW       Anywhere                  
  123/udp                    ALLOW       Anywhere                  
  22                         DENY        Anywhere                  
  2200/tcp (v6)              ALLOW       Anywhere (v6)             
  80/tcp (v6)                ALLOW       Anywhere (v6)             
  123/udp (v6)               ALLOW       Anywhere (v6)             
  22 (v6)                    DENY        Anywhere (v6)
  ```

- termine a conexão ssh com o comando: `exit`.

- Clique na opção  `Manage` na sua instância na página da amazon, 
- Clique na opção `Networking`  e então mude a configuração do firewall para coincidir com a configuração do firewall interno

- Permitir conexões nas portas 80(TCP), 123(UDP), and 2200(TCP), e não permitir na porta porta 22.

- Na sua máquina local execute o comando: `ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@100.24.107.188`


**Referências**
- [UFW ](https://help.ubuntu.com/community/UFW).
- [How to install and use Uncomplicated Firewall in Ubuntu](https://www.techrepublic.com/article/how-to-install-and-use-uncomplicated-firewall-in-ubuntu/).





<a name="step_5_1"></a>
### 5: Instalando atualizações automaticamente

O pacote `unattended-upgrades` pode ser usado para instalar automaticamente atualizaçoes do sistema 
- Permite atualizações de segurança automáticas com o comando: `sudo apt-get install unattended-upgrades`.
- Edite o arquivo `/etc/apt/apt.conf.d/50unattended-upgrades`, e retire a tag de comentário da linha `${distro_id}:${distro_codename}-updates`, salve o arquivo.
- Edite o arquivo`/etc/apt/apt.conf.d/20auto-upgrades` para que as atualizações sejam instaladas todo os dias:
  ```
  APT::Periodic::Update-Package-Lists "1";
  APT::Periodic::Download-Upgradeable-Packages "1";
  APT::Periodic::AutocleanInterval "7";
  APT::Periodic::Unattended-Upgrade "1";
  ```
- Use o comando: `sudo dpkg-reconfigure --priority=low unattended-upgrades`.


**Referências**
- [Automatic Updates](https://help.ubuntu.com/lts/serverguide/automatic-updates.html).
- [AutomaticSecurityUpdates](https://help.ubuntu.com/community/AutomaticSecurityUpdates).



## Usuário `grader` 


### 6: Criando usuário `grader`

- Enquanto vc esta logado como usuario `ubuntu`, adicione o usuário: `sudo adduser grader`. 


### Step 7: Dando ao usuário `grader` permissões de superusuário

- Edite o arquivo sudoers com o comando: `sudo visudo`.
- Procure esta linha:
  ```
  root    ALL=(ALL:ALL) ALL
  ```

- Adicione na linha abaixo para dar permissões ao usuário `grader`.
  ```
  grader  ALL=(ALL:ALL) ALL
  ```

- Salve o arquivo.
- Verifique que o usuário `grader` tem permissões de superusuário. Digite `su - grader`, preencha a senha, 
digite `sudo -l` e preencha a senha novamente. Voce verá a seguinte mensagem após o último comando:

  ```
  
  User grader may run the following commands :
      (ALL : ALL) ALL
  ```

**Resources**
- [How To Add and Delete Users on an Ubuntu 14.04 VPS](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)


### 8: Criando um par de chaves para o user `grader` usando `ssh-keygen` 

- Na máquina local:
  - digite `ssh-keygen`
  - Digite `grader_key`
  - Digite a frase de segurança duas vezes. Dois arquivos seram gerados (  `~/.ssh/grader_key` e `~/.ssh/grader_key.pub`)
  - Execute o comando `cat ~/.ssh/grader_key.pub` e copie o seu conteudo
  - Faça o login na VPS
- VPS:
  - Crie um diretório chamado `~/.ssh`.
  - Digite `sudo nano ~/.ssh/authorized_keys` e copie o conteúdo do arquivo `~/.ssh/grader_key.pub`, salve o arquivo.
  - Dê permissões : `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
  - Veja se o arquivo `/etc/ssh/sshd_config` tem a opção `catalogAuthentication` configurada com `no`
  - Reinicie a conexão ssh: `sudo service ssh restart`
- Na máquina local: `ssh -i ~/.ssh/grader_key -p 2200 grader@100.24.107.188`.


**Referências**
- [How To Set Up SSH Keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2).
- [SSH/OpenSSH/Keys](https://help.ubuntu.com/community/SSH/OpenSSH/Keys).





### 9: Configurando a TimeZone

- Enquanto vc esta logado com o usuario `grader`, configure a time zone: `sudo dpkg-reconfigure tzdata`. Após a configuração vc verá:

  ```
  Current default time zone: 'America/Belem'
  Local time is now:      Thu Oct 19 21:55:16 EDT 2017.
  Universal Time is now:  Fri Oct 20 01:55:16 UTC 2017.
  ```

**Referências**
- [UbuntuTime](https://help.ubuntu.com/community/UbuntuTime)
- [How do I change my timezone to UTC/GMT?](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt/138442)



### 10: Instalando e configurando Apache para servir a aplicação flask com mod_wsgi

- logado com o usuário `grader`, instale o servidor Apache: `sudo apt-get install apache2`.
- Enter public IP of the Amazon Lightsail instance into browser e veja a página do apache no browser.

- Instale o pacote mod_wsgi:  
 `sudo apt-get install libapache2-mod-wsgi-py3`.
- Habilite `mod_wsgi` com: `sudo a2enmod wsgi`.


### 11: Instalando e configurando PostgreSQL

- logado com o usuário `grader`, instale PostgreSQL:
 `sudo apt-get install postgresql`.

- Digite: `sudo su - postgres`.
- Abra o terminal do PostgreSQL com comando `psql`.
- Crie o usuário `catalog` com o comando:
  ```
  postgres=# CREATE ROLE catalog WITH LOGIN catalog 'catalog';
  postgres=# ALTER ROLE catalog CREATEDB;
  ```

- Veja a lista de usuários com o comando: `\du`. Voce verá a seguinte mensagem após o comando anterior:
  ```
                                     List of roles
   Role name |                         Attributes                         | Member of 
  -----------+------------------------------------------------------------+-----------
   catalog   | Create DB                                                  | {}
   postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
  ```

- Saia do terminal interativo: `\q`.
- Saia da linha de comando do PostgreSQL.
- Crie um novo usuário chamado `catalog`: `sudo adduser catalog`.
- Dê ao usuário `catalog` permissões de superusuário assim como fizemos passo 7.
- Logado com o usuário `catalog`, create o banco de dados: `createdb catalog`.
- Execute `psql` e então digite `\l` para ver a lista de bancos criados:
  ```
                                    List of databases
     Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
  -----------+----------+----------+-------------+-------------+-----------------------
   catalog   | catalog  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
   postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
   template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |          |          |             |             | postgres=CTc/postgres
   template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |          |          |             |             | postgres=CTc/postgres
  (4 rows)
  ```
- Saia como comando: `\q`.
- Volte ao terminal do ubuntu com Ctrl + D

**Referência**
- [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps).



### 12: Instalando o git

- While logged in as `grader`, install `git`: `sudo apt-get install git`.

## Fazendo o Deploy do Projeto

### 13.1: Clone o projeto do github

- logado com o usuário `grader`, crie o diretório `/var/www/catalog/`.
- Clone o diretório e mude seu nome para `catalog`:<br>
`sudo git clone https://github.com/LorenzoTomaz/Restaurant-Menu-App.git catalog`.
- Mude a permissão do diretório `/var/www/catalog` para o usuário `grader` using: `sudo chown -R grader:grader catalog/`.
- Vá ao diretório `/var/www/catalog/catalog`.

- No arquivo `database_setup.py`, comente a linha superior e adicione a linha inferior no texto abaixo:
   ```
   # engine = create_engine("sqlite:///catalog.db")
   engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
   ``` 
- No arquivo `lotsofmenus.py`, comente a linha superior e adicione a linha inferior no texto abaixo:
   ```
   # engine = create_engine("sqlite:///catalog.db")
   engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
   ``` 

- No arquivo `project.py`, comente a linha superior e adicione a linha inferior no texto abaixo:
   ```
   # engine = create_engine("sqlite:///catalog.db")
   engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
   ``` 



### 13.2: Autenticando o Login com o Google

- Vá ao site [Google Cloud Plateform](https://console.cloud.google.com/).
- Clique em `APIs & services`
- Clique em `Credentials`.
- Crie uma id do OAuth e adicione http://100.24.107.188 and 
http://100.24.107.188.xip.io como authorized JavaScript 
origins.
- Adicione as urls http://100.24.107.188.xip.io/, http://100.24.107.188.xip.io/login, http://100.24.107.188.xip.io/oauth/google, http://100.24.107.188.xip.io/clientOAuth, http://100.24.107.188.xip.io/gconnect/, http://100.24.107.188.xip.io/gdisconnect/ 
como authorized redirect URI.
- Baixe o arquivo JSON.
- Abra o arquivo `/var/www/catalog/catalog/client_secret.json` e copie o conteúdo do arquivo baixado no item anterior.
- Substitua a id do cliente  no arquivo `templates/main.html`.


### 14.1: Instale o venv

- Logado com o usuário `grader`, instale o pip: `sudo apt-get install python3-pip`.
- Instale o venv: `sudo apt-get install python3-venv`
- Vá ao diretório `/var/www/catalog/catalog/`.
- Crie o venv: `sudo python3 -m venv .venv3`.
- Dê permissões ao usuário `grader` ao ambiente virtual com: `sudo chown -R grader:grader .venv3/`.
- Ative o venv: `source .venv3/bin/activate`.
- Instale as dependências:
  ```
  pip3 install -r requirements.txt
  ```

- Execute o comando `python3 project.py`:
  ```
  * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
  ```

- Desative o venv: `deactivate`.

**Referências**
- [virtualenv](http://flask.pocoo.org/docs/0.12/installation/).
- [Create a Python 3 virtual environment](https://superuser.com/questions/1039369/create-a-python-3-virtual-environment).






### 14.2: Configurando o Host

- Adicione a linha abaixo no arquivo `/etc/apache2/mods-enabled/wsgi.conf` para usar python3

  ```
  #WSGIPythonPath directory|directory-1:directory-2:...
  WSGIPythonPath /var/www/catalog/catalog/.venv3/lib/python3.5/site-packages
  ```

- Crie o arquivo `/etc/apache2/sites-available/catalog.conf` e adicione as linhas abaixo:

  ```
  <VirtualHost *:80>
    ServerName 100.24.107.188.xip.io
  ServerAlias 100.24.107.188.xip.io
    WSGIScriptAlias / /var/www/catalog/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
          Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
          Order allow,deny
          Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>

  ```

- Habilite o virtual host com o comando: `sudo a2ensite catalog`:
  ```
  Enabling site catalog.
  To activate the new configuration, you need to run:
    service apache2 reload
  ```

- Digite o comando para subir as modificações: `sudo service apache2 reload`.

**Resources** 
- [Getting Flask to use Python3 (Apache/mod_wsgi)](https://stackoverflow.com/questions/30642894/getting-flask-to-use-python3-apache-mod-wsgi)
- [Run mod_wsgi with virtualenv or Python with version different that system default](https://stackoverflow.com/questions/27450998/run-mod-wsgi-with-virtualenv-or-python-with-version-different-that-system-defaul)


### 14.3: Configurando a Aplicação

- Crie o arquivo `/var/www/catalog/catalog/catalog.wsgi` e adicione as linhas abaixo:

  ```

  #!/usr/bin/python3
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/catalog/")
  sys.path.insert(1, "/var/www/catalog/")

  from project import app as application
  
  ```

- Reinicie o Apache: `sudo service apache2 restart`.

**Resource** 
- Flask documentation, [Working with Virtual Environments](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/#working-with-virtual-environments)


### 14.4: Populando o DB

- No diretório `/var/www/catalog/catalog/` directory, 
ative o venv: `source .venv3/bin/activate`.
- Execute `python3 database_setup.py`
- Execute `python3 lotsofmenus.py`

- Desative o venv: `deactivate`.

### 14.5: Desabilitando o default do Apache

- Desabilite o default do apache2: `sudo a2dissite 000-default.conf`:

  ```
  Site 000-default disabled.
  To activate the new configuration, you need to run:
    service apache2 reload
  ```

- Digite o comando para subir as modificações: `sudo service apache2 reload`.

### 14.6: Deploy

- Mude a permissão do diretório: `sudo chown -R www-data:www-data catalog/`.
- Reinicie Apache novamente: `sudo service apache2 restart`.
- Abra o seu navegador http://100.24.107.188 or http://100.24.107.188.xip.io.



## Diretório da aplicação

Depois de todos os passos vc verá a seguinte estrutura de arquivos:

```
└── catalog
    ├── catalog.wsgi
    ├── client_secrets.json
    ├── database_setup.py
    ├── database_setup.pyc
    ├── lotsofmenus.py
    ├── project.py
    ├── __pycache__
    │   ├── database_setup.cpython-35.pyc
    │   ├── database_setup.cpython-36.pyc
    │   └── project.cpython-35.pyc
    ├── README.md
    ├── req.py
    ├── requirements.txt
    ├── restaurantmenu.db
    ├── restaurantmenuwithusers.db
    ├── static
    │   ├── blank_user.gif
    │   ├── bootstrap-social.css
    │   ├── bootstrap-social.scss
    │   └── styles.css
    ├── .venv3
    ├── templates
    │   ├── deletemenuitem.html
    │   ├── deleteRestaurant.html
    │   ├── editmenuitem.html
    │   ├── editRestaurant.html
    │   ├── header.html
    │   ├── login.html
    │   ├── main.html
    │   ├── menu.html
    │   ├── newmenuitem.html
    │   ├── newRestaurant.html
    │   ├── publicmenu.html
    │   ├── publicrestaurants.html
    │   └── restaurants.html
    └── wsgi.py
 
```

## Referências

- DigitalOcean [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

