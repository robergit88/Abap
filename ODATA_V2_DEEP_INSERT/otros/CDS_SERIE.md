``` abap
@VDM.lifecycle.contract.type: #PUBLIC_REMOTE_API
@AbapCatalog.preserveKey: true
@AbapCatalog.sqlViewName: 'ZASERNOITEM'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Serial Numbers'
@VDM.viewType : #COMPOSITE
@ClientHandling.algorithm: #SESSION_VARIABLE

@ObjectModel.createEnabled: false
@ObjectModel.updateEnabled: false
@ObjectModel.usageType.serviceQuality: #C
@ObjectModel.usageType.sizeCategory: #XL
@ObjectModel.usageType.dataClass: #TRANSACTIONAL

@Metadata.ignorePropagatedAnnotations: true
define view ZA_SerialNumberMaterialDoc
  as select from I_SerialNumberMaterialDocument
  association [1..1] to ZA_MaterialDocumentItem as _MaterialDocumentItem on  $projection.MaterialDocumentYear = _MaterialDocumentItem.MaterialDocumentYear
                                                                         and $projection.MaterialDocument     = _MaterialDocumentItem.MaterialDocument
                                                                         and $projection.MaterialDocumentItem = _MaterialDocumentItem.MaterialDocumentItem
  association [0..1] to A_Equipment             as _Equipment            on  $projection.Material     = _Equipment.Material
                                                                         and $projection.SerialNumber = _Equipment.SerialNumber
{


  key Material,
  key SerialNumber,
  key MaterialDocument,
  key MaterialDocumentItem,
  key MaterialDocumentYear,

      _Equipment,
      _MaterialDocumentItem

}

```