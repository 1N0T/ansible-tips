![logo](https://raw.github.com/1N0T/images/master/global/1N0T.png)
# Encriptar el valor de una variable.

Antes o después, acabaremos poniendo en nuestro playbook (de forma provisional, naturalmente) una contraseña de usuario requerida para alguna instalación. Pero llega una edad en que la memoria flaquea y, con la complicidad de la larga lista de TODOs, la provisionalidad deja de ser tal (voluntaria o involuntariamente) y el valor en claro se perpetua en el playbook definitivo (o en alguno de nuestros commits intermedios).

Para que esto no ocurra, nada mejor que colocar desde el inicio el valor encriptado, después de todo tampoco cuesta tanto.

Imaginemos que quremos definir el equvalente a:

```yaml
---
  - hosts: localhost
  
    vars:
      mi_password_secreto: "NoLoConoceNadie"
```
Deberemos ejecutar el siguiente comando:

```bash
echo -n "NoLoConoceNadie" | ansible-vault encrypt_string --stdin-name "mi_password_secreto"
```
Después de introducir la contraseña para la encriptación que se nos solicitará, obtendremos una salida parecida a la siguiente:
```
New Vault password: 
Confirm New Vault password: 
Reading plaintext input from stdin. (ctrl-d to end input)
mi_password_secreto: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          62626464613161396533623034613237363839353836353334333739373463326437313163353263
          3661346139613838336336336232323830326237323364370a316464363563653039353061303338
          63613833303461356265623232376563373839343432643262326237343665313530346262336230
          3332616365383161340a353564623333303735623131326539343435626638326333376639633337
          3262
Encryption successful
```

Sólo hay que tener en cuenta un par de consideraciones:
* No olvidar el parámetro **-n** del comando **echo**, para que no nos añada un salto de línea **\n** al final del string, cosa que con toda probabilidad no deseamos.
* Si vamos a crear más de una variable, debemos utilizar la misma contraseña para encriptar todos los valores del mismo playbook.

Ya sólo nos queda copiar el resultado obtenido para crear algo parecido a:

```yaml
---
- hosts: localhost

  vars:
    mi_password_secreto: !vault |
        $ANSIBLE_VAULT;1.1;AES256
        62626464613161396533623034613237363839353836353334333739373463326437313163353263
        3661346139613838336336336232323830326237323364370a316464363563653039353061303338
        63613833303461356265623232376563373839343432643262326237343665313530346262336230
        3332616365383161340a353564623333303735623131326539343435626638326333376639633337
        3262
    desencriptada: "Mi contraseña secreta es: [{{ mi_password_secreto }}]"

  tasks:
    - name: Mostramos valor desencriptado
      debug: var=desencriptada
```

Para ejecutar nuestro **playbook**, deberemos hacer lo que se muestra a continuación:

```bash
ansible-playbook playbook.yml --ask-vault-pass
```

Tras introducir la contraseña que nos solicita, que es la utilizada para la encriptación, obtendremos la siguiente salida:
```
Vault password: 

PLAY [localhost] **********************************************************

TASK [Gathering Facts] ****************************************************
ok: [localhost]

TASK [Mostramos valor desencriptado] **************************************
ok: [localhost] => {
    "desencriptada": "Mi contraseña secreta es: [NoLoConoceNadie]"
}

PLAY RECAP ****************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0   
```

Como se puede comprobar, una vez proporcionada la contraseña de encriptación, la variable se comporta internamente como si se hubiera especificado el valor en claro.

Aquellos que no conozcan la contraseña de encriptación, obtendrán un resultado como este.
```
Vault password: 

PLAY [localhost] **********************************************************************

TASK [Gathering Facts] ****************************************************************
ok: [localhost]

TASK [Mostramos valor desencriptado] **************************************************
fatal: [localhost]: FAILED! => {"msg": "An unhandled exception occurred while templating 'Mi contraseña secreta es: [{{ mi_password_secreto }}]'. Error was a <class 'ansible.parsing.vault.AnsibleVaultError'>, original message: Decryption failed (no vault secrets were found that could decrypt)"}
	to retry, use: --limit @/home/tsolis/sources/ansible/ansible-vault/playbook.retry

PLAY RECAP ****************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=1   
```
