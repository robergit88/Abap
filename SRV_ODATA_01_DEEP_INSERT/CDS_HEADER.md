

``` cds
@VDM.lifecycle.contract.type: #PUBLIC_REMOTE_API
@AbapCatalog.preserveKey: true
@AbapCatalog.sqlViewName: 'ZAMATDOCHEAD'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Document Header'

@ObjectModel.createEnabled: true
@VDM.viewType : #BASIC
@ClientHandling.algorithm: #SESSION_VARIABLE
@ObjectModel.usageType.serviceQuality: #B
@ObjectModel.usageType.sizeCategory: #XXL
@ObjectModel.usageType.dataClass: #TRANSACTIONAL
@ObjectModel.compositionRoot:true

@Metadata.ignorePropagatedAnnotations: true
define view ZA_MaterialDocumentHeader 
 as select from I_GoodsMovementDocument
 
  left outer to one join P_GoodsMovementCode on I_GoodsMovementDocument.TransactionCode = P_GoodsMovementCode.TransactionCode
   
  association [1..*] to ZA_MaterialDocumentItem as _MaterialDocumentItem on  $projection.MaterialDocumentYear = _MaterialDocumentItem.MaterialDocumentYear
                                                                        and $projection.MaterialDocument     = _MaterialDocumentItem.MaterialDocument
{
  key MaterialDocumentYear,
  key MaterialDocument,

      InventoryTransactionType, 
      DocumentDate,
      PostingDate,
      CreationDate,
      CreationTime,
      CreatedByUser,
      MaterialDocumentHeaderText,
      ReferenceDocument,
      VersionForPrintingSlip,
      ManualPrintIsTriggered,

      P_GoodsMovementCode.GoodsMovementCode,
      
      @ObjectModel.association.type: [#TO_COMPOSITION_CHILD]
      _MaterialDocumentItem
} where
      MaterialDocumentRecordType = 'MDOC'
  and IsMaterialDocumentHeader   = 1           

```