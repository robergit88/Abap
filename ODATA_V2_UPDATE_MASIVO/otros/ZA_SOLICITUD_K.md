
``` abap
@VDM.lifecycle.contract.type: #PUBLIC_REMOTE_API
@AbapCatalog.preserveKey: true
@AbapCatalog.sqlViewName: 'ZPM_V_SOL_K'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Cabecera actualizacion masiva series y equipos'

@ObjectModel.createEnabled: true
@VDM.viewType : #BASIC
@ClientHandling.algorithm: #SESSION_VARIABLE
@ObjectModel.usageType.serviceQuality: #B
@ObjectModel.usageType.sizeCategory: #XXL
@ObjectModel.usageType.dataClass: #TRANSACTIONAL
@ObjectModel.compositionRoot:true

@Metadata.ignorePropagatedAnnotations: true
define view ZA_SOLICITUD_K
  as select from zpm_solicitud_k as a

  association [1..*] to ZA_SOLICITUD_P as _numerosSerie on $projection.Id = _numerosSerie.Id

{
  key a.id          as Id,
      a.user_status as EstadoDestino,
      a.total       as Total,
      a.exitosos    as Exitosos,
      a.fallidos    as Fallidos,
      @ObjectModel.association.type: [#TO_COMPOSITION_CHILD]
      _numerosSerie
}

```