``` abap
@VDM.lifecycle.contract.type: #PUBLIC_REMOTE_API
@AbapCatalog.sqlViewName: 'ZV_SOLPED_HDR'
@AbapCatalog.compiler.compareFilter: true
@AbapCatalog.preserveKey: true
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Document Attach for Solicitudes Pedido - Padre (ME53N)'
@ObjectModel.createEnabled: true
@VDM.viewType : #BASIC
@ClientHandling.algorithm: #SESSION_VARIABLE
@ObjectModel.usageType.serviceQuality: #B
@ObjectModel.usageType.sizeCategory: #XXL
@ObjectModel.usageType.dataClass: #TRANSACTIONAL
@ObjectModel.compositionRoot:true
@Metadata.ignorePropagatedAnnotations: true

define view ZA_SOLPED_HEADER
  as select from ztsolped_header as a

  association [1..*] to ZA_SOLPED_ATTACH as _Attachments on  $projection.SolicitudId = _Attachments.SolicitudId
                                                         and $projection.Sapadokid   = _Attachments.Sapadokid
                                                         and $projection.Saparcid    = _Attachments.Saparcid

{
  key a.solicitud_id as SolicitudId,
  key a.sapadokid    as Sapadokid,
  key a.saparcid     as Saparcid,
      @ObjectModel.association.type: [#TO_COMPOSITION_CHILD]
      _Attachments

}

```