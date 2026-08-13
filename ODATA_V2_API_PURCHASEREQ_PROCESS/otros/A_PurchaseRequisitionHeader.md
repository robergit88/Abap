
``` abap
/*===================================================================================
   Compositional Hierarchy (Nodes):

   A_PurchaseRequisitionHeader
     |
     |   1..*
     +------ A_PurchaseRequisitionItem
                |
                |   0..*
                +-------- A_PurReqnAcctAssgmt
                |   0..1
                +-------- A_PurReqAddDelivery
                |   0..*
                +-------- A_PurReqItemText

==================================================================================== */
@AbapCatalog.sqlViewName: 'APURREQH'
@AbapCatalog.compiler.compareFilter: true
@VDM.viewType : #COMPOSITE
@VDM.lifecycle.contract.type: #PUBLIC_REMOTE_API
@ObjectModel.usageType.dataClass: #TRANSACTIONAL
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Purchase Requisition'
@ObjectModel.usageType.serviceQuality: #C
@ObjectModel.usageType.sizeCategory: #L
@ClientHandling.algorithm: #SESSION_VARIABLE
@ObjectModel.compositionRoot:true
@Metadata.ignorePropagatedAnnotations:true
@ObjectModel: {
 createEnabled: true,
 updateEnabled: true,
 deleteEnabled: false
}
define view A_PurchaseRequisitionHeader
  as select distinct from I_Purchaserequisitionitem as _PurchaseRequisition

  association [1..*] to A_PurchaseRequisitionItem as _PurchaseReqnItem on $projection.PurchaseRequisition = _PurchaseReqnItem.PurchaseRequisition
{

      
  key _PurchaseRequisition.PurchaseRequisition,
       @ObjectModel.sapObjectNodeTypeReference: 'PurchaseRequisitionType'
      _PurchaseRequisition.PurchaseRequisitionType,
      _PurchaseRequisition.PurReqnDescription,

      cast (''       as xfeld)   as SourceDetermination,
      //Added for Check functionality in Ariba guided buying
      cast(''        as boolean) as PurReqnDoOnlyValidation,

      /*Associations*/
      @ObjectModel.association.type: #TO_COMPOSITION_CHILD
      _PurchaseReqnItem
}

```