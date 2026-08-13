# Clase ZCL_API_PURCHASEREQ_P_DPC_EXT

Propiedades 

![image](/ODATA_V2_API_PURCHASEREQ_PROCESS/img/DPC_EXT_1.png)

``` abap

class ZCL_API_PURCHASEREQ_P_DPC_EXT definition
  public
  inheriting from ZCL_API_PURCHASEREQ_P_DPC
  create public .

public section.

  class-data MV_BATCH type XFELD .

  methods /IWBEP/IF_MGW_APPL_SRV_RUNTIME~CREATE_DEEP_ENTITY
    redefinition .
protected section.
private section.

  data:
    mt_return_msgs TYPE STANDARD TABLE OF bapi_matreturn2 .

  methods CHECK_MESSAGES
    importing
      !IT_MESSAGES type BAPIRET2_T
    raising
      /IWBEP/CX_MGW_BUSI_EXCEPTION .
ENDCLASS.



CLASS ZCL_API_PURCHASEREQ_P_DPC_EXT IMPLEMENTATION.


  method /iwbep/if_mgw_appl_srv_runtime~create_deep_entity.
*&---------------------------------------------------------------------*
*& ID_PROGRAMA:   ZCL_API_PURCHASEREQ_P_DPC->CREATE_DEEP_ENTITY        *
*&                                                                     *
*& TIPO DE PROGRAMA: METODO                                            *
*&                                                                     *
*& DESCRIPCIÓN: Se crea SOLPED ME51N.                                  *
*&              Clase base CL_API_PURCHASEREQ_DPC_EXT                  *
*&                                                                     *
*& AUTOR: 99010760 - Roberto Puma                                      *
*&                                                                     *
*& FECHA DE CREACIÓN: 05.06.2026                                       *
*&                                                                     *
*&---------------------------------------------------------------------*
*&                                                                     *
*& HISTORIAL DE CAMBIOS                                                *
*&                                                                     *
*&  FECHA       AUTOR   MARCA                                          *
*&  ------     -------  ------                                         *
*&  dd.mm.aaaa  usuario                                                *
*&                                                                     *
*&---------------------------------------------------------------------*

    types begin of ty_item.
    include type cl_api_purchasereq_mpc=>ts_a_purchaserequisitionitemty as item.
    types end of ty_item.

    types begin of ty_deep_entity.
    include type cl_api_purchasereq_mpc=>ts_a_purchaserequisitionheader as header.
    types   to_purchasereqnitem type standard table of ty_item with default key.
    types end of ty_deep_entity.

    data ls_deep_entity  type ty_deep_entity.

    data lv_header_text type string.
    data lv_preq_no type bapimereqheader-preq_no.
    data ls_hdr_res type bapimereqheader.
    data lt_return  type bapiret2_t.
    data lt_pritemexp type ty_bapimereqitem.

*--------------------------------------------------------------------*
* Lectura de datos del request
*--------------------------------------------------------------------*
    try.

        " Se lee estructura de datos profunda.
        io_data_provider->read_entry_data( importing es_data = ls_deep_entity ).

      catch /iwbep/cx_mgw_tech_exception into data(lr_tech).
        raise exception lr_tech.
    endtry.

*--------------------------------------------------------------------*
* Mapeo estructuras de BAPI
*--------------------------------------------------------------------*
    " Cabecera BAPI
    data(ls_hdr) = value bapimereqheader(
      pr_type = ls_deep_entity-purchaserequisitiontype ).

    data(ls_hdrx) = value bapimereqheaderx(
      pr_type = abap_true ).

    " Posiciones BAPI
    data(lt_pritem) = value ty_bapimereqitemimp(
      for ls_item in ls_deep_entity-to_purchasereqnitem
      (
        preq_item  = ls_item-purchaserequisitionitem
        preq_name  = ls_item-requisitionername
        short_text = ls_item-purchaserequisitionitemtext
        matl_group = ls_item-materialgroup
        quantity   = ls_item-requestedquantity
        unit       = ls_item-baseunit
        deliv_date = ls_item-deliverydate
        plant      = cond #( when ls_item-plant is not initial
                             then ls_item-plant
                             else ls_item-supplyingplant )  " <-- aquí el fix
        pur_group  = ls_item-purchasinggroup
        purch_org  = ls_item-purchasingorganization
        preq_price = ls_item-purchaserequisitionprice
        currency   = ls_item-purreqnitemcurrency
        des_vendor = ls_item-supplier
        acctasscat = ls_item-accountassignmentcategory ) ).

    data(lt_pritemx) = value ty_bapimereqitemx(
      for ls_item in ls_deep_entity-to_purchasereqnitem
      (
        preq_item  = ls_item-purchaserequisitionitem
        preq_name  = abap_true
        short_text = abap_true
        matl_group = abap_true
        quantity   = abap_true
        unit       = abap_true
        deliv_date = abap_true
        plant      = abap_true
        pur_group  = abap_true
        purch_org  = abap_true
        preq_price = abap_true
        currency   = abap_true
        des_vendor = abap_true
        acctasscat = abap_true ) ).

    data(lt_header_text) = value ty_bapimereqheadtext( (
          text_id = 'B01'
          text_form = sy-langu
          text_line = ls_deep_entity-purreqndescription ) ).

*--------------------------------------------------------------------*
* BAPI creación de SOLPEd
*--------------------------------------------------------------------*

* important to keep the confirmation in the same luw
    set update task local.

    call function 'BAPI_PR_CREATE'
      exporting
        prheader     = ls_hdr
        prheaderx    = ls_hdrx
        testrun      = abap_false
      importing
        number       = lv_preq_no
        prheaderexp  = ls_hdr_res
      tables
        return       = lt_return
        pritem       = lt_pritem
        pritemx      = lt_pritemx
        pritemexp    = lt_pritemexp
        prheadertext = lt_header_text.

    check_messages( exporting it_messages = lt_return ).

    if lv_preq_no is initial.
      return.
    endif.

    call function 'BAPI_TRANSACTION_COMMIT'
      exporting
        wait = 'X'.

*--------------------------------------------------------------------*
    " Response de la API
*--------------------------------------------------------------------*

    "Se recupera texto de cabecera
    call method cl_mm_pur_text_helper=>read_text_db
      exporting
        iv_text_id     = 'B01'
        iv_language    = sy-langu
        iv_text_object = 'EBANH'
        iv_text_name   = conv tdobname( lv_preq_no )
      importing
        ev_text        = lv_header_text.

    " Leer cabecera
    select *
      from a_purchaserequisitionheader
      where purchaserequisition = @lv_preq_no
      into @data(ls_header) up to 1 rows.
    endselect.

    " Leer posiciones
    select *
      from a_purchaserequisitionitem
      where purchaserequisition = @lv_preq_no
      into table @data(lt_items).

    " Construir deep entity
    data(ls_result) = value ty_deep_entity(
      purchaserequisition     = ls_header-purchaserequisition
      purchaserequisitiontype = ls_header-purchaserequisitiontype
      purreqndescription      = lv_header_text
      to_purchasereqnitem     = value #(
        for aux in lt_items (
          purchaserequisitionitemtext = aux-purchaserequisitionitemtext
          purchaserequisition         = aux-purchaserequisition
          purchaserequisitionitem     = aux-purchaserequisitionitem
          plant                       = aux-plant
          materialgroup               = aux-materialgroup
          requestedquantity           = aux-requestedquantity
          baseunit                    = aux-baseunit
          deliverydate                = aux-deliverydate
          purchasingorganization      = aux-purchasingorganization
          purchasinggroup             = aux-purchasinggroup
          accountassignmentcategory   = aux-accountassignmentcategory ) ) ).

    copy_data_to_ref(
      exporting is_data = ls_result
      changing  cr_data = er_deep_entity ).

  endmethod.


  method check_messages.
*&---------------------------------------------------------------------*
*& ID_PROGRAMA:   ZCL_API_PURCHASEREQ_P_DPC->CHECK_MESSAGES            *
*&                                                                     *
*& TIPO DE PROGRAMA: METODO                                            *
*&                                                                     *
*& DESCRIPCIÓN: Se revisan mensajes de creacion SOLPED ME51N.          *
*&              Clase base CL_API_PURCHASEREQ_DPC_EXT                  *
*&                                                                     *
*& AUTOR: 99010760 - Roberto Puma                                      *
*&                                                                     *
*& FECHA DE CREACIÓN: 05.06.2026                                       *
*&                                                                     *
*&---------------------------------------------------------------------*
*&                                                                     *
*& HISTORIAL DE CAMBIOS                                                *
*&                                                                     *
*&  FECHA       AUTOR   MARCA                                          *
*&  ------     -------  ------                                         *
*&  dd.mm.aaaa  usuario                                                *
*&                                                                     *
*&---------------------------------------------------------------------*

    data(lo_msg_container) = mo_context->get_message_container( ).

    loop at it_messages into data(ls_error).

      lo_msg_container->add_message( iv_msg_id     = ls_error-id
                                     iv_msg_type   = ls_error-type
                                     iv_msg_number = ls_error-number
                                     iv_msg_v1     = ls_error-message_v1
                                     iv_msg_v2     = ls_error-message_v2
                                     iv_msg_v3     = ls_error-message_v3
                                     iv_msg_v4     = ls_error-message_v4 ).

    endloop.

    data(lv_worst_error_type) = lo_msg_container->get_worst_message_type( ).

    if    lv_worst_error_type = /iwbep/if_message_container=>gcs_message_type-error
       or lv_worst_error_type = /iwbep/if_message_container=>gcs_message_type-abort.

      rollback work.                                  "#EC CI_ROLLBACK.

      if lines( it_messages ) > 1.

        " Al agregar un mensaje genérico, los demás se convierten en "mensajes detallados".
        " en lugar de tener solo el último de los mensajes detallados como mensaje contenedor

        "Error general en tratamiento interno de una solicitud de pedido
        lo_msg_container->add_message( iv_msg_id     = 'CO'
                                       iv_msg_type   = if_nsdm_odata_text_message=>sc_severity-error
                                       iv_msg_number = 399 ).

      endif.

      raise exception new /iwbep/cx_mgw_busi_exception( textid            = /iwbep/cx_mgw_busi_exception=>business_error
                                                        message_container = lo_msg_container ).
    endif.

  endmethod.
ENDCLASS.

```