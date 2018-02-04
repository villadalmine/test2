# DEMO POC

Un ejemplo de ANSIBLE para levantar un stack basico de aws con un wordpress

## Ansible Playbook.
El playbook consta de tres roles

1. base: En donde realiza tareas basicas como.
   * Instalar paquetes docker y dockercompose.
   * Configurar sudoers
   * Configurar permisos de grupo para ec2-user
   * Actualizar el sistema operativo.
   * Reiniciar servicio docker
   * Configurar el acesso ssh para poder conectarse con ec2-user y deployar todo.
2. distrostack: En donde deploya toda la infra VPC y EC2 con sus componentes usando cloudformation.
   * Usa templates de cloudformation ya parametrizado con variables basicas.
   * Dos templates hay en el ejemplo, uno crea toda la VPC desde cero y la otra crea instancia usando la VPC por defecto de la zona.
3. dockercompose: Levanta un docker con wordpress + mysql
   * Tiene un template del docker-compose parametrizado
   * Crea la estructura que necesita el dockercompose y la copia en el target que se creo para deployar y levantar el container.

El demo es limitado, no esta totalmente paremetrizado, esta hecho para un ejemplo basico y se puede seguir mejorando para algo mas particular.

## Variables Playbook

1. Todos los roles tienen un directorio roles/*/defaults/main.yml
   * Aca se setean por defecto las variables por defecto de cada rol, se puede definir otras desde el playbook en la seccion correspondiente o tocar directamente las defaults.
2. Variables en Playbook
   * Tener en cuenta que el nombre - hosts: tag_Name_demopoc_docker se tiene que declar igual que las variables stack_name y stack_app, por defecto usa demopoc_docker en el playbook demo1 y en el demo2 usa demopoc2_docker2.
3. El archivo keys/private/aws_private.yml
   * Contiene la variable aws_secret_key , aws_access_key, user,sshcfg, pathkeys.
   * pathkey  indica donde esta la key privada de aws para loguearse a la instancia.
   * sshcfgpath indica la ruta donde graba el ssh.cfg

4. El archivo ec2.ini se puede editar para traer en la busqueda solo las instancias que queremos.

## Archivo ansible.cfg

```
#https://raw.githubusercontent.com/ansible/ansible/devel/examples/ansible.cfg
[ssh_connection]
ssh_args = -F $USER/playbook/ssh.cfg -o ControlMaster=auto -o ControlPersist=30m
control_path = ~/.ssh/ansible-%%r@%%h:%%p

[defaults]
retry_files_enabled = False
host_key_checking = False
roles_path = $USER/playbook/roles
deprecation_warnings = False
```
Se debe poner en donde dice $USER el path en donde uno clone el repositorio.
```

git clone git@github.com:villadalmine/test2.git
cd test2
pwd (capturar salida y remplazar por $USER)
```
### Remover Playbook
```
ansible-playbook remove-provision-demo1.yml  -vvv
```

### Ejecutar Playbook
```
ansible-playbook -i ec2.py  aws-provision-demo1.yaml  -vvv
```
## Para ver las salidas de los comandos
Ver el archivo removedemo.out y createdemo1.out


# INSTALAR DESDE CERO EN UNA DISTRO BASADA EN FEDORA
Instalo ANSIBLE
```
[root@rosas ~]# dnf install ansible libselinux-python boto awscli
Last metadata expiration check: 2:17:13 ago on Sat 03 Feb 2018 08:00:39 PM EST.
Dependencies resolved.
[root@rosas playbook]# pip install boto3
WARNING: Running pip install with root privileges is generally not a good idea. Try `pip install --user` instead.
Collecting boto3
[root@rosas playbook]# setenforce 0
```

COnfigurar aws credentials para el usario

```
[root@rosas ~]# aws configure
AWS Access Key ID [None]: rino
AWS Secret Access Key [None]: rino
Default region name [None]: us-east-2
Default output format [None]: json
[root@rosas ~]# cat /root/.aws/credentials
[default]
aws_access_key_id = rino
aws_secret_access_key = rino
[root@rosas ~]# cat /root/.aws/config
[default]
region = us-east-2
output = json
[root@rosas ~]#

```
Chequeo si esta andando las keys

```

[root@rosas ~]#   aws sts get-caller-identity
{
    "UserId": "asdads",
    "Account": "asdasd",
    "Arn": "arn:aws:iam::asdasd:user/asd"
}
[root@rosas ~]#

```
Me clono le repo.

```
[root@rosas ~]# mkdir repos
[root@rosas ~]# cd repos
[root@rosas repos]# pwd
/root/repos
[root@rosas repos]# git clone git@github.com:villadalmine/test2.git
Cloning into 'test2'...
Warning: Permanently added the RSA host key for IP address '192.30.253.113' to the list of known hosts.
Enter passphrase for key '/root/.ssh/id_rsa':
remote: Counting objects: 79, done.
remote: Compressing objects: 100% (51/51), done.
remote: Total 79 (delta 8), reused 75 (delta 7), pack-reused 0
Receiving objects: 100% (79/79), 29.68 KiB | 333.00 KiB/s, done.
Resolving deltas: 100% (8/8), done.
[root@rosas repos]#
```

Armo la estructura con mis keys y el archivo ansible.cfg.
```
[root@rosas ~]#cd test2; cd playbook
[root@rosas playbook]# cp ansible.example.cfg ansible.cfg
[root@rosas playbook]# pwd
/root/repos/test2/playbook
[root@rosas playbook]#
[root@rosas playbook]# cat ansible.cfg
#https://raw.githubusercontent.com/ansible/ansible/devel/examples/ansible.cfg
[ssh_connection]
ssh_args = -F /root/repos/test2/playbook/ssh.cfg -o ControlMaster=auto -o ControlPersist=30m
control_path = ~/.ssh/ansible-%%r@%%h:%%p

[defaults]
retry_files_enabled = False
host_key_checking = False
roles_path = /root/repos/test/playbook/roles
deprecation_warnings = False
[root@rosas test2]#

```
Me genero las llaves de amazon en el directorio correspondiente.
```
[root@rosas test2]# cd keys/private/
[root@rosas private]# ls
aws_private.yml.example  key.example
[root@rosas private]# cp key.example RINO_OHIO.pem
[root@rosas private]#chmod 600 RINO_OHIO.pem
```
Me genero bien el archivo aws_private.key con los datos correspondientes
```
[root@rosas private]# ls
RINO_OHIO.pem  aws_private.yml.example  key.example
[root@rosas private]# cp aws_private.yml.example aws_private.yml
[root@rosas private]# vi aws_private.yml
[root@rosas private]# cat aws_private.yml
#Para conectar a amazon poner aca las key
aws_access_key: aws-access-key
aws_secret_key: aws-secret-key
#Para hacer andar las key se necesita poner la ruta en donde esta la key de amazon
pathkeys: "/root/repos/test2/playbook/keys/private/RINO_OHIO.pem"
#Se necesita poner la ruta del ssh.cfg en el mismo directorio que el playbook
sshcfgpath: "/root/repos/test2/playbook/ssh.cfg"
user: ec2-user
[root@rosas private]#chmod 600 aws_private.yml
```
Realizo un chequeo de sintaxis.
```
root@rosas private]# cd ..
[root@rosas keys]# cd ..
[root@rosas playbook]# ansible-playbook aws-provision-demo1.yaml --syntax -vvv
```
Corro el stack demo2.
```
[root@rosas playbook]# ansible-playbook -i ec2.py aws-provision-demo2.yaml

PLAY [Provision Stack Aws Demo 2] ********************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [distrostack : Replacing variables for cloudformation file] *************************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [distrostack : Remove ssh.cfg config] ***********************************************************************************************************************************************************************************************************************************
changed: [localhost]

TASK [distrostack : Run my CloudFormation stack] *****************************************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [distrostack : debug] ***************************************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "ec2_awsecs_stack.stack_outputs.URL": "VARIABLE IS NOT DEFINED!"
}

TASK [distrostack : debug] ***************************************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "ec2_awsecs_stack.stack_outputs.PublicIP": "18.217.137.32"
}

TASK [distrostack : set_fact] ************************************************************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [distrostack : debug] ***************************************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "ipaddress": "18.217.137.32"
}

TASK [distrostack : definition of 18.217.137.32] *****************************************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [distrostack : configure sshcfg file] ***********************************************************************************************************************************************************************************************************************************
changed: [localhost]

TASK [distrostack : Wait for server to restart] ******************************************************************************************************************************************************************************************************************************
ok: [localhost -> localhost]

TASK [distrostack : Refresh EC2 cache] ***************************************************************************************************************************************************************************************************************************************
changed: [localhost]

PLAY [Set up instances] ******************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************************************************************************************
ok: [18.217.137.32]

TASK [base : include] ********************************************************************************************************************************************************************************************************************************************************
included: /root/repos/test2/playbook/roles/base/tasks/base-yum.yml for 18.217.137.32

TASK [base : OS upgrade] *****************************************************************************************************************************************************************************************************************************************************
ok: [18.217.137.32]

TASK [base : Install list of packages] ***************************************************************************************************************************************************************************************************************************************
changed: [18.217.137.32] => (item=[u'docker', u'mc', u'git'])

TASK [base : Install docker-compose] *****************************************************************************************************************************************************************************************************************************************
 [WARNING]: Consider using get_url or uri module rather than running curl

changed: [18.217.137.32]

TASK [base : Set permission to docker-compose] *******************************************************************************************************************************************************************************************************************************
 [WARNING]: Consider using file module with mode rather than running chmod

changed: [18.217.137.32]

TASK [base : Update ec2-user to use docker] **********************************************************************************************************************************************************************************************************************************
changed: [18.217.137.32]

TASK [base : Passwordless sudo] **********************************************************************************************************************************************************************************************************************************************
 [WARNING]: when statements should not include jinja2 templating delimiters such as {{ }} or {% %}. Found: {{passwordless_sudo}}

changed: [18.217.137.32]

TASK [base : Make sure docker is running] ************************************************************************************************************************************************************************************************************************************
changed: [18.217.137.32]

PLAY [Set up instances] ******************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************************************************************************************
ok: [18.217.137.32]

TASK [dockercompose : Create wordpress compose directory] ********************************************************************************************************************************************************************************************************************
 [WARNING]: when statements should not include jinja2 templating delimiters such as {{ }} or {% %}. Found: {{ wordpressdbdeploy }}

 [WARNING]: Consider using file module with state=directory rather than running mkdir

changed: [18.217.137.32]

TASK [dockercompose : Create wordpress compose file] *************************************************************************************************************************************************************************************************************************
 [WARNING]: when statements should not include jinja2 templating delimiters such as {{ }} or {% %}. Found: {{ wordpressdbdeploy }}

changed: [18.217.137.32]

TASK [dockercompose : Run Docker-compose] ************************************************************************************************************************************************************************************************************************************
 [WARNING]: when statements should not include jinja2 templating delimiters such as {{ }} or {% %}. Found: {{ wordpressdbdeploy }}

 [WARNING]: Consider using 'become', 'become_method', and 'become_user' rather than running sudo

changed: [18.217.137.32]

PLAY RECAP *******************************************************************************************************************************************************************************************************************************************************************
18.217.137.32              : ok=13   changed=9    unreachable=0    failed=0
localhost                  : ok=12   changed=3    unreachable=0    failed=0

[root@rosas playbook]# ssh -i keys/private/RINO_OHIO.pem ec2-user@18.217.137.32 docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                NAMES
dcd84c90489d        mysql:5.7           "docker-entrypoint..."   About a minute ago   Up About a minute   3306/tcp             wordpress_mysql_1
ef23b1d8f122        wordpress           "docker-entrypoint..."   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   wordpress_wordpress_1
[root@rosas playbook]#

```
Luego voy al browser y tipeo http://IPAWS y me abre el wordpress.

Luego borro el stack al finalizar.

```
[root@rosas playbook]# ansible-playbook remove-provision-demo2.yml
 [WARNING]: Could not match supplied host pattern, ignoring: all

 [WARNING]: provided hosts list is empty, only localhost is available


PLAY [Provision Stack] *******************************************************************************************************************************************************************************************************************************************************

TASK [tear down old demopoc deployment] **************************************************************************************************************************************************************************************************************************************
changed: [localhost]

TASK [file] ******************************************************************************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [definition of ipaddress] ***********************************************************************************************************************************************************************************************************************************************
changed: [localhost]

PLAY RECAP *******************************************************************************************************************************************************************************************************************************************************************
localhost                  : ok=3    changed=2    unreachable=0    failed=0

[root@rosas playbook]#

```
