

``` abap
class ZCL_CG_HELPER_EQUI definition
  public
  final
  create public .

public section.

  class-methods S_TIENE_ACTIVO_FIJO
    importing
      !EQUIPMENT type EQUNR
    returning
      value(RESULT) type ABAP_BOOL .
  class-methods S_DETERMINAR_ACTIVO_FIJO
    importing
      !EQUIPMENT type EQUNR
    returning
      value(ET_RETURN) type BAPIRET2_T .
  class-methods S_CHANGE_USER_STATUS_BY_SERNR
    importing
      !SERNR type GERNR
      !USER_STATUS type J_STATUS
    returning
      value(RS_RETURN) type BAPIRET2 .
  class-methods S_CHANGE_MASIVE_USER_STATUS
    importing
      !USER_STATUS type J_STATUS
      !IT_SERNR type RANGES_SERNR
    returning
      value(RS_RESULT) type SYSUUID_C
    raising
      ZCX_ROOT .
protected section.
private section.
ENDCLASS.



CLASS ZCL_CG_HELPER_EQUI IMPLEMENTATION.


  METHOD s_change_masive_user_status.
*&---------------------------------------------------------------------*
*& ID_PROGRAMA: ZCL_CG_HELPER_EQUI=>S_CHANGE_MASIVE_USER_STATUS        *
*&                                                                     *
*& TIPO DE PROGRAMA: METODO                                            *
*&                                                                     *
*& DESCRIPCIÓN: Se actualiza estado de equipo por número de serie      *
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

    DATA lt_return TYPE bapiret2_t.
    DATA ls_header TYPE zpm_solicitud_k.
    DATA lt_detail TYPE zpm_tt_solicitud_p.

*--------------------------------------------------------------------*

    IF it_sernr IS INITIAL.

      "Error interno: La tabla de parámetros está vacía
      MESSAGE e115(ehsb_rc) INTO sy-msgli.

      zcl_exception_helper=>s_raise_exception_root( ).

    ENDIF.

    "Se valida existencia del número de serie
    SELECT
           a~sernr,
           a~equnr,
           a~objnr,
           b~stsma
     FROM v_equi AS a LEFT OUTER JOIN
          jsto AS b ON b~objnr = a~objnr
      FOR ALL ENTRIES IN @it_sernr
       WHERE
        sernr = @it_sernr-low
         INTO TABLE @DATA(lt_sernr).

    IF sy-subrc > 0.

      "No se ha seleccionado ningún objeto
      MESSAGE e047(ih) INTO sy-msgli.

      zcl_exception_helper=>s_raise_exception_root( ).

    ENDIF.

*   Se valida que el estado exista en la configuracion...
    SELECT stsma,
           estat,
           txt04,
           txt30
     FROM tj30t
      FOR ALL ENTRIES IN @lt_sernr
      WHERE
       stsma = @lt_sernr-stsma AND
       spras = @sy-langu AND
       txt04 = @user_status
      INTO TABLE @DATA(lt_tj30t).

    IF sy-subrc > 0.

      "No se ha podido determinar ningún esquema status
      MESSAGE e117(cnif_pi) INTO sy-msgli.

      zcl_exception_helper=>s_raise_exception_root( ).

    ENDIF.

    "Proceso de actualización de estados series...
    LOOP AT it_sernr INTO DATA(ls).

      ls_header-total += 1.

      DATA(to_update) = lt_sernr[ sernr = ls-low ].

      IF to_update IS INITIAL.

        ls_header-fallidos += 1.

        "No existe el número de serie &
        MESSAGE e571(61) WITH ls-low INTO sy-msgli.

        lt_detail = VALUE zpm_tt_solicitud_p( BASE lt_detail (
                              sernr         = ls-low
                              estado        = sy-msgty
                              codigo_error  = sy-msgno
                              mensaje_error = sy-msgli ) ).

        CONTINUE.

      ENDIF.

      DATA(lt_status_syst_tab) = VALUE bapi_itob_status_tab( ).
      DATA(lt_status_user_tab) = VALUE bapi_itob_status_tab( ).
      DATA lv_system_status TYPE j_stext.
      DATA lv_user_status TYPE asttx.

      "se lee estado actual del equipo
      CALL FUNCTION 'ITO3_EQUIPMENT_READ_STATUS'
        EXPORTING
          i_equnr           = to_update-equnr
        IMPORTING
          e_systemstatus    = lv_system_status
          e_userstatus      = lv_user_status
        TABLES
          e_status_syst_tab = lt_status_syst_tab
          e_status_user_tab = lt_status_user_tab
        EXCEPTIONS
          not_successful    = 1
          OTHERS            = 2.

      IF sy-subrc > 0.

        ls_header-fallidos += 1.

        "Para & no existe ningún objeto de status
        MESSAGE e001(bs) WITH to_update-sernr INTO sy-msgli.

        lt_detail = VALUE zpm_tt_solicitud_p( BASE lt_detail (
                                      sernr         = to_update-sernr
                                      estado        = sy-msgty
                                      codigo_error  = sy-msgno
                                      mensaje_error = sy-msgli ) ).

        CONTINUE.

      ENDIF.

      "Se valida que el estado exista en la configuracion...
      DATA(ls_tj30t) = lt_tj30t[ stsma = to_update-stsma
                                 txt04 = user_status ].

      IF ls_tj30t IS INITIAL.

        ls_header-fallidos += 1.

        "El status de usuario & no existe para el esquema de status &.
        MESSAGE e010(bs) WITH user_status to_update-stsma INTO sy-msgli.

        lt_detail = VALUE zpm_tt_solicitud_p( BASE lt_detail (
                                      sernr         = to_update-sernr
                                      estado        = sy-msgty
                                      codigo_error  = sy-msgno
                                      mensaje_error = sy-msgli ) ).

        CONTINUE.

      ENDIF.

      "Se valida que el estado sea un estado actualizable.
      IF lv_system_status = zif_pm_constant=>status-system-almacen AND (
         lv_user_status = zif_pm_constant=>status-usuario-disponible OR
         lv_user_status = zif_pm_constant=>status-usuario-pendiente_revision ).

        "ok.
      ELSE.
        "Registrar error
        ls_header-fallidos += 1.

        "El contador no tiene estado correcto
        MESSAGE e019(zpm_msg) INTO sy-msgli.

        lt_detail = VALUE zpm_tt_solicitud_p( BASE lt_detail (
                                      sernr         = to_update-sernr
                                      estado        = sy-msgty
                                      codigo_error  = sy-msgno
                                      mensaje_error = sy-msgli ) ).

        CONTINUE.

      ENDIF.


      "Se actualiza estado del equipo
      CALL FUNCTION 'STATUS_CHANGE_EXTERN'
        EXPORTING
          client              = sy-mandt
          objnr               = to_update-objnr
          user_status         = ls_tj30t-estat
          set_chgkz           = abap_true
        EXCEPTIONS
          object_not_found    = 1
          status_inconsistent = 2
          status_not_allowed  = 3
          OTHERS              = 4.

      IF sy-subrc > 0.

        ls_header-fallidos += 1.

        "Se ha producido un error interno en la gestión de status
        MESSAGE e607(bs) INTO sy-msgli.

        lt_detail = VALUE zpm_tt_solicitud_p( BASE lt_detail (
                                      sernr         = to_update-sernr
                                      estado        = sy-msgty
                                      codigo_error  = sy-msgno
                                      mensaje_error = sy-msgli ) ).

        CONTINUE.

      ENDIF.
*
      ls_header-exitosos += 1.

      "El status ha sido modificado
      MESSAGE s016(bd) INTO sy-msgli.

      lt_detail = VALUE zpm_tt_solicitud_p( BASE lt_detail (
                                    sernr         = to_update-sernr
                                    estado        = sy-msgty
                                    codigo_error  = sy-msgno
                                    mensaje_error = sy-msgli ) ).

    ENDLOOP.

    "Se graba en BB.DD

    ls_header-id = cl_system_uuid=>create_uuid_c32_static( ).
    ls_header-user_status = user_status.
    ls_header-creaauthor  = sy-uname.
    ls_header-creadate    = sy-datum.
    ls_header-creatime    = sy-uzeit.

    INSERT zpm_solicitud_k FROM ls_header.

    LOOP AT lt_detail ASSIGNING FIELD-SYMBOL(<f>).
      <f>-id         = ls_header-id.
      <f>-creaauthor = sy-uname.
      <f>-creadate   = sy-datum.
      <f>-creatime   = sy-uzeit.
    ENDLOOP.

    INSERT zpm_solicitud_p FROM TABLE lt_detail.

    COMMIT WORK AND WAIT.

    rs_result = ls_header-id.

  ENDMETHOD.


  method s_change_user_status_by_sernr.
*&---------------------------------------------------------------------*
*& ID_PROGRAMA: ZCL_CG_HELPER_EQUI=>S_CHANGE_USER_STATUS_BY_SERNR      *
*&                                                                     *
*& TIPO DE PROGRAMA: METODO                                            *
*&                                                                     *
*& DESCRIPCIÓN: Se actualiza estado de equipo por número de serie      *
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

    "Se valida existencia del número de serie
    select a~equnr,
           a~objnr,
           b~stsma
     from v_equi as a left outer join
          jsto as b on b~objnr = a~objnr
      where
       sernr = @sernr
        into @data(ls) up to 1 rows.
    endselect.

    if sy-subrc > 0.

      "No se ha seleccionado ningún objeto
      message e047(ih) into sy-msgli.

      zcl_exception_helper=>s_add_mess_to_bapiret2( importing es_return = rs_return ).

      return.

    endif.

    data(lt_status_syst_tab) = value bapi_itob_status_tab( ).
    data(lt_status_user_tab) = value bapi_itob_status_tab( ).
    data lv_system_status type j_stext.
    data lv_user_status type  asttx.

    call function 'ITO3_EQUIPMENT_READ_STATUS'
      exporting
        i_equnr           = ls-equnr
      importing
        e_systemstatus    = lv_system_status
        e_userstatus      = lv_user_status
      tables
        e_status_syst_tab = lt_status_syst_tab
        e_status_user_tab = lt_status_user_tab
      exceptions
        not_successful    = 1
        others            = 2.

    if sy-subrc > 0.

      "Para & no existe ningún objeto de status
      message e001(bs) with sernr into sy-msgli.

      zcl_exception_helper=>s_add_mess_to_bapiret2( importing es_return = rs_return ).

      return.

    endif.

*   Se valida que el estado exista en la configuracion...
    select stsma,
           estat,
           txt04,
           txt30
     from tj30t
      where
       stsma = @ls-stsma and
       spras = @sy-langu and
       txt04 = @user_status
      into @data(ls_tj30t) up to 1 rows.
    endselect.

    if sy-subrc > 0.

      "El status de usuario & no existe para el esquema de status &.
      message e010(bs) with user_status ls-stsma into sy-msgli.

      zcl_exception_helper=>s_add_mess_to_bapiret2( importing es_return = rs_return ).

      return.

    endif.

    "Se actualiza estado del equipo
    call function 'STATUS_CHANGE_EXTERN'
      exporting
        client              = sy-mandt
        objnr               = ls-objnr
        user_status         = ls_tj30t-estat
        set_chgkz           = abap_true
      exceptions
        object_not_found    = 1
        status_inconsistent = 2
        status_not_allowed  = 3
        others              = 4.

    if sy-subrc > 0.

      "Se ha producido un error interno en la gestión de status
      message e607(bs) into sy-msgli.

      zcl_exception_helper=>s_add_mess_to_bapiret2( importing es_return = rs_return ).

      return.

    endif.

    commit work and wait.

    "El status ha sido modificado
    message s016(bd) into sy-msgli.

    zcl_exception_helper=>s_add_mess_to_bapiret2( importing es_return = rs_return ).

  endmethod.


  method s_determinar_activo_fijo.
*&---------------------------------------------------------------------*
*& ID_PROGRAMA: ZCL_CG_HELPER_EQUI=>S_DETERMINAR_ACTIVO_FIJO           *
*&                                                                     *
*& TIPO DE PROGRAMA: METODO                                            *
*&                                                                     *
*& DESCRIPCIÓN: Determinar activo fijo asignado del Equipo             *
*&                                                                     *
*& AUTOR: 99010760 - Roberto Puma                                      *
*&                                                                     *
*& FECHA DE CREACIÓN: 11.06.2026                                       *
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

    data ls_equi type equi. " Equipment Master Data
    data ls_eqkt type eqkt. " Equipment Short Text
    data ls_equz type equz. " Equipment Usage Period
    data ls_iloa type iloa. " Equipment Location Data

    try.

        new cl_ie01( )->if_ie01~equipment_read(
           exporting equi_no      = equipment
                     reading_date = '99991231'
           importing equi         = ls_equi
                     eqkt         = ls_eqkt
                     equz         = ls_equz
                     iloa         = ls_iloa ).

      catch cx_eam_wf_appl_exception_w_msg into data(appl_exception).

        " Appl. workflow exception with message
        data(msg_tab) = appl_exception->get_msg_tab( ).

        et_return = value #( for ls in msg_tab
                             ( type       = ls-msgty
                               id         = ls-msgid
                               number     = ls-msgno
                               message    = ls-message
                               message_v1 = ls-msgv1
                               message_v2 = ls-msgv2
                               message_v3 = ls-msgv3
                               message_v4 = ls-msgv4 ) ).
        return.
    endtry.

    set update task local.

    "Se ejecuta funcion que determina activo fijo por equipo
    call function 'ZFM_FIXEDASSET_AUTO_DET_IE02'
      exporting data_equi = ls_equi
                data_equz = ls_equz
                data_eqkt = ls_eqkt
                data_iloa = ls_iloa.

    commit work and wait.

  endmethod.


  method s_tiene_activo_fijo.
*&---------------------------------------------------------------------*
*& ID_PROGRAMA: ZCL_CG_HELPER_EQUI=>S_TIENE_ACTIVO_FIJO                *
*&                                                                     *
*& TIPO DE PROGRAMA: METODO                                            *
*&                                                                     *
*& DESCRIPCIÓN: Se verifica si el equipo tiene activo fijo asignado    *
*&                                                                     *
*& AUTOR: 99010760 - Roberto Puma                                      *
*&                                                                     *
*& FECHA DE CREACIÓN: 11.06.2026                                       *
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

    "Por defecto se asume que el equipo no tiene activo fijo
    result = abap_false.

    select anlnr from v_equi
      where equnr = @equipment
        and datbi = '99991231'
      into @data(ls)
      up to 1 rows.
    endselect.

    if sy-subrc > 0.
      return.
    endif.

    if ls is not initial.
      "El equipo si tiene activo fijo
      result = abap_true.
    endif.

  endmethod.
ENDCLASS.
```