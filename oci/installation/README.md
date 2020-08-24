![logo](https://raw.github.com/1N0T/images/master/global/1N0T.png)
# Instalación OCI (Oracle Cloud Infrastructure).
Aunque al tratarse de uno de los úlltimos en llegar, no están entre los proveedores de servicios cloud más populares, pero lo cierto es que durante el último año, mi impresión particular ha mejorado y ya parece una plataforma más madura.

Estamos aquí porque una de las opciones disponibles es la administración de lso componentes principales utilizando **ansible**. Para ello, se requiere instalar previamente el **cliente oci** y el **SDK de python**.

El procedimiento está perfectamente explicado en la propia documentación de **OCI** pero, como está principalmente orientado a servidores linux de la familia **redhat** y, me encontre alguna dificultad inicial al realizarla sobre una distribución de la familia **Debian** (Ubuntu 18.04 en concreto), por lo que procedo a detallar los pasos que he seguido para obtener un resultado satisfactorio.

Lo primero que haremos, para evitar interferencias con nuestra instalación actual, es crear un **entorno virtual de python**, para realizar toda la instalación dentro del mismo.
```bash
mkdir ansibleOCI
cd ansibleOCI/
mkdir configuracion
source venv/bin/activate
pip3 install oci 
pip3 install oci-cli 
```
A continusación, deberemos configurar el entorno para que disponga de todos los datos requeridos para la conexión con nuestro **tenant** de **oci**.

Antes de empezar, necesitamos recuperar cierta información que vamos a necesitar durante el proceso.

Naturalmente, debemos disponer de una **cuenta oci** con un **usuario** que disponga de los permisos necesarios para realizar las acciones que nos interesa. Con éste, nos logaremos a nuestro **tenant** y recuperaremos el **tenant OCID** (menú **Administration > Tenancy Details**).

También necesitaremos el **OCID del usuario** (menú **Identity > Users > User Details**).

Una vez recopilados estos datos, podemos proceder a configurar el entorno, ejecutando el siguiente comando e informando los datos solicitados.
```bash
oci setup config
```
```stdout
    This command provides a walkthrough of creating a valid CLI config file.

    The following links explain where to find the information required by this
    script:

    User API Signing Key, OCID and Tenancy OCID:

        https://docs.cloud.oracle.com/Content/API/Concepts/apisigningkey.htm#Other

    Region:

        https://docs.cloud.oracle.com/Content/General/Concepts/regions.htm

    General config documentation:

        https://docs.cloud.oracle.com/Content/API/Concepts/sdkconfig.htm


Enter a location for your config [/home/miusuario/.oci/config]: /home/miusuario/ansibleOCI/configuracion/config
Enter a user OCID: ocid1.user.oc1..aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
Enter a tenancy OCID: ocid1.tenancy.oc1..aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
Enter a region (e.g. ap-chuncheon-1, ap-hyderabad-1, ap-melbourne-1, ap-mumbai-1, ap-osaka-1, ap-seoul-1, ap-sydney-1, ap-tokyo-1, ca-montreal-1, ca-toronto-1, eu-amsterdam-1, eu-frankfurt-1, eu-zurich-1, me-jeddah-1, sa-saopaulo-1, uk-gov-cardiff-1, uk-gov-london-1, uk-london-1, us-ashburn-1, us-gov-ashburn-1, us-gov-chicago-1, us-gov-phoenix-1, us-langley-1, us-luke-1, us-phoenix-1, us-sanjose-1): eu-frankfurt-1
Do you want to generate a new API Signing RSA key pair? (If you decline you will be asked to supply the path to an existing key.) [Y/n]: y
Enter a directory for your keys to be created [/home/miusuario/.oci]: /home/miusuario/ansibleOCI/configuracion
Enter a name for your key [oci_api_key]: 
Public key written to: /home/miusuario/ansibleOCI/configuracion/oci_api_key_public.pem
Enter a passphrase for your private key (empty for no passphrase): 
Private key written to: /home/miusuario/ansibleOCI/configuracion/oci_api_key.pem
Fingerprint: 94:4a:85:a6:57:a9:a5:80:76:9a:04:7a:0a:6a:48:59
Config written to /home/miusuario/ansibleOCI/configuracion/config


    If you haven't already uploaded your API Signing public key through the
    console, follow the instructions on the page linked below in the section
    'How to upload the public key':

        https://docs.cloud.oracle.com/Content/API/Concepts/apisigningkey.htm#How2



```
Los pasos anterirores, generarán un par de claves (pública y privada) y, un fichero de configuración de la conexión, en el directorio que hayamos especificado. Si hemos dejado las ubicaciones por defecto, ambos se situarán dentro de **$HOME/.oci/** que es donde lo esperan encontrar, tando el **Python SDK** como los **módulos ansible**. Pero para completar el ejemplo, he modificado la ubicación por defecto.

Antes de poder probar la conexión, tenemos que **subir la clave pública** al perfil del usuario (menú **Identity > Users > User Details > API Keys**).

Si hemos mantenido el destino por defecto, podríamos probar la configuración de la conexión con el siguiente comando.
```bash
oci os ns get
```
Pero, como hemos decidido complicarnos un poco la vida personalizando el destino, deberemos asumir las consecuencias de nuestra decición y ejecutar el comando de la siguiente forma.
```bash
oci os ns get --config-file /home/miusuario/ansibleOCI/configuracion/config
```
Una forma de solventar en parte esta incomodidad, es definir una variable de entorno **OCI_CONFIG_FILE**
```bash
export OCI_CONFIG_FILE="/home/miusuario/ansibleOCI/configuracion/config"

```
En caso de no definir la variable de entorno, posteriormente, nos veremos obligados a informar el parámetro **config_file_location** siempre que utilicemos los **módulos ansible para oci**.

Sea como fuere, si todo ha ido bien, obtendremos una respuesta parecida a la siguiente:
```json
{
  "data": "frdjxxxxxjgwk"
}
```
Una vez confirmado que podemos conectar a nuestra infraestructura cloud, procederemos a instalar **ansible**, y los **módulos oci**, dentro de nuestro entorno virtual.
```bash
pip3 install ansible
git clone https://github.com/oracle/oci-ansible-modules.git
cd oci-ansible-modules/
python3 ./install.py
```
A partir de este momento, ya podríamos empezar a utilizar **ansible** para gestionar nuestra **infraestructura oci**.


