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
##Para ver las salidas de los comandos
Ver el archivo removedemo.out y createdemo1.out


#INSTALAR DESDE CERO EN UNA DISTRO BASADA EN FEDORA
Instalo ANSIBLE
```
[root@rosas ~]# dnf install ansible
Last metadata expiration check: 2:17:13 ago on Sat 03 Feb 2018 08:00:39 PM EST.
Dependencies resolved.
```

Me clone le repo.

```


```
