# Desarrollo de servicio OData V2 Deep insert

## Modelo de datos basado en CDS

#### CDS ROOT

[ZA_MaterialDocumentHeader](/SRV_ODATA_01_DEEP_INSERT/CDS_HEADER.md)

#### CDS CHILD

[ZA_MaterialDocumentItem](/SRV_ODATA_01_DEEP_INSERT/CDS_ITEM.md)

#### CDS HOJA

[ZA_SerialNumberMaterialDoc](/SRV_ODATA_01_DEEP_INSERT/CDS_SERIE.md)

## Proyecto SEGW - SAP Gateway service builder
Se crea proyecto para crear documentos de entrada de mercadería. 

Se crea el proyecto como exposición de una entidad CDS. Se expone CDS ROOT

![image](/SRV_ODATA_01_DEEP_INSERT/img/SEGW_1.png)

Artefactos generados:
* ZCL_ZMM_API_GRID_DPC
* [ZCL_ZMM_API_GRID_DPC_EXT](/SRV_ODATA_01_DEEP_INSERT/ZCL_ZMM_API_GRID_DPC_EXT.md)
* ZCL_ZMM_API_GRID_MPC
* ZCL_ZMM_API_GRID_MPC_EXT
* ZMM_API_GRID_ANNO_MDL
* ZMM_API_GRID_MDL
* ZMM_API_GRID_SRV

### Clase Auxiliar

[ZCL_MATERIAL_DOCUMENT_API](/SRV_ODATA_01_DEEP_INSERT/ZCL_MATERIAL_DOCUMENT_API.md)

### Prueba Postman - POST

![image](/SRV_ODATA_01_DEEP_INSERT/img/POST_1.png)

Fichero Json en Body de mensaje

``` json
{
  "ReferenceDocument" : "0180045407",
  "to_MaterialDocumentItem" : {
      "results" : []
  }
}
```

