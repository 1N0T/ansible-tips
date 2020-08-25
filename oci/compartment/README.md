![logo](https://raw.github.com/1N0T/images/master/global/1N0T.png)
# Creación y borrado de compartimentos.
Para la creación de un nuevo **compartment**, necesitaremos conocer previamente el **OCID** del compartimento padre (el root en nuestor caso).

Una cosa a tener en cuenta, es que el módulo no funciona correctamente si asignamos el contenido de una variable, por lo que, para solucionar el problema, tenemos que interpolar un string al estilo **"{{ variable }}"**.

A continuación, un **playbook** de ejemplo que crea un compartimento y, acto seguido lo borra. Para ello, necesitaremos guardar el **OCID** del compartimento recién creado, para poder borrarlo posteriormente.

Si hemos modificado la ubicación por defecto, del fichero de configuración de nuestra conexión **oci**, tendremos que especificar un valor para **config_file_location**, indicando el nombre y ruta de acceso al fichero. Si hemos mantenido los valores por defecto, podremos omitir dicho parámetro.

```yaml
---
- hosts: localhost
  vars:
    compartimento:
      # OCID del compartment root
      OCID_root: "ocid1.tenancy.oc1..aaaaaaaaaaaaaaa...aaaaaaaaaa..." 
      nombre: "compartimento-ansible"
      descripcion: "Compartimento creado por ansible"

  tasks:
    - name: "Creación de un compartimento {{ compartimento.nombre }} dentro del raiz"
      oci_compartment:
        config_file_location: ./configuracion/config
        parent_compartment_id: "{{ compartimento.OCID_root }}"
        name: "{{ compartimento.nombre }}"
        description: "{{ compartimento.descripcion }}"
        state: present
      register: resultado

    - name: "Resultado creación {{ compartimento.nombre }}"
      debug:
        var: resultado

    - name: "Borramos compartimento {{ compartimento.nombre }}"
      oci_compartment:
        config_file_location: ./configuracion/config
        id: "{{ resultado.compartment.id }}" 
        state: absent
      register: resultado_borrado

    - name: "Resultado borrado {{ compartimento.nombre }}"
      debug:
        var: resultado_borrado
```
Después de la ejecución, obtendremos una salida similar a la que mostramos a continuación.
```
PLAY [localhost] *********************************************************************************************

TASK [Gathering Facts] **********************************************************************
ok: [localhost]

TASK [Creación de un compartimento compartimento-ansible dentro del raiz] **********************************************************************
changed: [localhost]

TASK [Resultado creación compartimento-ansible] *************************************************************************
ok: [localhost] => {
    "resultado": {
        "changed": true,
        "compartment": {
            "compartment_id": "ocid1.tenancy.oc1..aaaaaaaaaaaaaaa...aaaaaaaaaa...",
            "defined_tags": {},
            "description": "Compartimento creado por ansible",
            "freeform_tags": {},
            "id": "ocid1.compartment.oc1..bbbbbbb...bbbbbbbb...",
            "inactive_status": null,
            "is_accessible": true,
            "lifecycle_state": "ACTIVE",
            "name": "compartimento-ansible",
            "time_created": "2020-08-25T14:43:01.122000+00:00"
        },
        "failed": false
    }
}

TASK [Borramos compartimento compartimento-ansible] **********************************************************************
changed: [localhost]

TASK [Resultado borrado compartimento-ansible] **********************************************************************
ok: [localhost] => {
    "resultado_borrado": {
        "changed": true,
        "compartment": {
            "compartment_id": "ocid1.tenancy.oc1..aaaaaaaaaaaaaaa...aaaaaaaaaa...",
            "defined_tags": {},
            "description": "Compartimento creado por ansible",
            "freeform_tags": {},
            "id": "ocid1.compartment.oc1..bbbbbbb...bbbbbbbb...",
            "inactive_status": null,
            "is_accessible": true,
            "lifecycle_state": "DELETED",
            "name": "compartimento-ansible.21Ll60Xg",
            "time_created": "2020-08-25T14:43:01.122000+00:00"
        },
        "failed": false,
        "work_request": {
            "compartment_id": "ocid1.tenancy.oc1..aaaaaaaaaaaaaaa...aaaaaaaaaa...",
            "errors": null,
            "id": "ocid1.compartment.oc1..bbbbbbb...bbbbbbbb...",
            "logs": null,
            "operation_type": "DELETE_COMPARTMENT",
            "percent_complete": 100.0,
            "resources": null,
            "status": "SUCCEEDED",
            "time_accepted": "2020-08-25T14:43:19.835000+00:00",
            "time_finished": "2020-08-25T14:45:05.512000+00:00",
            "time_started": "2020-08-25T14:43:29.612000+00:00"
        }
    }
}

PLAY RECAP ***********************************************************
localhost                  : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   


```
