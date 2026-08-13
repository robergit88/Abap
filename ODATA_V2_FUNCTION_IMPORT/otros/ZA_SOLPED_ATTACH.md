``` abap
@VDM.lifecycle.contract.type: #PUBLIC_REMOTE_API
@AbapCatalog.preserveKey: true
@AbapCatalog.sqlViewName: 'ZSOLPEDATTACH'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Document Attach for Solicitudes Pedido (ME53N)'
@VDM.viewType : #COMPOSITE
@ClientHandling.algorithm: #SESSION_VARIABLE
@ObjectModel.createEnabled: false
@ObjectModel.updateEnabled: false
@ObjectModel.usageType.serviceQuality: #C
@ObjectModel.usageType.sizeCategory: #XXL
@ObjectModel.usageType.dataClass: #TRANSACTIONAL

@Metadata.ignorePropagatedAnnotations: true
define view ZA_SOLPED_ATTACH
  as select from ztsolped_attach as a

  association [1..1] to ZA_SOLPED_HEADER as _HEADER on  $projection.SolicitudId = _HEADER.SolicitudId
                                                    and $projection.Sapadokid   = _HEADER.Sapadokid
                                                    and $projection.Saparcid    = _HEADER.Saparcid
{

  key a.solicitud_id   as SolicitudId,
  key a.sapadokid      as Sapadokid,
  key a.saparcid       as Saparcid,
  key a.part_id        as PartId,
      a.object_id      as ObjectId,
      a.mimetype       as MimeType,
      a.filename       as FileName,
      a.content_base64 as ContentBase64,

      @ObjectModel: { association: { type: [#TO_COMPOSITION_PARENT , #TO_COMPOSITION_ROOT] } }
      _HEADER

}
```
