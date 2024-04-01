# Proyecto de Despliegue con Ansible en AWS

Este proyecto automatiza el despliegue de una aplicación de 3 capas en AWS utilizando Ansible.

## Infraestructura

**La infraestructura en AWS se compone de los siguientes elementos:**
- VPC en us-east-1 con 2 subnets públicas y 2 subnets privadas
- Internet Gateway y NAT Gateways para conectividad
- 3 instancias EC2 con Amazon Linux 2 (1 Bastion Host en subnet pública, 2 en subnets privadas)
- Application Load Balancer en subnets públicas, escuchando en puerto 80
- EFS (Elastic File System) en subnets privadas
- RDS (MySQL) en subnets privadas

## Configuración de Seguridad
**Los Security Groups están configurados de la siguiente manera:**
- Bastion Host: Permite acceso SSH sólo desde la IP del administrador
- Instancias Privadas: Permiten puerto 5000 desde el ALB y SSH desde el Bastion Host
- EFS: Permite acceso por puerto 2049 desde las instancias privadas
- RDS: Permite acceso por puerto 3306 desde las instancias privadas
- ALB: Permite acceso por puerto 80 desde cualquier origen

## Despliegue con Ansible
**El despliegue se realiza con Ansible, separando el playbook en los siguientes roles:**
1. *server-setup*: Configuración inicial de servidores
2. *efs*: Montaje de EFS
3. *nginx*: Instalación de Nginx
4. *git*: Clonación del repositorio
5. *rds*: Conexión a RDS y creación de BD/usuario
5. *app*: Instalación de dependencias y ejecución de la aplicación

Para la creación de roles ejecutar por cada uno de ellos
```yml
ansible-galaxy init server-setup --init-path ./roles
```
## Estructura del Proyecto
```yml
ansible_project/
├── ansible.cfg
├── inventories/
│   └── production/
│       ├── group_vars/
│       │   ├── all/
│       │   │   └── config.yml
│       │   ├── all.yml
│       │   └── EC2_1/
│       │       ├── secret.vault
│       │       └── vars.yml
│       └── inventory_aws_ec2.yml
├── playbooks/
│   └── playbook.yml
├── requirements.yml
└── roles/
    ├── app/
    ├── efs/
    ├── git/
    ├── nginx/
    ├── rds/
    └── server-setup/
```

### Uso

**Para ejecutar el despliegue de la aplicación con Ansible, sigue estos pasos:**


1. **Configurar credenciales de AWS:**

    - Asegúrate de tener configuradas las credenciales de AWS en tu entorno local. Puedes usar el CLI de AWS y ejecutar aws configure para configurar tu Access Key ID y Secret Access Key.

2. **Configurar el inventario dinámico:**

    - Revisa el archivo inventories/production/inventory_aws_ec2.yml y asegúrate de que los filtros coincidan con las instancias EC2 que deseas incluir en el despliegue.
    - Verifica que los grupos de hosts estén correctamente configurados según las etiquetas (tags) de tus instancias.

3. **Definir variables:**

    - Revisa y ajusta las variables definidas en inventories/production/group_vars/all/config.yml, como el usuario SSH, la clave privada y los argumentos de SSH.
    - Configura las variables específicas de la aplicación en `inventories/production/group_vars/EC2_1/vars.yml`, como la URL del repositorio, la rama a desplegar y la ruta de destino.
    - Asegúrate de que las variables de conexión a RDS y las credenciales estén correctamente definidas en `roles/rds/vars/main.yml`.

4. **Personalizar los roles:**

    - Si necesitas realizar cambios en la configuración de los servidores, el montaje de EFS, la instalación de Nginx, la clonación del repositorio, la conexión a RDS o la ejecución de la aplicación, revisa y modifica los roles correspondientes en el directorio roles/.

5. **Ejecutar el playbook:**

    - Desde el directorio raíz del proyecto, ejecuta el siguiente comando para iniciar el despliegue:
```ansible-playbook playbooks/playbook.yml```
    - Ansible se conectará a las instancias EC2 especificadas en el inventario y ejecutará las tareas definidas en el playbook y los roles.

6. **Verificar el despliegue:**

    - Una vez que el playbook se haya ejecutado correctamente, verifica que la aplicación esté accesible a través del Application Load Balancer.
    - Accede a la URL del ALB en tu navegador y comprueba que la aplicación se esté ejecutando correctamente.

7. **Solución de problemas:**

    - Si encuentras algún problema durante el despliegue, revisa los mensajes de error en la salida de Ansible.
    - Verifica que las variables estén correctamente definidas y que los roles estén configurados según tus necesidades.
    - Si es necesario, conecta a las instancias EC2 a través de SSH para realizar una inspección más detallada.

Recuerda que antes de ejecutar el despliegue en un entorno de producción, es recomendable probar el *playbook* en un entorno de prueba o *staging* para asegurarte de que todo funcione correctamente.


## Contribución

Las contribuciones son bienvenidas. Por favor abrir un issue primero para discutir cambios mayores.

## License

[MIT](https://choosealicense.com/licenses/mit/) 