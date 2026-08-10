``` abap
@VDM.lifecycle.contract.type: #PUBLIC_REMOTE_API
@AbapCatalog.preserveKey: true
@AbapCatalog.sqlViewName: 'ZAMATDOCITEM'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Document Items'
@VDM.viewType : #COMPOSITE
@ClientHandling.algorithm: #SESSION_VARIABLE

@ObjectModel.createEnabled: false
@ObjectModel.updateEnabled: false
@ObjectModel.usageType.serviceQuality: #C
@ObjectModel.usageType.sizeCategory: #XXL
@ObjectModel.usageType.dataClass: #TRANSACTIONAL

@Metadata.ignorePropagatedAnnotations: true
define view ZA_MaterialDocumentItem
  as select from I_GoodsMovementDocument

  association [1..1] to ZA_MaterialDocumentHeader  as _MaterialDocumentHeader on  $projection.MaterialDocumentYear = _MaterialDocumentHeader.MaterialDocumentYear
                                                                              and $projection.MaterialDocument     = _MaterialDocumentHeader.MaterialDocument


  association [0..*] to ZA_SerialNumberMaterialDoc as _SerialNumbers          on  $projection.MaterialDocument     = _SerialNumbers.MaterialDocument
                                                                              and $projection.MaterialDocumentItem = _SerialNumbers.MaterialDocumentItem
                                                                              and $projection.MaterialDocumentYear = _SerialNumbers.MaterialDocumentYear

{
  key MaterialDocumentYear,
  key MaterialDocument,
  key MaterialDocumentItem,

      Material,
      Plant,
      StorageLocation,
      Batch,
      GoodsMovementType,
      InventoryStockType,
      InventoryValuationType,
      InventorySpecialStockType,

      Supplier,
      Customer,
      SalesOrder,
      SalesOrderItem,
      SalesOrderScheduleLine,
      PurchaseOrder,
      PurchaseOrderItem,
      _WBSElement.WBSElement,

      ManufacturingOrder,
      ManufacturingOrderItem,
      GoodsMovementRefDocType,
      GoodsMovementReasonCode,

      //Quantities
      @Semantics.unitOfMeasure: true
      MaterialBaseUnit,
      @Semantics.quantity.unitOfMeasure: 'MaterialBaseUnit'
      QuantityInBaseUnit,
      @Semantics.unitOfMeasure: true
      EntryUnit,
      @Semantics.quantity.unitOfMeasure: 'EntryUnit'
      QuantityInEntryUnit,

      //Amounts
      @Semantics.currencyCode: true
      CompanyCodeCurrency,
      @Semantics.amount.currencyCode: 'CompanyCodeCurrency'
      GdsMvtExtAmtInCoCodeCrcy,
      @Semantics.amount.currencyCode: 'CompanyCodeCurrency'
      case when EnteredSlsAmtInCoCodeCrcy <> 0 then SlsPrcAmtInclVATInCoCodeCrcy
                                               else 0 end as SlsPrcAmtInclVATInCoCodeCrcy,

      FiscalYear,
      FiscalYearPeriod,
      FiscalYearVariant,

      IssgOrRcvgMaterial,
      IssgOrRcvgBatch,
      IssuingOrReceivingPlant,
      IssuingOrReceivingStorageLoc,
      IssuingOrReceivingStockType,
      IssgOrRcvgSpclStockInd,
      IssuingOrReceivingValType,

      IsCompletelyDelivered,
      @Semantics.text: true
      MaterialDocumentItemText,
      UnloadingPointName,
      ShelfLifeExpirationDate,
      ManufactureDate,
      SerialNumbersAreCreatedAutomly,

      Reservation,
      ReservationItem,
      ReservationIsFinallyIssued,

      SpecialStockIdfgSalesOrder,
      SpecialStockIdfgSalesOrderItem,
      _SpecialStockIdfgWBSElement.WBSElement              as SpecialStockIdfgWBSElement,

      //Automatically created entries
      IsAutomaticallyCreated,
      MaterialDocumentLine,
      MaterialDocumentParentLine,
      HierarchyNodeLevel,

      // Cancellation information
      GoodsMovementIsCancelled,
      ReversedMaterialDocumentYear,
      ReversedMaterialDocument,
      ReversedMaterialDocumentItem,

      // Serial Numbers
      _SerialNumbers,

      @ObjectModel: { association: { type: [#TO_COMPOSITION_PARENT , #TO_COMPOSITION_ROOT] } }
      _MaterialDocumentHeader

}
where
  MaterialDocumentRecordType = 'MDOC'

``` 