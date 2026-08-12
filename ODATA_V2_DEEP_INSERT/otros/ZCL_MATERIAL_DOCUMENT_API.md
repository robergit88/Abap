# Clase Auxiliar


## Clase ZCL_MATERIAL_DOCUMENT_API

Propiedades

<!-- ![image](./img/API_1.png) -->
![image](/img/API_1.png)

Interfaces

![image](/img/API_2.png)

Métodos

![image](/img/API_3.png)

CREATE_MATERIAL_DOCUMENT

![image](/img/API_4.png)


## Código abap
``` abap
class ZCL_MATERIAL_DOCUMENT_API definition
  public
  final
  create public .

public section.

  interfaces IF_MATERIAL_DOCUMENT_API .
protected section.
private section.

  methods PREPARE_DOCUMENT
    importing
      !IV_DELIVERY type LIKP-VBELN
      !IT_ITEM type IF_MATERIAL_DOCUMENT_API=>TT_ITEM_INPUT
      !IT_SERNR type IF_MATERIAL_DOCUMENT_API=>TT_SERNR_INPUT optional
    exporting
      !ES_DELIVERY_HEADER type BAPIDLVHDR
      !ET_ITEM type BAPI2017_GM_ITEM_CREATE_T
      !ET_SERIAL_NUMBERS type BAPI2017_GM_SERIALNUMBER_T
    changing
      !CT_RETURN type BAPIRET2_T
    returning
      value(RV_FAILED) type ABAP_BOOL .
ENDCLASS.



CLASS ZCL_MATERIAL_DOCUMENT_API IMPLEMENTATION.


  method if_material_document_api~cancel_material_document.
  endmethod.


  method if_material_document_api~cancel_material_document_item.
  endmethod.


  METHOD if_material_document_api~create_material_document.
*&---------------------------------------------------------------------*
*& ID_PROGRAMA: ZCL_MATERIAL_DOCUMENT_API->CREATE_MATERIAL_DOCUMENT    *
*&                                                                     *
*& TIPO DE PROGRAMA: METODO                                            *
*&                                                                     *
*& DESCRIPCIÓN: Se crea documento de material con referencia a         *
*&              entrega entrante                                       *
*&                                                                     *
*& AUTOR: 99010760 - Roberto Puma                                      *
*&                                                                     *
*& FECHA DE CREACIÓN: 12.05.2026                                       *
*&                                                                     *
*&---------------------------------------------------------------------*
*&                                                                     *
*& HISTORIAL DE CAMBIOS                                                *
*&                                                                     *
*&  FECHA       AUTOR   MARCA                                          *
*&  ------     -------  ------                                         *
*&  dd.mm.aaaa  usuario                                                *
*&  17.07.2026  99010760 Se añade campo Distribución de entrega        *
*&                                                                     *
*&---------------------------------------------------------------------*

    DATA ls_goodsmvt_header       TYPE bapi2017_gm_head_01.
    DATA lt_goodsmvt_item         TYPE bapi2017_gm_item_create_t.
    DATA lt_goodsmvt_serialnumber TYPE bapi2017_gm_serialnumber_t.
    DATA ls_goodsmvt_out          TYPE bapi2017_gm_head_ret.
    DATA lt_return                TYPE bapiret2_t.
* >>> BEGIN INSERT <<< 17.07.2026 - 99010760
    DATA ls_delivery_header       TYPE bapidlvhdr.
* >>> END   INSERT <<< 17.07.2026 - 99010760

    " ----------------------------------------------------------------------

    CLEAR: et_message,
           es_header.

    DATA(lv_delivery) = EXACT vbeln_vl( is_header-referencedocument ).

    lv_delivery = |{ lv_delivery ALPHA = IN }|. " Se añaden ceros izq

    " Se aplican validaciones de negocio previas a la contabilización
    rv_failed = prepare_document( EXPORTING iv_delivery        = lv_delivery
                                            it_item            = it_item
                                            it_sernr           = it_sernr
                                  IMPORTING
* >>> BEGIN INSERT <<< 17.07.2026 - 99010760
                                            es_delivery_header = ls_delivery_header
* >>> END   INSERT <<< 17.07.2026 - 99010760
                                            et_item            = lt_goodsmvt_item
                                            et_serial_numbers  = lt_goodsmvt_serialnumber
                                  CHANGING  ct_return          = lt_return ).

    IF rv_failed = abap_true.
      et_message = VALUE #( FOR ls IN lt_return
                            ( msgid = ls-id
                              msgno = ls-number
                              msgty = ls-type
                              msgv1 = ls-message_v1
                              msgv2 = ls-message_v2
                              msgv3 = ls-message_v3
                              msgv4 = ls-message_v4 ) ).
      RETURN.
    ENDIF.

    " Se crea entrada de mercancía
    ls_goodsmvt_header-pstng_date      = sy-datum.
    ls_goodsmvt_header-doc_date        = sy-datum.
    ls_goodsmvt_header-pr_uname        = sy-uname.
* >>> BEGIN MODIFY <<< 17.07.2026 - 99010760
*    ls_goodsmvt_header-ref_doc_no      = is_header-referencedocument.
    ls_goodsmvt_header-ref_doc_no      = COND #( WHEN ls_delivery_header-verur IS INITIAL THEN is_header-referencedocument
                                                 ELSE ls_delivery_header-verur ).
* >>> END   MODIFY <<< 17.07.2026 - 99010760
    ls_goodsmvt_header-ver_gr_gi_slip  = 3. "Vale colectivo
    ls_goodsmvt_header-ver_gr_gi_slipx = 'X'.

    "Para lograr la impresión del documento MIGO
    "el usuario que ejecuta el OData
    "debe tener el parámetro NDR = X o probar lo siguiente, activar el parámetro.
    DATA lv_ndr_prev TYPE c.

    " Guardar valor previo
    GET PARAMETER ID 'NDR' FIELD lv_ndr_prev.

    " Forzar el vale colectivo
    SET PARAMETER ID 'NDR' FIELD 'X'.

    " important to keep the confirmation in the same luw
    SET UPDATE TASK LOCAL.

    CALL FUNCTION 'BAPI_GOODSMVT_CREATE'
      EXPORTING
        goodsmvt_header       = ls_goodsmvt_header
        goodsmvt_code         = is_header-goodsmovementcode " 01
        testrun               = abap_false
      IMPORTING
        goodsmvt_headret      = ls_goodsmvt_out
      TABLES
        goodsmvt_item         = lt_goodsmvt_item
        goodsmvt_serialnumber = lt_goodsmvt_serialnumber
        return                = lt_return.

    et_message = VALUE #( FOR ls IN lt_return
                          ( msgid = ls-id
                            msgno = ls-number
                            msgty = ls-type
                            msgv1 = ls-message_v1
                            msgv2 = ls-message_v2
                            msgv3 = ls-message_v3
                            msgv4 = ls-message_v4 ) ).

    IF ls_goodsmvt_out-mat_doc IS INITIAL.
      rv_failed = abap_true.
      RETURN.
    ENDIF.

    CALL FUNCTION 'BAPI_TRANSACTION_COMMIT'
      EXPORTING
        wait = 'X'.

    SELECT *
      UP TO 1 ROWS
      INTO CORRESPONDING FIELDS OF @es_header
      FROM a_materialdocumentheader
      WHERE materialdocumentyear = @ls_goodsmvt_out-doc_year
        AND materialdocument     = @ls_goodsmvt_out-mat_doc.
    ENDSELECT.

    " Restaurar valor previo
    SET PARAMETER ID 'NDR' FIELD lv_ndr_prev.

  ENDMETHOD.


  METHOD prepare_document.
    " TODO: parameter IT_ITEM is never used (ABAP cleaner)

*&---------------------------------------------------------------------*
*& ID_PROGRAMA: ZCL_MATERIAL_DOCUMENT_API->PREPARE_DOCUMENT            *
*&                                                                     *
*& TIPO DE PROGRAMA: METODO                                            *
*&                                                                     *
*& DESCRIPCIÓN: Se llevan a cabo validaciones previas a la contabiliza *
*&                                                                     *
*& AUTOR: 99010760 - Roberto Puma                                      *
*&                                                                     *
*& FECHA DE CREACIÓN: 12.05.2026                                       *
*&                                                                     *
*&---------------------------------------------------------------------*
*&                                                                     *
*& HISTORIAL DE CAMBIOS                                                *
*&                                                                     *
*&  FECHA       AUTOR   MARCA                                          *
*&  ------     -------  ------                                         *
*&  dd.mm.aaaa  usuario                                                *
*&  17.07.2026 99010760 Se devuelve cabecera de entrega (LIKP)         *
*&                                                                     *
*&---------------------------------------------------------------------*
    DATA lt_vbeln_range TYPE STANDARD TABLE OF bapidlv_range_vbeln.

    DATA(lt_delivery_header) = VALUE bapidlvhdr_t( ).
    DATA(lt_delivery_item) = VALUE bapidlvitem_t( ).
    DATA(lt_item_serial_no) = VALUE /spe/bapidlvitmserno_t( ).
    DATA(lt_return) = VALUE bapiret2_t( ).

    " ------------------------------------------------------------------------

    APPEND VALUE #( sign           = if_fico_general_constants=>g_con_sign_i
                    option         = if_fico_general_constants=>g_con_option_eq
                    deliv_numb_low = iv_delivery ) TO lt_vbeln_range.

    DATA(ls_control) = VALUE bapidlvbuffercontrol( item        = abap_true
                                                   item_status = abap_true
                                                   serno       = abap_true ).

    " Se recuperan datos de la entrega
    CALL FUNCTION 'BAPI_DELIVERY_GETLIST'
      EXPORTING
        is_dlv_data_control = ls_control
      TABLES
        it_vbeln            = lt_vbeln_range
        et_delivery_header  = lt_delivery_header
        et_delivery_item    = lt_delivery_item
        et_item_serial_no   = lt_item_serial_no
        return              = lt_return.

    IF zcl_exception_helper=>s_exists_error( lt_return ) = abap_true.
      ct_return = lt_return.
      rv_failed = abap_true.
      RETURN.
    ENDIF.

    " VALIDACIONES PREVIA A LA CONTABILIZACIÓN..

*    " Las cantidades de series no coinciden
*    if lines( it_sernr[] ) <> lines( lt_item_serial_no[] ).
*
*      message e010(zmm_msg) into sy-msgli.
*
*      zcl_exception_helper=>s_add_mess_to_bapiret2( changing ct_return = ct_return ).
*
*      rv_failed = abap_true.
*      return.
*    endif.
*
*    " Número Serie &1 no pertenece a Entrega &2
*    loop at it_sernr into data(ls_sernr).
*
*      if line_exists( lt_item_serial_no[ serialno = ls_sernr-SerialNumber ] ).
*        continue.
*      endif.
*
*      message e011(zmm_msg) with ls_sernr-SerialNumber iv_delivery into sy-msgli.
*      zcl_exception_helper=>s_add_mess_to_bapiret2( changing ct_return = ct_return ).
*
*    endloop.
*
*    if zcl_exception_helper=>s_exists_error( ct_return ) = abap_true.
*      rv_failed = abap_true.
*      return.
*    endif.

    " Número de Serie &1 con estado &2 &3. Incorrecto.
    SELECT a~vbeln,
           a~posnr,
           a~matnr,
           a~sernr,
           a~equnr,
           b~siststatus04,
           b~userstatus04
      FROM zi_ext0040_ent_contadores AS a
             LEFT OUTER JOIN
               zi_ext0099_contadores AS b ON  b~matnr = a~matnr
                                          AND b~sernr = a~sernr
                                          AND b~equnr = a~equnr
      WHERE a~vbeln = @iv_delivery
      INTO TABLE @DATA(lt_equnr_status).

    LOOP AT lt_equnr_status INTO DATA(ls_equnr_status).

      IF     ls_equnr_status-siststatus04 = 'DISP'
         AND ls_equnr_status-userstatus04 = 'DISP'.

        CONTINUE.
      ENDIF.

      MESSAGE e012(zmm_msg) WITH ls_equnr_status-sernr ls_equnr_status-siststatus04 ls_equnr_status-userstatus04 INTO sy-msgli.
      zcl_exception_helper=>s_add_mess_to_bapiret2( CHANGING ct_return = ct_return ).

    ENDLOOP.

    IF zcl_exception_helper=>s_exists_error( ct_return ) = abap_true.
      rv_failed = abap_true.
      RETURN.
    ENDIF.

    " realizar mapeo de datos...
* >>> BEGIN INSERT <<< 17.07.2026 - 99010760
    CLEAR es_delivery_header.
    es_delivery_header = VALUE #( lt_delivery_header[ 1 ] OPTIONAL ).
* >>> END   INSERT <<< 17.07.2026 - 99010760

    et_item = VALUE #( FOR x IN lt_delivery_item
                       ( material             = x-matnr
                         plant                = x-werks
                         stge_loc             = x-lgort
                         batch                = x-charg
                         move_type            = '101'
                         mvt_ind              = 'B'
                         val_type             = x-bwtar
                         stck_type            = x-insmk
                         spec_stock           = x-sobkz
                         entry_qnt            = x-lfimg
                         deliv_numb_to_search = x-vbeln "REFERENCIA ENTREGA ENTRANTE!
                         deliv_item_to_search = x-posnr "REFERENCIA ENTREGA ENTRANTE!
                         line_id              = x-posnr / 10 ) ).

    et_serial_numbers = VALUE #( FOR y IN lt_item_serial_no
                                 ( matdoc_itm = y-itm_number / 10
                                   serialno   = y-serialno ) ).
  ENDMETHOD.
ENDCLASS.
```