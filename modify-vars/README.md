![logo](https://raw.github.com/1N0T/images/master/global/1N0T.png)
# Modificción variables.

Guardar información en una variable, para usarla posterirormente, es uno de lss necesidades más habituales en cualquier proceso de automatización.

Y no es menos frecuente necesitar modificar el valor de la misma, por uno nuevo que refleje la nueva situación.

Para variables elementales, el proceso es relativamente simple.

```yaml
---
  - hosts: localhost
  
    vars:
      variable: valor
    tasks:
      - name: Modificamos variable simple.
        set_fact:
          variable: valor modificado
  
      - name: Mostramos modificación.
        debug: var=variable

```
Si ejecutamos el **playbook** anterior, vemos como la variable **variable** ha cambiado su valor de **valor** a **valor modificado** gracias a **set_fact**.
```json
ok: [localhost] => {
    "variable": "valor modificado"
}
```
Para estructuras más complejas, cabria esperar que se pudiera utilizar algo parecido a:

```yaml
---
- hosts: localhost

  vars:
    autor:
      nombre: mi nombre
      apellido: mi apellido
      telefono: 
        prefijo: 00
        numero: 000 00 00

  tasks:
    - name: Modificamos teléfono del autor.
      set_fact:
        autor.telefono.prefijo: 93
```
Pero si lo intentamos ejecutar, nos encontraremos un desagradable error. Lamentablemente, el procedimiento para realizar tal modificación, es bastante menos intuitiva y, tiene un aspecto similar al siguiente:

``` yaml
---
- hosts: localhost

  vars:
    autor:
      nombre: mi nombre
      apellido: mi apellido
      telefono: 
        prefijo: 00
        numero: 000 00 00

  tasks:
    - name: Modificamos teléfono del autor.
      set_fact:
        autor: "{{ autor|combine({'telefono': {'prefijo': 99, 'numero': '888 88 88', 'extension': 777}}, recursive=True) }}"

    - name: Mostramos modificación.
      debug: var=autor
```

El resultado obtenido sería algo parecido a lo que se muestra a continuación.

```json
ok: [localhost] => {
    "autor": {
        "apellido": "mi apellido",
        "nombre": "mi nombre",
        "telefono": {
            "extension": 777,
            "numero": "888 88 88",
            "prefijo": 99
        }
    }
}
```
