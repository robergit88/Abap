

Propiedades

![Propiedades](./img/DPC_EXT_1.png)

Métodos

![Propiedades](./img/DPC_EXT_2.png)

CREATE_DEEP_ENTITY

![Métodos](./img/DPC_EXT_3.png)

_MAPPING

![Métodos](./img/DPC_EXT_4.png)

CREATE_DOCUMENT

![Métodos](./img/DPC_EXT_5.png)

HANDLE_ERRORS

![Métodos](./img/DPC_EXT_6.png)

Código

``` abap

*"* use this source file for the definition and implementation of
*"* local helper classes, interface definitions and type
*"* declarations
types: begin of ty_s_item.
         include type zcl_zmm_api_grid_mpc=>ts_za_materialdocumentitemtype as item.
types:   to_serialnumbers type standard table of zcl_zmm_api_grid_mpc=>ts_za_serialnumbermaterialdoct with default key,
       end of ty_s_item.


types: begin of ty_deep_entity.
         include type zcl_zmm_api_grid_mpc=>ts_za_materialdocumentheaderty as header.
types:   to_materialdocumentitem type standard table of ty_s_item with default key,
       end of ty_deep_entity.


  method /iwbep/if_mgw_appl_srv_runtime~create_deep_entity.
*&---------------------------------------------------------------------*
*& ID_PROGRAMA: ZCL_ZMM_API_GRID_DPC_EXT->CREATE_DEEP_ENTITY           *
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
*&                                                                     *
*&---------------------------------------------------------------------*

*    while 1 = 1.
*    endwhile.

    data(lo_msg_container) = mo_context->get_message_container( ).

    data(lo_api) = new zcl_material_document_api( ).

    create_document( exporting io_api           = lo_api
                               io_data_provider = io_data_provider
                               io_msg_container = lo_msg_container
                               iv_is_changeset  = abap_false
                     importing er_deep_entity   = er_deep_entity ).
  endmethod.

*&---------------------------------------------------------------------*
*& ID_PROGRAMA: ZCL_ZMM_API_GRID_DPC_EXT->_MAPPING                     *
*&                                                                     *
*& TIPO DE PROGRAMA: METODO                                            *
*&                                                                     *
*& DESCRIPCIÓN: Se realiza mapeo desde estructura profunda             *
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
*&                                                                     *
*&---------------------------------------------------------------------*
  method _mapping.

    data ls_deep_entity type ty_deep_entity.
    data ls_item_create type if_material_document_api=>ty_item_input.
    data lv_item_id     type sy-tabix.

    try.
        " Se lee estrucura de datos profunda.
        io_data_provider->read_entry_data( importing es_data = ls_deep_entity ).

      catch /iwbep/cx_mgw_tech_exception into data(lr_tech).
        raise exception lr_tech.
        return.
    endtry.

    es_header = corresponding if_material_document_api=>ty_header_input( ls_deep_entity ).

    es_header-goodsmovementcode = '01'.

  endmethod.

  method create_document.
*&---------------------------------------------------------------------*
*& ID_PROGRAMA: ZCL_ZMM_API_GRID_DPC_EXT->CREATE_DOCUMENT              *
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
*&                                                                     *
*&---------------------------------------------------------------------*

    data ls_header_create type if_material_document_api=>ty_header_input.
    data lt_item_create   type if_material_document_api=>tt_item_input.
    data lt_sernr_create  type if_material_document_api=>tt_sernr_input.
    data ls_output_header type if_material_document_api=>ty_header_output.
    data lt_message       type if_nsdm_odata_text_message=>tt_message.

    " ---------------------------------------------------------------------

    _mapping( exporting io_data_provider = io_data_provider
              importing es_header        = ls_header_create " ¡datos aqui!
                        it_item          = lt_item_create
                        it_sernr         = lt_sernr_create ).

    " Se crea el doc. material
    data(lv_failed) = io_api->create_material_document( exporting is_header  = ls_header_create
                                                                  it_item    = lt_item_create
                                                                  it_sernr   = lt_sernr_create
                                                        importing es_header  = ls_output_header
                                                                  et_message = lt_message ).

    handle_errors( iv_is_changeset  = iv_is_changeset
                   it_error         = lt_message
                   io_msg_container = io_msg_container ).

    copy_data_to_ref( exporting is_data = ls_output_header
                      changing  cr_data = er_deep_entity ).
  endmethod.

  method handle_errors.
    loop at it_error into data(ls_error).

      io_msg_container->add_message( iv_msg_id     = ls_error-msgid
                                     iv_msg_type   = ls_error-msgty
                                     iv_msg_number = conv #( ls_error-msgno )
                                     iv_msg_v1     = ls_error-msgv1
                                     iv_msg_v2     = ls_error-msgv2
                                     iv_msg_v3     = ls_error-msgv3
                                     iv_msg_v4     = ls_error-msgv4 ).

    endloop.

    data(lv_worst_error_type) = io_msg_container->get_worst_message_type( ).

    if    lv_worst_error_type = /iwbep/if_message_container=>gcs_message_type-error
       or lv_worst_error_type = /iwbep/if_message_container=>gcs_message_type-abort.

      if iv_is_changeset = abap_false.
        rollback work.                                "#EC CI_ROLLBACK.
      endif.

      if lines( it_error ) > 1.

        " Al agregar un mensaje genérico, los demás se convierten en "mensajes detallados".
        " en lugar de tener solo el último de los mensajes detallados como mensaje contenedor
        " No es posible el tratamiento de documentos de material
        io_msg_container->add_message( iv_msg_id     = 'MM_IM_ODATA_API_MDOC'
                                       iv_msg_type   = if_nsdm_odata_text_message=>sc_severity-error
                                       iv_msg_number = 014 ).

      endif.

      raise exception new /iwbep/cx_mgw_busi_exception( textid            = /iwbep/cx_mgw_busi_exception=>business_error
                                                        message_container = io_msg_container ).
    endif.
  endmethod.
  
```