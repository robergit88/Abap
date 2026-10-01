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

Se ejecuta Clase con F9 y se cargan datos.

![image](./img/TABLA_DATOS.png)


## Fase 1: GET de la colección y GET por clave

Aquí no hay código de negocio. Declaras el modelo y el framework hace el resto.

### Qué crear (en este orden)

#### 1. CDS view entity `ZI_CLIENTE`

Seleccionar este template **defineRootViewEntity**

![image](./img/TEMPLATE_CDS_FOR_API.png)

``` abap
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'CDS Cliente'
@Metadata.ignorePropagatedAnnotations: true

define root view entity ZI_CLIENTE
  as select from zcliente_01

{
  key cliente_id   as ClienteId,
      nombre       as Nombre,
      ciudad       as Ciudad,
      email        as Email,
      estado       as Estado,
      ultima_modif as UltimaModif

}
```
* Es root porque en la Fase 2 le añadiremos el behavior.

#### 2. Service definition ZSD_CLIENTE

``` abap
@EndUserText.label: 'Service Definition on CDS'
define service ZSD_CLIENTE {
  expose ZI_CLIENTE as Cliente;
}
```

El alias Cliente será el nombre del entity set en la URL.

#### 3. Service binding ZUI_CLIENTE_O4

* Binding Type: OData V4 - Web API.
* Service Definition: ZSD_CLIENTE.
* Activar y publicar (Publish).

![image](./img/SERVICE_BINDING.png)

Con esto ya se puede probar presionando **Test**

![image](./img/TEST_SWAGGER.png)

Equivalencias de consulta en BTP

con SEGW

/sap/opu/odata/sap/API_PURCHASECONTRACT_PROCESS_SRV/A_PurchaseContract?$top=2

Con BTP

http://localhost:49370/testclient/sap/opu/odata4/sap/zui_cliente_o4/srvd_a2x/sap/zsd_cliente/0001/Cliente?$top=3


con SEGW

/sap/opu/odata/sap/ZAPI_PURCHASEREQ_PROCESS_SRV/$metadata

Con BTP

http://localhost:49370/testclient/sap/opu/odata4/sap/zui_cliente_o4/srvd_a2x/sap/zsd_cliente/0001/$metadata
