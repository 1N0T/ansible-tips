![logo](https://raw.github.com/1N0T/images/master/global/1N0T.png)
# Ejecutar un conjunto de tareas para una lista de elementos.

Resulta fácil realizar una tarea para cada uno de los elemntos de una lista. Un ejemplo muy básico sería el siguiente:

``` yaml
---
- hosts: localhost
  tasks:
    - name: Tarea 
      debug:
        msg: "Tarea {{ item }} ejecutada"
      with_items:
        - principal
        - secundaria  
```

El resultado obtenido sería algo parecido al siguiente:
```
ok: [localhost] => (item=None) => {
    "msg": "Tarea principal ejecutada"
}
ok: [localhost] => (item=None) => {
    "msg": "Tarea secundaria ejecutada"
}
```

El problema aparece cuando queremos ejecutar varias tareas consecutivas para cada uno de los elementos de la lista (realizar todas las tareas antes de pasar al siguiente elemento). de momento, no podemos añadir **with_items** a los bloques de tareas.

La solución pasa por crear un fichero con todas las tareas que queremos ejecutar para cada elemento, y después incluirlo en el playbook principal, como se muestra a continuación.

Creamos un fichero para las tareas, llamado **sub-tasks.yml** con un contenido similar al siguiente:
``` yaml
- name: Tarea 1
  debug:
    msg: "Tarea 1 {{ item }} ejecutada"

- name: Tarea 2
  debug:
    msg: "Tarea 2 {{ item }} ejecutada"
```

Poaterirorment lo incluimos en el playbook principal, como se muestra a continuación:
``` yaml
---
- hosts: localhost
  tasks:
    - include: sub-tasks.yml 
      with_items:
        - principal
        - secundaria  
```
El resultado, tras ejecutar ```ansible-playbook block-with-iteml.yml``` será el deseado, como podemos observar.
```
TASK [Tarea 1] **************************
ok: [localhost] => {
    "msg": "Tarea 1 principal ejecutada"
}

TASK [Tarea 2] **************************
ok: [localhost] => {
    "msg": "Tarea 2 principal ejecutada"
}

TASK [Tarea 1] **************************
ok: [localhost] => {
    "msg": "Tarea 1 secundaria ejecutada"
}

TASK [Tarea 2] **************************
ok: [localhost] => {
    "msg": "Tarea 2 secundaria ejecutada"
}

```
