# Clase ZCL_MM_API_SOLPED_ATT_DPC_EXT

Propiedades 

![image](/ODATA_V2_DEEP_INSERT_ATTACH/img/DPC_EXT1.png)

``` abap
class zcl_mm_api_solped_att_dpc_ext definition
  public
  inheriting from zcl_mm_api_solped_att_dpc
  create public.

  public section.
    methods /iwbep/if_mgw_appl_srv_runtime~create_deep_entity redefinition.

  protected section.

  private section.
    methods handle_errors
      importing iv_is_changeset  type abap_bool
                it_error         type bapiret2_t
                io_msg_container type ref to /iwbep/if_message_container
      raising   /iwbep/cx_mgw_busi_exception.
ENDCLASS.



CLASS ZCL_MM_API_SOLPED_ATT_DPC_EXT IMPLEMENTATION.


  method /iwbep/if_mgw_appl_srv_runtime~create_deep_entity.
    " & ----------------------------------------------------------------------
    " & ID_PROGRAMA: /IWBEP/IF_MGW_APPL_SRV_RUNTIME~CREATE_DEEP_ENTITY      -
    " &                                                                     -
    " & TIPO DE PROGRAMA: METODO                                            -
    " &                                                                     -
    " & DESCRIPCIÓN: Se adjunta documentación para una Solicitud de pedido  -
    " &                                                                     -
    " & AUTOR: 99010760 - Roberto Puma                                      -
    " &                                                                     -
    " & FECHA DE CREACIÓN: 25.05.2026                                       -
    " &                                                                     -
    " &----------------------------------------------------------------------
    " &                                                                     -
    " & HISTORIAL DE CAMBIOS                                                -
    " &                                                                     -
    " &  FECHA       AUTOR   MARCA                                          -
    " &  ------     -------  ------                                         -
    " &  dd.mm.aaaa  usuario                                                -
    " &                                                                     -
    " &----------------------------------------------------------------------

    types begin of ty_attachments.
            include type zcl_mm_api_solped_att_mpc=>ts_za_solped_attachtype as attachments.
    types end of ty_attachments.

    types begin of ty_deep_entity.
            include type zcl_mm_api_solped_att_mpc=>ts_za_solped_headertype as header.
    types   to_attachments type standard table of ty_attachments with default key.
    types end of ty_deep_entity.

    data ls_deep_entity  type ty_deep_entity.
    data ls_toadt        type toadt.
    data lt_header       type standard table of ztsolped_header.
    data lt_attach       type standard table of ztsolped_attach.
    data lt_return       type bapiret2_t.
    data lv_is_changeset type abap_bool value abap_false.
    data lv_string       type string.
    data lv_xstring      type xstring.
    " ---------------------------------------------------------------------

    try.

        " Se lee estructura de datos profunda.
        io_data_provider->read_entry_data( importing es_data = ls_deep_entity ).

      catch /iwbep/cx_mgw_tech_exception into data(lr_tech).
        raise exception lr_tech.
    endtry.

    " Se itera cada fichero...
    loop at ls_deep_entity-to_attachments into data(ls_attachment).

      call function 'SCMS_BASE64_DECODE_STR'
        exporting  input  = ls_attachment-contentbase64
        importing  output = lv_xstring
        exceptions failed = 1
                   others = 2.

      if sy-subrc <> 0.

        message id sy-msgid type sy-msgty number sy-msgno
                with sy-msgv1 sy-msgv2 sy-msgv3 sy-msgv4 into sy-msgli.

        zcl_exception_helper=>s_add_mess_to_bapiret2( changing ct_return = lt_return ).

        continue.

      endif.

      data(lv_objid) = exact saeobjid( |{ ls_attachment-filename }_{ sy-datum }_{ sy-uzeit }| ).

      data(lv_doc_type) = exact saedoktyp( substring_after( val = ls_attachment-mimetype
                                                            sub = '/' ) ).
      lv_string = lv_xstring.

      " important to keep the confirmation in the same luw
      set update task local.

      " Se crea fichero en repositorio SHAREPOINT
      call function 'ZFIFM_CARGA_BO'
        exporting i_file_str   = lv_string
                  i_object_id  = lv_objid
                  i_ar_object  = 'ZSHP365'
                  i_sap_object = 'BUS2105'
                  i_doc_type   = lv_doc_type
                  i_archiv_id  = 'Z1'
                  i_name       = conv toaat-filename( ls_attachment-filename )
        importing e_toadt      = ls_toadt.

      if ls_toadt is initial.

        " Error al ejecutar la función &.
        message e168(sy) with 'ZFIFM_CARGA_BO' into sy-msgli.

        zcl_exception_helper=>s_add_mess_to_bapiret2( changing ct_return = lt_return ).
        continue.
      endif.

      " Se enlaza fichero de SHAREPOINT en SAP.
      call function 'ARCHIV_CONNECTION_INSERT'
        exporting  archiv_id             = ls_toadt-contrep_id
                   arc_doc_id            = ls_toadt-arc_doc_id
                   ar_object             = 'ZSHP365'
                   object_id             = conv saeobjid( ls_deep_entity-header-solicitudid ) " ID SOLPED
                   sap_object            = 'BUS2105' " OBJETO SOLPED
                   doc_type              = ls_toadt-doc_class
                   filename              = conv char255( ls_attachment-filename )
                   creator               = sy-uname
                   descr                 = conv char60( ls_attachment-filename )
        exceptions error_connectiontable = 1
                   others                = 2.

      if sy-subrc <> 0.

        message id sy-msgid type sy-msgty number sy-msgno
                with sy-msgv1 sy-msgv2 sy-msgv3 sy-msgv4 into sy-msgli.

        zcl_exception_helper=>s_add_mess_to_bapiret2( changing ct_return = lt_return ).

        continue.

      endif.

      call function 'BAPI_TRANSACTION_COMMIT'
        exporting wait = 'X'.

      append value #( solicitud_id = ls_deep_entity-solicitudid
                      sapadokid    = ls_toadt-arc_doc_id
                      saparcid     = ls_toadt-contrep_id
                      creaauthor   = sy-uname
                      creadate     = sy-datum
                      creatime     = sy-uzeit ) to lt_header.

      append value #( solicitud_id   = ls_deep_entity-solicitudid
                      sapadokid      = ls_toadt-arc_doc_id
                      saparcid       = ls_toadt-contrep_id
                      part_id        = 1
                      object_id      = ls_deep_entity-solicitudid
                      mimetype       = ls_attachment-mimetype
                      filename       = ls_attachment-filename
                      content_base64 = ls_attachment-contentbase64
                      creaauthor     = sy-uname
                      creadate       = sy-datum
                      creatime       = sy-uzeit ) to lt_attach.

      " Enlace creado para el fichero & &.
      message s056(/edge/dc) with ls_deep_entity-solicitudid ls_attachment-filename into sy-msgli.

      zcl_exception_helper=>s_add_mess_to_bapiret2( changing ct_return = lt_return ).

      clear: lv_xstring,
             lv_string,
             ls_toadt.

    endloop.

    data(lo_msg_container) = mo_context->get_message_container( ).

    lv_is_changeset = abap_true.

    handle_errors( iv_is_changeset  = lv_is_changeset
                   it_error         = lt_return
                   io_msg_container = lo_msg_container ).

    if lt_header is not initial.
      insert ztsolped_header from table lt_header.
    endif.

    if lt_attach is not initial.
      insert ztsolped_attach from table lt_attach.
    endif.

    " Se confirman operaciones!
    commit work and wait.

    " ---------------------------------------------------------------------
    " Response de API
    " ---------------------------------------------------------------------

    if lt_attach is initial.
      return.
    endif.

    select * from za_solped_attach
      for all entries in @lt_attach
      where Sapadokid = @lt_attach-sapadokid
      into table @data(aux_attach).

    data(aux_header) = aux_attach[ 1 ].

    " Construir deep entity
    data(ls_result) = value ty_deep_entity( solicitudid    = aux_header-solicitudid
                                            sapadokid      = aux_header-sapadokid
                                            saparcid       = aux_header-saparcid
                                            to_attachments = value #( for aux in aux_attach
                                                                      ( solicitudid   = aux-solicitudid
                                                                        sapadokid     = aux-sapadokid
                                                                        saparcid      = aux-saparcid
                                                                        partid        = aux-partid
                                                                        mimetype      = aux-mimetype
                                                                        filename      = aux-filename
                                                                        contentbase64 = aux-contentbase64 ) ) ).
    copy_data_to_ref( exporting is_data = ls_result
                      changing  cr_data = er_deep_entity ).
  endmethod.


  method handle_errors.
    " & -----------------------------------------------------------------------
    " & ID_PROGRAMA: HANDLE_ERRORS                                          -
    " &                                                                     -
    " & TIPO DE PROGRAMA: METODO                                            -
    " &                                                                     -
    " & DESCRIPCIÓN: Gestión de mensajes de error                           -
    " &                                                                     -
    " & AUTOR: 99010760 - Roberto Puma                                      -
    " &                                                                     -
    " & FECHA DE CREACIÓN: 25.05.2026                                       -
    " &                                                                     -
    " &----------------------------------------------------------------------
    " &                                                                     -
    " & HISTORIAL DE CAMBIOS                                                -
    " &                                                                     -
    " &  FECHA       AUTOR   MARCA                                          -
    " &  ------     -------  ------                                         -
    " &  dd.mm.aaaa  usuario                                                -
    " &                                                                     -
    " &----------------------------------------------------------------------

    loop at it_error into data(ls).

      io_msg_container->add_message( iv_msg_id     = ls-id
                                     iv_msg_type   = ls-type
                                     iv_msg_number = ls-number
                                     iv_msg_v1     = ls-message_v1
                                     iv_msg_v2     = ls-message_v2
                                     iv_msg_v3     = ls-message_v3
                                     iv_msg_v4     = ls-message_v4 ).

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
        " Error al adjuntar un anexo
        io_msg_container->add_message( iv_msg_id     = 'UDM_MSG'
                                       iv_msg_type   = if_nsdm_odata_text_message=>sc_severity-error
                                       iv_msg_number = 170 ).

      endif.

      raise exception new /iwbep/cx_mgw_busi_exception( textid            = /iwbep/cx_mgw_busi_exception=>business_error
                                                        message_container = io_msg_container ).
    endif.
  endmethod.
ENDCLASS.
```