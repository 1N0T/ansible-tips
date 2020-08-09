![logo](https://raw.github.com/1N0T/images/master/global/1N0T.png)
# Fechas y duraciones.

Imagina que tienes un playbook que realiza toda una serie de tareas sobre las cuales tienes que dejar, lo que los auditorea llaman **evidencia documental**. Evidentemente, lo que procede es que utilices un template **jinja2** para generar el fichero del informe.

Una de las cosas que seguro debe aparecer, es la fecha en la que se ha ejecutado el proceso. Para ello, lo más simple es utilizar el contenido de la variable **ansible_date_time**.

Para ver el detalle, podemos ejecutar el siguiente **playbook**.

``` yaml
---
- hosts: localhost
  tasks:
  - name: Mostramos fecha de ejcución
    debug:
      msg: "{{ ansible_date_time }}"
```

El resultado obtenido sería algo parecido a lo que se muestra a continuación. Como podemos comprobar, disponemos de todos los detalles que podamos necesitar para nuesstro informe generado con **jinja2**.
```json
ok: [localhost] => {
    "msg": {
        "date": "2020-08-08",
        "day": "08",
        "epoch": "1596915648",
        "hour": "21",
        "iso8601": "2020-08-08T19:40:48Z",
        "iso8601_basic": "20200808T214048338811",
        "iso8601_basic_short": "20200808T214048",
        "iso8601_micro": "2020-08-08T19:40:48.338929Z",
        "minute": "40",
        "month": "08",
        "second": "48",
        "time": "21:40:48",
        "tz": "CEST",
        "tz_offset": "+0200",
        "weekday": "sábado",
        "weekday_number": "6",
        "weeknumber": "31",
        "year": "2020"
    }
}
```
Otra al ternativa, sería utilzar el comando **date** para obtener la fecha en el formato que nos interese, como se muestra en el siguiente playbook.
```yaml
---
- hosts: localhost
  tasks:
  - name: Utilizamos bash para obtener la fecha en el formato que nos interese.
    set_fact: 
      fecha: "{{lookup('pipe','date \"+%Y-%m-%d %H:%M:%S\"')}}"    

  - name: Mostramos fecha
    debug:
      msg: "{{ fecha }}"

```
Con el que obtendríamos algo parecido a lo siguiente:
```json
ok: [localhost] => {
    "msg": "2020-08-09 19:29:14"
}

```

Sin duda, otra cosa que podemos necesitar, es conocer el tiempo que ha tardado en ejecutarse un determinado número de tareas. Para ello, el procedimiento es simple, pero requiere algo de preparación. Tenemos que guardar la fecha y hora en una variable, justo antes de empezar lo que nos interesa medir, lo mismo justo después, y calcular la diferencia a posteriori.

```yaml
---
- hosts: localhost
  tasks:
  - name: Guardamos segundos época al inicio
    set_fact: 
      inicio: "{{lookup('pipe','date -d now +%s')}}"

  - name: Realizamos nuestro árduo trabajo
    pause:
      seconds: 8

  - name: Guardamos segundos época al acabar
    set_fact: 
      fin: "{{lookup('pipe','date -d now +%s')}}"

  - name: Calculamos diferencia en segundos, entre inicio y fin.
    set_fact: 
      dif: "{{ (fin|int - inicio|int) }}"
  
  - name: Realizamos cálculo de días, horas, minutos y segundos.
    shell: |
      printf '%d días, %02d horas, %02d minutos y %02d segundos' \
              $(({{ dif }}/86400))                               \ 
              $(({{ dif }}%86400/3600))                          \
              $(({{ dif }}%3600/60))                             \
              $(({{ dif }}%60))                                  \
    register: diferencia

  - name: Mostramos duración
    debug:
      msg: "{{ diferencia.stdout }}"

```

Que nos proporcionará un resultado parecido al siguiente:

```json
ok: [localhost] => {
    "msg": "0 días, 00 horas, 00 minutos y 08 segundos"
}
```
Aunque podrímos haber intentado realizar todos los cálculos con ansible, pero me ha parecido que resultaba más fácil utilizar **bash**.