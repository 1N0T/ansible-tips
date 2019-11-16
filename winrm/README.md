![logo](https://raw.github.com/1N0T/images/master/global/1N0T.png)
# Gestionar equipos windows usando WINRM.

Existe abundante información para realizar todo tipo de tareas sobre servidores **Linux**, y aunque es frecuente encontrar reerencias a que es posible realizar lo propio sobre **Winodws**, lo habitual es que no se expliquen los detalles. Parece que, finalmente, el mecanismo de comunicación utilizado por anible, para comunicar con los equipos será **WinRM** (Windows Remote Management es) que permite la administración remota de equipos, utilizando peticiones **HTTP/S**.

## WinRM. 
El primer escollo a solucionar se que, por defecto, esta funcionalidad no viene habilitada en Windows, así que el primer requisito a cumplir, es que esté disponible en los equipos windows que queramos administrar. Afortunadamente, ya hay quien se ha preocupado de crear un script **PowerShell** que se encarga de todo el proceso. He dejado una copia de uno que a mi me ha funcionado [ConfigureRemotingForAnsible.ps1](ConfigureRemotingForAnsible.ps1).

Debemos copiarlo en el equipo windows, y ejecutar (como administrador) algo parecido a lo siguiente:
```bat
Powershell.exe -ExecutionPolicy Unrestricted -File c:\utils\WinRM\ConfigureRemotingForAnsible.ps1
```
## Paquetes adicionales en servidor de ansible.
Además de ansible, se requiere instalar los siguientes paquetes, con la sentencia que corresponda, dependiendo de la distribución que tengamos.
```bash
sudo apt install python-dev libkrb5-dev krb5-user
```
```bash 
yum install python-devel krb5-devel krb5-libs krb5-workstation
```
Y adicionalmente, se tiene que instalar los siguientes módulos de python.
```bash
pip install pywinrm
pip install requests-ntlm
pip install requests-kerberos
pip install requests-credssp
pip install pypsexec
pip install pywinrm[kerberos]
```
## Configuración de kerberos.
Vamos a imaginar que se utilizará kerberos para autentificar los usuarios del dominio, por lo que debemos realizar la configuración con la información del domino. Para ello, vamos a suponer que nuestro dominio es **dominio.net** y que **dc01.dominio.net** y **dc02.dominio.net** son los controladores de dominio.

Debemos editar el fichero **/etc/krb5.conf** para añadir el siguiente contenido:
```
[realms]
  DOMINIO.NET = {
    kdc = dc01.dominio.net
    kdc = dc02.dominio.net
  }

  ...

  [domain_realm]
    .dominio.net = DOMINIO.NET

  ...        

```
Para comprobar que la configuración es correcta, podemos ejecutar el siguiente comando.
```bash
kinit miusuario@DOMINIO.NET
```
Sólo remarcar que, el nombre del dominio se tiene que especificar en mayúsculas.

Podemos validar que WinRM está activado de la siguiente forma.
```bash
curl --insecure https://miEquipoWindows:5986
```

```
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN""http://www.w3.org/TR/html4/strict.dtd">
<HTML><HEAD><TITLE>Not Found</TITLE>
<META HTTP-EQUIV="Content-Type" Content="text/html; charset=us-ascii"></HEAD>
<BODY><h2>Not Found</h2>
<hr><p>HTTP Error 404. The requested resource is not found.</p>
</BODY></HTML>
```
Si no estuviera activo devolvería
```
curl: (7) Failed to connect to miEquipoWindows port 5986: Conexión rehusada
```
## A tener en cuenta en ansible.
En el inventario, si se utiliza kerberos, debe figurar el nombre del equipo. Si especificamos la IP, nos dará un error de autenticación, indicando kerberos que no encuentra el equipo.

Se tienen que especificar, en algún lugar, las siguientes variables:
 - **ansible_ssh_user:** "ususario@DOMINIO.NET"
 - **ansible_ssh_pass:** "contraseña" 
   - Se puede solicitar con el parámetro **-k** al ejecutar el play-book pero, si se ha definido en el mismo, prevalece el valor del playbook sobre el solicitado por el parámetro **-k**.
 - **ansible_ssh_port:** 5986
 - **ansible_connection:** winrm
 - **ansible_winrm_server_cert_validation:** ignore

Ya podemos escribir un play-book similar al siguiente:
```yaml
---
- hosts: winserver
  vars:
    - ansible_ssh_user: "usuario@DOMINIO.NET"
    - ansible_ssh_port: 5986
    - ansible_connection: winrm
    - ansible_winrm_server_cert_validation: ignore

  tasks:
    - name: Test uso WinRM
      win_command: ipconfig
```
Que podremos ejecutar de la siguiente forma:
```bash
ansible-playbook winrm.yml -k
