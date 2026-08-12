# Desarrollo de servicio OData V2 Deep insert

## Explicacion:
Se desea desarrollar un servicio OData V2 que permita la creación de una solicitud de pedidos y luego que sobre la solicitud se añada ficheros que llegan adjuntos al mensaje.

Transacción ME51N

![image](./img/MIGO_1.png)


## Modelo de datos basado en CDS

#### CDS ROOT

[ZA_SOLPED_HEADER](./otros/ZA_SOLPED_HEADER.md)

#### CDS CHILD

[ZA_SOLPED_ATTACH](./otros/ZA_SOLPED_ATTACH.md)


## Proyecto SEGW - SAP Gateway service builder
Se crea proyecto para crear documentos de solicitu de pedidos (ME51N). 

Se crea el proyecto como exposición de una entidad CDS. Se expone CDS ROOT

![image](./img/SEGW_1.png)

Artefactos generados:
* ZCL_MM_API_SOLPED_ATT_DPC
* [ZCL_MM_API_SOLPED_ATT_DPC_EXT](./otros/ZCL_MM_API_SOLPED_ATT_DPC_EXT.md)
* ZCL_MM_API_SOLPED_ATT_MPC
* ZCL_MM_API_SOLPED_ATT_MPC_EXT
* ZMM_API_SOLPED_ATTACH_ANNO_MDL
* ZMM_API_SOLPED_ATTACH_MDL
* ZMM_API_SOLPED_ATTACH_SRV


### Prueba Postman - POST

![image](./img/POST_1.png)

Fichero Json en Body de mensaje

``` json
{
  "ReferenceDocument" : "0180045407",
  "to_MaterialDocumentItem" : {
      "results" : []
  }
}
```

