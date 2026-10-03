managed implementation in class zbp_i_cliente unique;
strict ( 2 );

define behavior for ZI_CLIENTE alias Cliente
persistent table zcliente_01
lock master
authorization master ( instance )

{
 create;

  field ( mandatory : create ) ClienteId;
  field ( readonly : update )  ClienteId;

  mapping for zcliente_01
  {
    ClienteId   = cliente_id;
    Nombre      = nombre;
    Ciudad      = ciudad;
    Email       = email;
    Estado      = estado;
    UltimaModif = ultima_modif;
  }
}