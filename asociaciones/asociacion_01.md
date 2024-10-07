# Exposed Association

### Añadir el atributo KEY como clave ya que su ausencia no realizará el filtro adecuado.

> En el fragmento de código siguiente, 
_item es la asociación expuesta.
¿Por qué?
Porque no se selecciona ningún campo explícitamente de _item.
No hay un JOIN fijado entre dfkkko y dfkkop.


``` sql
@AbapCatalog.sqlViewName: 'ZASSOCIA01'
@AbapCatalog.compiler.compareFilter: true
@AbapCatalog.preserveKey: true
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'asociación 01'
@Metadata.ignorePropagatedAnnotations: true
define view ZCDS_ASSOCIA_01
  as select from dfkkko as header
  association to dfkkop as _detail on header.opbel = _detail.opbel
{
  key  header.opbel,
       _detail
}
```

>No hay ningún campo de _item en la lista de resultados.

![select](/asociaciones/img/follow1.png)

>Pero cuando el usuario hace clic derecho en la salida y va a la asociación,
se mostrarán todos los detalles de dfkkop.
Esta es una unión bajo demanda o diferida.


![detalle](/asociaciones/img/detail1.png)