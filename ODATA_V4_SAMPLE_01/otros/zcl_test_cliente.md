CLASS zcl_test_cliente DEFINITION
  PUBLIC
  FINAL
  CREATE PUBLIC .

  PUBLIC SECTION.

    INTERFACES if_oo_adt_classrun .
  PROTECTED SECTION.
  PRIVATE SECTION.
ENDCLASS.



CLASS zcl_test_cliente IMPLEMENTATION.

  METHOD if_oo_adt_classrun~main.

    " 1) Pedimos la creacion (todavia NO se guarda en la tabla)
    MODIFY ENTITIES OF zi_cliente
           ENTITY Cliente
           CREATE FIELDS ( ClienteId Nombre Ciudad Email Estado )
           WITH VALUE #( ( %cid      = 'c1'
                           ClienteId = 'CLI9000001'
                           Nombre    = 'Cliente creado por EML'
                           Ciudad    = 'Madrid'
                           Email     = 'eml@ejemplo.com'
                           Estado    = 'A' ) )
           " TODO: variable is assigned but never used (ABAP cleaner)
           MAPPED   DATA(mapped)
           FAILED   DATA(failed)
           " TODO: variable is assigned but never used (ABAP cleaner)
           REPORTED DATA(reported).

    IF failed-cliente IS NOT INITIAL.
      out->write( 'Fallo en MODIFY ENTITIES' ).
      RETURN.
    ENDIF.

    " 2) Confirmamos: aqui es donde se escribe en la base de datos
    COMMIT ENTITIES
           RESPONSE OF zi_cliente
           FAILED   DATA(failed_commit)
           " TODO: variable is assigned but never used (ABAP cleaner)
           REPORTED DATA(reported_commit).

    IF failed_commit-cliente IS NOT INITIAL.
      out->write( 'Fallo en COMMIT ENTITIES' ).
      RETURN.
    ENDIF.

    out->write( 'Cliente creado' ).

    " 3) Comprobamos leyendo de la CDS
    SELECT SINGLE ClienteId, Nombre, Ciudad, Estado
      FROM zi_cliente
      WHERE ClienteId = 'CLI9000001'
      INTO @DATA(ls_cliente).

    IF sy-subrc = 0.
      out->write( |Leido: { ls_cliente-ClienteId } - { ls_cliente-Nombre } - { ls_cliente-Ciudad }| ).
    ENDIF.
  ENDMETHOD.

ENDCLASS.