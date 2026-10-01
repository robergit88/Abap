``` abap
CLASS zcl_carga_cliente DEFINITION
  PUBLIC
  FINAL
  CREATE PUBLIC .

  PUBLIC SECTION.

    INTERFACES if_oo_adt_classrun .

  PROTECTED SECTION.

  PRIVATE SECTION.
  CONSTANTS c_max_registros TYPE i VALUE 100.
ENDCLASS.



CLASS zcl_carga_cliente IMPLEMENTATION.
  METHOD if_oo_adt_classrun~main.
    " Numero de registros a generar (entre 1 y 100). Cambialo aqui.
    DATA(lv_num) = 100.
    lv_num = nmin( val1 = lv_num
                   val2 = c_max_registros ).

    DATA lt_data TYPE STANDARD TABLE OF zcliente_01 WITH DEFAULT KEY.
    DATA lv_ts   TYPE timestampl.

    DATA(lt_ciudades) = VALUE string_table( ( `Madrid` )
                                            ( `Barcelona` )
                                            ( `Valencia` )
                                            ( `Sevilla` )
                                            ( `Bilbao` )
                                            ( `Zaragoza` )
                                            ( `Malaga` )
                                            ( `Vigo` )
                                            ( `Granada` )
                                            ( `Murcia` ) ).

    GET TIME STAMP FIELD lv_ts.

    DO lv_num TIMES.
      DATA(lv_i)   = sy-index.
      DATA(lv_idx) = ( lv_i - 1 ) MOD lines( lt_ciudades ) + 1.

      " El campo client no se rellena: el mandante lo pone el sistema
      APPEND VALUE #( cliente_id   = |CLI{ lv_i WIDTH = 7 ALIGN = RIGHT PAD = '0' }|
                      nombre       = |Cliente de prueba { lv_i }|
                      ciudad       = lt_ciudades[ lv_idx ]
                      email        = |cliente{ lv_i }@ejemplo.com|
                      estado       = COND #( WHEN lv_i MOD 4 = 0 THEN 'I' ELSE 'A' )
                      ultima_modif = lv_ts )
             TO lt_data.
    ENDDO.

    " Borra solo el contenido de ZCLIENTE_01, asi se puede ejecutar varias veces
    DELETE FROM zcliente_01.

    INSERT zcliente_01 FROM TABLE @lt_data.

    IF sy-subrc = 0.
      COMMIT WORK.
      out->write( |Registros insertados: { lines( lt_data ) }| ).
      out->write( |Primer ID: { lt_data[ 1 ]-cliente_id } - Ultimo ID: { lt_data[ lines( lt_data ) ]-cliente_id }| ).
    ELSE.
      ROLLBACK WORK.
      out->write( |Error al insertar los registros (sy-subrc = { sy-subrc })| ).
    ENDIF.
  ENDMETHOD.
ENDCLASS.

```