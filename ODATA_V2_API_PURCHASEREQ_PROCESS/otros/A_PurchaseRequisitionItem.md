
``` abap
@ObjectModel.usageType.dataClass: #MIXED
@ObjectModel.usageType.serviceQuality: #C
@ObjectModel.usageType.sizeCategory: #L //Inserted by VDM CDS Suite Plugin
@AbapCatalog.sqlViewName: 'APURREQITM'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Item'
@ClientHandling.algorithm: #SESSION_VARIABLE
@VDM.viewType: #COMPOSITE
@VDM.lifecycle.contract.type: #PUBLIC_REMOTE_API
@Metadata.ignorePropagatedAnnotations: true
@ObjectModel: {
 createEnabled: true,
 updateEnabled: true,
 deleteEnabled: false
}
@AccessControl.personalData.blocking: #BLOCKED_DATA_EXCLUDED

define view A_PurchaseRequisitionItem
  as select from I_Purchaserequisitionitem

  association [1..1] to A_PurchaseRequisitionHeader as _PurchaseReqn                on  _PurchaseReqn.PurchaseRequisition = $projection.PurchaseRequisition


  association [0..*] to A_PurReqnAcctAssgmt         as _PurchaseReqnAcctAssgmt      on  _PurchaseReqnAcctAssgmt.PurchaseRequisition     = $projection.PurchaseRequisition
                                                                                    and _PurchaseReqnAcctAssgmt.PurchaseRequisitionItem = $projection.PurchaseRequisitionItem
  association [0..1] to A_PurReqAddDelivery         as _PurchaseReqnDeliveryAddress on  _PurchaseReqnDeliveryAddress.PurchaseRequisition     = $projection.PurchaseRequisition
                                                                                    and _PurchaseReqnDeliveryAddress.PurchaseRequisitionItem = $projection.PurchaseRequisitionItem
  // association [0..*] to A_PurReqItemText            as _PurchaseReqnItemText        on  _PurchaseReqnItemText.PurchaseRequisition     = $projection.PurchaseRequisition
  //                                                                                  and _PurchaseReqnItemText.PurchaseRequisitionItem = $projection.PurchaseRequisitionItem
  association [0..*] to A_PurchaseReqnItemText      as _PurchaseReqnItemText        on  _PurchaseReqnItemText.PurchaseRequisition     = $projection.PurchaseRequisition
                                                                                    and _PurchaseReqnItemText.PurchaseRequisitionItem = $projection.PurchaseRequisitionItem

  //3368768                                                                                   
  association [0..1] to I_Supplier                  as _Subcontractor               on  I_Purchaserequisitionitem.Subcontractor = _Subcontractor.Supplier
  association [0..1] to I_Customer                  as _Customer                    on  I_Purchaserequisitionitem.PurReqnReceivingCustomer = _Customer.Customer                                                                                      
                                                                                      
  ----Extension Association
  association [0..1] to E_Purchaserequisitionitem   as _PurchaseReqnItemExtension   on  _PurchaseReqnItemExtension.PurchaseRequisition     = $projection.PurchaseRequisition
                                                                                    and _PurchaseReqnItemExtension.PurchaseRequisitionItem = $projection.PurchaseRequisitionItem

{

  key I_Purchaserequisitionitem.PurchaseRequisition     as PurchaseRequisition,
  key I_Purchaserequisitionitem.PurchaseRequisitionItem as PurchaseRequisitionItem,
      I_Purchaserequisitionitem.PurchasingDocument,
      I_Purchaserequisitionitem.PurchasingDocumentItem,
      I_Purchaserequisitionitem.PurReqnReleaseStatus,
      @ObjectModel.sapObjectNodeTypeReference: 'PurchaseRequisitionType'      
      I_Purchaserequisitionitem.PurchaseRequisitionType,
      @ObjectModel.readOnly: true
      I_Purchaserequisitionitem.PurchasingDocumentSubtype,
      @ObjectModel.sapObjectNodeTypeReference: 'PurchasingDocumentItemCategory'
      I_Purchaserequisitionitem.PurchasingDocumentItemCategory,
      @Semantics.text: true
      I_Purchaserequisitionitem.PurchaseRequisitionItemText,
      @ObjectModel.sapObjectNodeTypeReference: 'AccountAssignmentCategory'      
      I_Purchaserequisitionitem.AccountAssignmentCategory,

      case I_Purchaserequisitionitem.Material
        when ''
            then
                 I_Purchaserequisitionitem.ExtMaterialForPurg
            else
                 I_Purchaserequisitionitem.Material
      end                                               as Material,

      I_Purchaserequisitionitem.MaterialGroup,
      @ObjectModel.readOnly: true
      I_Purchaserequisitionitem.PurchasingDocumentCategory,
      @Semantics.quantity.unitOfMeasure: 'BaseUnit'    
      I_Purchaserequisitionitem.RequestedQuantity,
      @Semantics.unitOfMeasure: true
//      @ObjectModel.sapObjectNodeTypeReference: 'UnitOfMeasure'   
      I_Purchaserequisitionitem.BaseUnit,
      @Semantics.amount.currencyCode: 'PurReqnItemCurrency'
      // @DefaultAggregation: #NONE
      I_Purchaserequisitionitem.PurchaseRequisitionPrice,
      @Semantics.quantity.unitOfMeasure: 'BaseUnit'
      I_Purchaserequisitionitem.PurReqnPriceQuantity,
      I_Purchaserequisitionitem.MaterialGoodsReceiptDuration,
      I_Purchaserequisitionitem.ReleaseCode,
      I_Purchaserequisitionitem.PurchaseRequisitionReleaseDate,
      
      @ObjectModel.sapObjectNodeTypeReference: 'PurchasingOrganization'      
      case I_Purchaserequisitionitem.PurchasingOrganization      
        when ''
            then
                I_Purchaserequisitionitem.ExtPurgOrgForPurg
            else
                I_Purchaserequisitionitem.PurchasingOrganization
       end                                              as PurchasingOrganization,

      @ObjectModel.sapObjectNodeTypeReference: 'PurchasingGroup'
      I_Purchaserequisitionitem.PurchasingGroup,

      case I_Purchaserequisitionitem.Plant
        when ''
            then
            I_Purchaserequisitionitem.ExtPlantForPurg
            else
            I_Purchaserequisitionitem.Plant
        end                                             as Plant,

      case I_Purchaserequisitionitem.ExtCompanyCodeForPurg
        when ''
            then I_Purchaserequisitionitem.CompanyCode
            else I_Purchaserequisitionitem.ExtCompanyCodeForPurg
        end                                             as CompanyCode,

      @ObjectModel.readOnly: true
      I_Purchaserequisitionitem.SourceOfSupplyIsAssigned,
      I_Purchaserequisitionitem.SupplyingPlant,
      @Semantics.quantity.unitOfMeasure: 'BaseUnit'
      I_Purchaserequisitionitem.OrderedQuantity,
      I_Purchaserequisitionitem.DeliveryDate,
      @ObjectModel.readOnly: true
      I_Purchaserequisitionitem.CreationDate,
      @ObjectModel.sapObjectNodeTypeReference: 'EntProjectProcessingStatus'
      I_Purchaserequisitionitem.ProcessingStatus,
      I_Purchaserequisitionitem.ExternalApprovalStatus,

      case I_Purchaserequisitionitem.PurchasingInfoRecord
       when ''
            then
                I_Purchaserequisitionitem.ExtInfoRecordForPurg
            else
                I_Purchaserequisitionitem.PurchasingInfoRecord
      end                                               as PurchasingInfoRecord,


      case I_Purchaserequisitionitem.Supplier
        when ''
            then
            I_Purchaserequisitionitem.ExtDesiredSupplierForPurg
        else
             I_Purchaserequisitionitem.Supplier
      end                                               as Supplier,
      I_Purchaserequisitionitem.IsDeleted,

      case I_Purchaserequisitionitem.FixedSupplier
        when ''
            then
                I_Purchaserequisitionitem.ExtFixedSupplierForPurg
            else
                I_Purchaserequisitionitem.FixedSupplier
      end                                               as FixedSupplier,

      I_Purchaserequisitionitem.RequisitionerName,
      @ObjectModel.readOnly: true
      I_Purchaserequisitionitem.CreatedByUser,
      @ObjectModel.readOnly: true
      I_Purchaserequisitionitem.PurReqCreationDate,
      
      //Delivery Address ID
      @ObjectModel.readOnly: true
      //_PurchaseReqnDeliveryAddress.AddressID as DeliveryAddressID,
      //--- 3368768 ---//  
      case
      //manual deliv address
      when I_Purchaserequisitionitem.ManualDeliveryAddressID <> ''
        then I_Purchaserequisitionitem.ManualDeliveryAddressID
      //referenced other address
      when I_Purchaserequisitionitem.ItemDeliveryAddressID <> ''
        then I_Purchaserequisitionitem.ItemDeliveryAddressID
      //address from supplier
      when I_Purchaserequisitionitem.Subcontractor <> ''
      //then _Supplier.AddressID
        then _Subcontractor.AddressID
      //address from customer
      when I_Purchaserequisitionitem.PurReqnReceivingCustomer <> ''
      //then _PurReqnItem.AddressID
        then _Customer.AddressID
      //default address from Plant and item not thrid party
      when I_Purchaserequisitionitem.ManualDeliveryAddressID = '' and I_Purchaserequisitionitem.ItemDeliveryAddressID = ''
      and Subcontractor = '' and PurReqnReceivingCustomer = '' and PurchasingDocumentItemCategory <> '5'
        then _Plant.AddressID
      else
        ''
      end                                                              as DeliveryAddressID,
  
      //  I_Purchaserequisitionitem.DeliveryAddressID,
      @ObjectModel.readOnly: true
      I_Purchaserequisitionitem.ManualDeliveryAddressID,
      @Semantics.currencyCode: true
      I_Purchaserequisitionitem.PurReqnItemCurrency,
      I_Purchaserequisitionitem.MaterialPlannedDeliveryDurn,
      I_Purchaserequisitionitem.DelivDateCategory,
      I_Purchaserequisitionitem.MultipleAcctAssgmtDistribution,
     
      @ObjectModel.sapObjectNodeTypeReference: 'StorageLocation'
      case I_Purchaserequisitionitem.StorageLocation
        when ''
            then
            I_Purchaserequisitionitem.ProcmtHubStorageLocation
            else
            I_Purchaserequisitionitem.StorageLocation
        end                                             as StorageLocation, 
      
      // I_Purchaserequisitionitem.StorageLocation,
      I_Purchaserequisitionitem.PurReqnSSPRequestor,
      I_Purchaserequisitionitem.PurReqnSSPAuthor,

      case I_Purchaserequisitionitem.PurchaseContract
      when ''
         then
            I_Purchaserequisitionitem.ExtContractForPurg
         else
            I_Purchaserequisitionitem.PurchaseContract
      end                                               as PurchaseContract,

      @ObjectModel.readOnly: true
      I_Purchaserequisitionitem.PurReqnSourceOfSupplyType,

      case I_Purchaserequisitionitem.PurchaseContractItem
      when ''
         then
            I_Purchaserequisitionitem.ExtContractItemForPurg
         else
            I_Purchaserequisitionitem.PurchaseContractItem
      end                                               as PurchaseContractItem,

      I_Purchaserequisitionitem.ConsumptionPosting,
      I_Purchaserequisitionitem.PurReqnOrigin,
      I_Purchaserequisitionitem.PurReqnSSPCatalog,
      I_Purchaserequisitionitem.PurReqnSSPCatalogItem,
      I_Purchaserequisitionitem.PurReqnSSPCrossCatalogItem,
      I_Purchaserequisitionitem.IsPurReqnBlocked,
    //  @ObjectModel.readOnly: true
      I_Purchaserequisitionitem.ItemDeliveryAddressID,
      I_Purchaserequisitionitem.Language,
      I_Purchaserequisitionitem.IsClosed,
      I_Purchaserequisitionitem.ReleaseIsNotCompleted,
      I_Purchaserequisitionitem.ServicePerformer,
      I_Purchaserequisitionitem.ProductType,
      I_Purchaserequisitionitem.PurchaseRequisitionStatus,
      I_Purchaserequisitionitem.ReleaseStrategy,
      I_Purchaserequisitionitem.PerformancePeriodStartDate,
      I_Purchaserequisitionitem.PerformancePeriodEndDate,
      I_Purchaserequisitionitem.PurchaseOrderPriceType,


      I_Purchaserequisitionitem.SupplierMaterialNumber,
      I_Purchaserequisitionitem.Batch,
      cast( I_Purchaserequisitionitem.MaterialRevisionLevel as revlv ) as MaterialRevisionLevel,
      I_Purchaserequisitionitem.MinRemainingShelfLife,
      @Semantics.amount.currencyCode: 'PurReqnItemCurrency'
      I_Purchaserequisitionitem.ItemNetAmount,
      I_Purchaserequisitionitem.GoodsReceiptIsExpected,
      I_Purchaserequisitionitem.InvoiceIsExpected,
      I_Purchaserequisitionitem.GoodsReceiptIsNonValuated,
      I_Purchaserequisitionitem.RequirementTracking,
      @ObjectModel.sapObjectNodeTypeReference: 'MRPController'
      I_Purchaserequisitionitem.MRPController,
      @ObjectModel.sapObjectNodeTypeReference: 'SalesTaxCode'      
      I_Purchaserequisitionitem.TaxCode,
      I_Purchaserequisitionitem.PurchaseRequisitionIsFixed,
      @ObjectModel.readOnly: true
      I_Purchaserequisitionitem.AddressID,
      @ObjectModel.readOnly: true
      I_Purchaserequisitionitem.LastChangeDateTime,
      @ObjectModel.readOnly: true
      I_Purchaserequisitionitem.Reservation,
      
      // Limits
      @Semantics.amount.currencyCode: 'PurReqnItemCurrency'
      I_Purchaserequisitionitem.ExpectedOverallLimitAmount,
      @Semantics.amount.currencyCode: 'PurReqnItemCurrency'
      I_Purchaserequisitionitem.OverallLimitAmount,
      I_Purchaserequisitionitem.PurContractForOverallLimit, 
      I_Purchaserequisitionitem.PurContractItemForOverallLimit,
      

      /*EBAN_TECH fields*/
      I_Purchaserequisitionitem.PurReqnExternalReference,
      I_Purchaserequisitionitem.PurReqnItemExternalReference,
      I_Purchaserequisitionitem.PurReqnExternalSystemId,
      I_Purchaserequisitionitem.PurReqnExternalSystemType,

      /* deprecated from 1808. Still available for compatibility */
      I_Purchaserequisitionitem.PurReqnTypeExternalReference,
      I_Purchaserequisitionitem.PurReqnProcessingType,
      I_Purchaserequisitionitem.PurReqnProcessingDateTime,
      //HUB Fields
      @Consumption.hidden: true
      I_Purchaserequisitionitem.ExtMaterialForPurg,
      @Consumption.hidden: true
      I_Purchaserequisitionitem.ExtFixedSupplierForPurg,
      @Consumption.hidden: true
      I_Purchaserequisitionitem.ExtDesiredSupplierForPurg,
      @Consumption.hidden: true
      I_Purchaserequisitionitem.ExtContractForPurg,
      @Consumption.hidden: true
      I_Purchaserequisitionitem.ExtContractItemForPurg,
      @Consumption.hidden: true
      I_Purchaserequisitionitem.ExtInfoRecordForPurg,
      @Consumption.hidden: true
      I_Purchaserequisitionitem.ExtPlantForPurg,
      @Consumption.hidden: true
      I_Purchaserequisitionitem.ProcmtHubStorageLocation,
      @Consumption.hidden: true
      I_Purchaserequisitionitem.ExtCompanyCodeForPurg,
      @Consumption.hidden: true
      I_Purchaserequisitionitem.ExtPurgOrgForPurg,
      @Consumption.hidden: true
      I_Purchaserequisitionitem.ProcurementHubSourceSystem,
      I_Purchaserequisitionitem.ProcmtHubBackendBusSyst,
      I_Purchaserequisitionitem.SSPAuthorExternalBPIdnNumber,
      I_Purchaserequisitionitem.SSPReqrUserId,
      
      //Field for item Hierarchy
    //  @Feature: 'MM_PUR_PR_ITM_HIER'
    @Feature: 'SW:MM_PUR_SRVCPROC_SFWS_ITM_HIER'
      I_Purchaserequisitionitem.IsOutline,
    //  @Feature: 'MM_PUR_PR_ITM_HIER'
    @Feature: 'SW:MM_PUR_SRVCPROC_SFWS_ITM_HIER'
      I_Purchaserequisitionitem.PurchasingParentItem,
    //  @Feature: 'MM_PUR_PR_ITM_HIER'
    @Feature: 'SW:MM_PUR_SRVCPROC_SFWS_ITM_HIER'
      I_Purchaserequisitionitem.PurgConfigurableItemNumber as PurgConfigurableItemNumber, //EXLIN
    //  @Feature: 'MM_PUR_PR_ITM_HIER'
    @Feature: 'SW:MM_PUR_SRVCPROC_SFWS_ITM_HIER'
      I_Purchaserequisitionitem.PurgExternalSortNumber as PurgExternalSortNumber, //EXSNR
      //End of Item Hierarchy Fields

      /* Associations */
      @ObjectModel.association.type: [ #TO_COMPOSITION_PARENT, #TO_COMPOSITION_ROOT ]
      _PurchaseReqn,
      @ObjectModel.association.type: #TO_COMPOSITION_CHILD
      _PurchaseReqnAcctAssgmt,
      @ObjectModel.association.type: #TO_COMPOSITION_CHILD
      _PurchaseReqnDeliveryAddress,
      @ObjectModel.association.type: #TO_COMPOSITION_CHILD
      _PurchaseReqnItemText
      // @ObjectModel.association.type: #TO_COMPOSITION_CHILD
      //_PurchaseReqnItemText


}

```