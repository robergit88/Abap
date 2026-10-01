# Servicio OData V4 en BTP CLOUD sin SEGW

## Explicacion:

CRUD básico sobre una sola tabla Z, el equivalente a lo que se puede hacer con SEGW. 


|Fase	| Objetivo|
|:---:|-----------|
|0	| Tabla Z y datos de prueba |
|1	| GET de la colección y GET por clave |
|2	| POST |
|3	| PATCH (y PUT para comparar) |
|4	| DELETE |


## Fase 0: Tabla y datos de prueba

``` js
@EndUserText.label : 'Clientes - ejercicio RAP'
@AbapCatalog.enhancement.category : #NOT_EXTENSIBLE
@AbapCatalog.tableCategory : #TRANSPARENT
@AbapCatalog.deliveryClass : #A
@AbapCatalog.dataMaintenance : #RESTRICTED
define table zcliente_01 {

  key client     : abap.clnt not null;
  key cliente_id : abap.char(10) not null;
  nombre         : abap.char(40);
  ciudad         : abap.char(30);
  email          : abap.char(60);
  estado         : abap.char(1);
  ultima_modif   : timestampl;

}
```

Clase para cargar datos ficticios

[Cargar tabla Z](./otros/zcl_carga_cliente.md)

