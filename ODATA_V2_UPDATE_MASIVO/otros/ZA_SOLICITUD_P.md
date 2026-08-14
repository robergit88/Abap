
``` abap
@VDM.lifecycle.contract.type: #PUBLIC_REMOTE_API
@AbapCatalog.preserveKey: true
@AbapCatalog.sqlViewName: 'ZPM_V_SOL_P'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Detalle actualizacion masiva series y equipos'
@VDM.viewType : #COMPOSITE
@ClientHandling.algorithm: #SESSION_VARIABLE

@ObjectModel.createEnabled: false
@ObjectModel.updateEnabled: false
@ObjectModel.usageType.serviceQuality: #C
@ObjectModel.usageType.sizeCategory: #XXL
@ObjectModel.usageType.dataClass: #TRANSACTIONAL

@Metadata.ignorePropagatedAnnotations: true
define view ZA_SOLICITUD_P
  as select from zpm_solicitud_p as a

  association [1..1] to ZA_SOLICITUD_K as _HEADER on $projection.Id = _HEADER.Id

{
  key a.id            as Id,
  key a.sernr         as NumeroSerie,
      a.estado        as Estado,
      a.codigo_error  as CodigoError,
      a.mensaje_error as MensajeError,
      @ObjectModel: { association: { type: [#TO_COMPOSITION_PARENT , #TO_COMPOSITION_ROOT] } }
      _HEADER
}

```