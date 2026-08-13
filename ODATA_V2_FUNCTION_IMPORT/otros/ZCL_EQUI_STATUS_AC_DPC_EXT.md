# Clase ZCL_EQUI_STATUS_AC_DPC_EXT

Propiedades

![image](/img/DPC_EXT_1.png)

Métodos

![image](/img/DPC_EXT_2.png)

EXECUTE_ACTION

![image](/img/EXECUTE_ACTION.png)

HANDLE_ERRORS

![image](/img/HANDLE_ERRORS.png)


``` abap
class zcl_equi_status_ac_dpc_ext definition
  public
  inheriting from zcl_equi_status_ac_dpc
  create public.

  public section.
    methods /iwbep/if_mgw_appl_srv_runtime~execute_action redefinition.

  protected section.

private section.

  methods HANDLE_ERRORS
    importing
      !IS_RETURN type BAPIRET2
      !IO_MSG_CONTAINER type ref to /IWBEP/IF_MESSAGE_CONTAINER
    raising
      /IWBEP/CX_MGW_BUSI_EXCEPTION .
ENDCLASS.



CLASS ZCL_EQUI_STATUS_AC_DPC_EXT IMPLEMENTATION.


  method /iwbep/if_mgw_appl_srv_runtime~execute_action.
*&---------------------------------------------------------------------*
*& ID_PROGRAMA: ZCL_EQUI_STATUS_AC_DPC_EXT->EXECUTE_ACTION             *
*&                                                                     *
*& TIPO DE PROGRAMA: METODO                                            *
*&                                                                     *
*& DESCRIPCIÓN: Se crea function import para modificaión estado equipo *
*&                                                                     *
*& AUTOR: 99010760 - Roberto Puma                                      *
*&                                                                     *
*& FECHA DE CREACIÓN: 30.06.2026                                       *
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

    case iv_action_name.

      when 'ChangeStatus'.

        " 1. Leer los parámetros de entrada
        data(lv_sernr)  = value gernr( it_parameter[ name = 'Sernr' ]-value optional ).
        data(lv_status) = value j_status( it_parameter[ name = 'UserStatus' ]-value optional ).

        data(ls_return) = zcl_cg_helper_equi=>s_change_user_status_by_sernr(
          exporting
            sernr       = lv_sernr
            user_status = lv_status ).

        "2. Handle errors...
        data(lo_msg_container) = mo_context->get_message_container( ).

        handle_errors( is_return        = ls_return
                       io_msg_container = lo_msg_container ).

        "3. Devolver resultado
        copy_data_to_ref( exporting is_data = ls_return
                          changing  cr_data = er_data ).

      when others.

        " otras acciones del sistema
        try.
            super->/iwbep/if_mgw_appl_srv_runtime~execute_action(
              exporting iv_action_name          = iv_action_name
                        it_parameter            = it_parameter
                        io_tech_request_context = io_tech_request_context
              importing er_data                 = er_data ).
          catch /iwbep/cx_mgw_busi_exception.
          catch /iwbep/cx_mgw_tech_exception.
        endtry.

    endcase.
  endmethod.


  method handle_errors.

    io_msg_container->add_message( iv_msg_id     = is_return-id
                                   iv_msg_type   = is_return-type
                                   iv_msg_number = is_return-number
                                   iv_msg_v1     = is_return-message_v1
                                   iv_msg_v2     = is_return-message_v2
                                   iv_msg_v3     = is_return-message_v3
                                   iv_msg_v4     = is_return-message_v4 ).


    data(lv_worst_error_type) = io_msg_container->get_worst_message_type( ).

    if    lv_worst_error_type = /iwbep/if_message_container=>gcs_message_type-error
       or lv_worst_error_type = /iwbep/if_message_container=>gcs_message_type-abort.

      rollback work.                                  "#EC CI_ROLLBACK.

      raise exception new /iwbep/cx_mgw_busi_exception( textid            = /iwbep/cx_mgw_busi_exception=>business_error
                                                        message_container = io_msg_container ).
    endif.
  endmethod.
ENDCLASS.
```