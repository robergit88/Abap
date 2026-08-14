# Clase ZCL_PM_EQUI_STATUS_MA_DPC_EXT

Propiedades

![Propiedades](/ODATA_V2_UPDATE_MASIVO/img/DPC_EXT_1.png)

Métodos

CREATE_DEEP_ENTITY

HANDLE_ERRORS



``` abap
class ZCL_PM_EQUI_STATUS_MA_DPC_EXT definition
  public
  inheriting from ZCL_PM_EQUI_STATUS_MA_DPC
  create public .

public section.

  methods /IWBEP/IF_MGW_APPL_SRV_RUNTIME~CREATE_DEEP_ENTITY
    redefinition .
protected section.
private section.

  methods HANDLE_ERRORS
    importing
      !IS_RETURN type BAPIRET2
      !IO_MSG_CONTAINER type ref to /IWBEP/IF_MESSAGE_CONTAINER
      !IV_IS_CHANGESET type ABAP_BOOL optional
    raising
      /IWBEP/CX_MGW_BUSI_EXCEPTION .
ENDCLASS.



CLASS ZCL_PM_EQUI_STATUS_MA_DPC_EXT IMPLEMENTATION.


  METHOD /iwbep/if_mgw_appl_srv_runtime~create_deep_entity.
*&---------------------------------------------------------------------*
*& ID_PROGRAMA: ZCL_PM_EQUI_STATUS_MA_DPC_EXT->CREATE_DEEP_ENTITY      *
*&                                                                     *
*& TIPO DE PROGRAMA: METODO                                            *
*&                                                                     *
*& DESCRIPCIÓN: Se actualizan estados de series y equipos              *
*&                                                                     *
*& AUTOR: 99010760 - Roberto Puma                                      *
*&                                                                     *
*& FECHA DE CREACIÓN: 04.08.2026                                       *
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

    "child or nodes
    TYPES BEGIN OF ty_series.
    INCLUDE TYPE zcl_pm_equi_status_ma_mpc=>ts_za_solicitud_ptype AS series.
    TYPES END OF ty_series.

    "root
    TYPES BEGIN OF ty_deep_entity.
    INCLUDE TYPE zcl_pm_equi_status_ma_mpc=>ts_za_solicitud_ktype AS header.
    TYPES to_numerosSerie TYPE STANDARD TABLE OF ty_series WITH DEFAULT KEY.
    TYPES END OF ty_deep_entity.

    DATA ls_deep_entity TYPE ty_deep_entity.

*--------------------------------------------------------------------*

    TRY.

        " 1. Se lee estructura de datos profunda.
        io_data_provider->read_entry_data( IMPORTING es_data = ls_deep_entity ).

        DATA(lv_estado_destino) = EXACT j_status( ls_deep_entity-estadodestino ).

        DATA(lt_sernr) = VALUE ranges_sernr( FOR ls IN ls_deep_entity-to_numerosserie
                                              sign = if_fsbp_const_range=>sign_include
                                              option = if_fsbp_const_range=>option_equal
                                               ( low = ls-numeroserie ) ).

      CATCH /iwbep/cx_mgw_tech_exception INTO DATA(lr_tech).
        RAISE EXCEPTION lr_tech.
    ENDTRY.

    " 2. Actualizacion masiva de series
    TRY.

        DATA(id_result) = zcl_cg_helper_equi=>s_change_masive_user_status(
          EXPORTING
            user_status = lv_estado_destino
            it_sernr    = lt_sernr ).

      CATCH zcx_root INTO DATA(lr_root).

        "Handle errors...
        DATA(lo_msg_container) = mo_context->get_message_container( ).

        DATA(return) = lr_root->tab_messages[ 1 ].

        handle_errors(
          is_return        = return
          io_msg_container = lo_msg_container ).

    ENDTRY.

    "3. Se retorna datos del mensaje...
    SELECT k~Id,
           k~user_status,
           k~Total,
           k~Exitosos,
           k~Fallidos
     FROM zpm_solicitud_k AS k
      WHERE id = @id_result
       INTO @DATA(header) UP TO 1 ROWS.
    ENDSELECT.

    ls_deep_entity-id            = header-id.
    ls_deep_entity-estadodestino = header-user_status.
    ls_deep_entity-total         = header-total.
    ls_deep_entity-exitosos      = header-exitosos.
    ls_deep_entity-fallidos      = header-fallidos.

    SELECT p~Id,
           p~sernr,
           p~estado,
           p~codigo_error,
           p~mensaje_error
     FROM zpm_solicitud_p AS p
      WHERE id = @id_result
       INTO TABLE @DATA(lt_series).

    ls_deep_entity-to_numerosSerie = VALUE #( FOR lx IN lt_series
                                      ( Id           = lx-id
                                        NumeroSerie  = lx-sernr
                                        Estado       = lx-estado
                                        CodigoError  = lx-codigo_error
                                        MensajeError = lx-mensaje_error ) ).

    "4. Devolver resultado
    copy_data_to_ref( EXPORTING is_data = ls_deep_entity
                      CHANGING  cr_data = er_deep_entity ).

  ENDMETHOD.


  METHOD handle_errors.
*&---------------------------------------------------------------------*
*& ID_PROGRAMA: ZCL_PM_EQUI_STATUS_MA_DPC_EXT->HANDLE_ERRORS           *
*&                                                                     *
*& TIPO DE PROGRAMA: METODO                                            *
*&                                                                     *
*& DESCRIPCIÓN: Se realiza tratamiento de error para respuesta         *
*&                                                                     *
*& AUTOR: 99010760 - Roberto Puma                                      *
*&                                                                     *
*& FECHA DE CREACIÓN: 05.08.2026                                       *
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

    io_msg_container->add_message( iv_msg_id     = is_return-id
                                   iv_msg_type   = is_return-type
                                   iv_msg_number = CONV #( is_return-number )
                                   iv_msg_v1     = is_return-message_v1
                                   iv_msg_v2     = is_return-message_v2
                                   iv_msg_v3     = is_return-message_v3
                                   iv_msg_v4     = is_return-message_v4 ).

    DATA(lv_worst_error_type) = io_msg_container->get_worst_message_type( ).

    IF    lv_worst_error_type = /iwbep/if_message_container=>gcs_message_type-error
       OR lv_worst_error_type = /iwbep/if_message_container=>gcs_message_type-abort.

      IF iv_is_changeset = abap_false.
        ROLLBACK WORK.                                "#EC CI_ROLLBACK.
      ENDIF.

      " Al agregar un mensaje genérico, los demás se convierten en "mensajes detallados".
      " en lugar de tener solo el último de los mensajes detallados como mensaje contenedor

      "El proceso de actualización se ha cancelado
      io_msg_container->add_message( iv_msg_id     = 'PPELUI'
                                     iv_msg_type   = if_nsdm_odata_text_message=>sc_severity-error
                                     iv_msg_number = 032 ).

      RAISE EXCEPTION NEW /iwbep/cx_mgw_busi_exception( textid            = /iwbep/cx_mgw_busi_exception=>business_error
                                                        message_container = io_msg_container ).
    ENDIF.

  ENDMETHOD.
ENDCLASS.
```