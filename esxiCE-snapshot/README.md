![logo](https://raw.github.com/1N0T/images/master/global/1N0T.png)
# Realizar un snapshot en ESXi Community Edition..

En las versiones actuales de ansible, para realizar un snapshot de una VM vmware, se debería utilizar el módulo **vmware_guest_snapshot** que requiere la instalación de ```pip install PyVmomi``` y que, al parecer, hace uso del API expuesta por vmware. El problema es que, en el caso de la versión Community Edition (la gratuita), la API sólo está expuesta en modo lectura, lo que no permite realizar ninguna modificación de las VM. Así que, el código siguiente, funcionaría en un ESXi licenciado, pero no en la versión Community Edition.

``` yaml
---
- hosts: localhost
  tasks:
    - name: Creaamos snapshot
      vmware_guest_snapshot:
        hostname: "{{ mi_vmware_server }}"
        username: "{{ mi_administrador_vmware }}"
        password: "{{ mi_password_administrador }}"
        datacenter: "{{ mi_datacenter }}"
        folder: "/"
        name: "{{ mi_vm }}"
        state: present
        snapshot_name: preAnsibleAcctions
        description: Antes ejecución playbook
      delegate_to: localhost

```
Una alternativa sería utilizar el módulo anterior, **vsphere_guest**, que requiere la instalación de ```pip install pysphere``` y que no utiliza el API de vmware, aunque hay un par de pegas. La primera es que este módulo desaparecerá en la versión 2.9 de ansible. La otra, es que no tiene disponible la funcionalidad de snapshot. Existe alguna propuesta de implementación en GitHub, pero que no ha sido incorporada al módulo oficial ya que está deprecated.

Así que, para a realizar la tarea, vamos a crear un pequeño script en python, que tendría el siguiente aspecto.

``` yaml
# Requiere de la instalación del siguiente módulo
#   pip install pysphere
#
# Podemos encontrar información sobre el módulo pyton en:
# https://github.com/itnihao/pysphere/wiki/Quick-guide-to-start-using-PySphere
#
#
---
- hosts: localhost

  vars:
    ESXi:
      host: 10.1.1.18
      vm_name: MI-VM
      admin_user: root
      admin_password: Contraseña

    script_content: |
        #!/usr/bin/env python

        from pysphere import VIServer 
        import ssl

        # Permitimos el uso de certificados autofirmados.
        ssl._create_default_https_context = ssl._create_unverified_context

        server = VIServer()
        server.connect("{{ ESXi.host }}", "{{ ESXi.admin_user }}", "{{ mi_password }}")
        vm = server.get_vm_by_name("{{ ESXi.vm_name }}")

        # La creación del snapshot es síncrona, por lo que no continuará hasta que finalice.
        # Si quisiéramos que fuera aiíncrona, tendráimos que hacer algo así.
        #   task = vm.create_snapshot("mysnapshot", sync_run=False)
        vm.create_snapshot("Test snapshot", description="Created by pysphere", memory=False, quiesce=False)
        server.disconnect() 
        quit(0)
    
    vars_prompt:
      - name: "mi_password"
        prompt: "Introduzca la contraseña de {{ ESXi.admin_user }} ESXi: "
        private: yes
        default: contraseña

    tasks:

      # En teoría, se podría pasar inline el código del script con:
      #    shell: |
      #      bla, bla,
      #      bla ...
      #
      # Pero si lo hacemos de esta forma, no se respeta la identación (cosa que resulta crítica en el caso de pytho).
      # La solución ha sido crear una variable y pasarla al módulo shell.
      - name: Creación snapshot
        shell: "{{ script_content }}"
        args:
          executable: /usr/bin/python
        delegate_to: localhost

      - name: Advertencia
        debug:
          msg: No te olvides de consolidar el snapshot creado tras revisar que todo está OK
```




