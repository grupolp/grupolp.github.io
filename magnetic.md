---
layout: page
title: magnetic
permalink: /magnetic/
---

# Api Master

## Endpoints

### Slave Login

URL = /slave-login

Cumple la funcion de generar el token de autorizacion del usuario, el mismo tiene un vencimiento con un plazo de un minuto, y esta limitado a la cantidad de licencias habilitadas presentes en la base de datos local Software. Este token debera ser incluido en los headers de cada endpoint.Caso contrario se devolvera un mensaje de error.


- EJEMPLO DE BODY

```
      {
        "username": "mcash",
        "password": "mcash",
        "device_id": "e0d55eb1300c",
        "name": "CASHIER"
        }
```
El username y el password siempre seran "mcash", sin embargo el device_id debera ser obtenido del disposivo y este se utilizara para la generacion del token. A su vez existen distintos tipos de licencias que deberan ser señaladas en el campo "name". Estas seran "CASHIER", "REDEMPTION" Y "AUTOCASHIER".

"CASHIER": Es el tipo de licencia que se utiliza para habilitar el cajero.

"REDEMPTION": Es el tipo de licencia que se utiliza para habilitar el sistema de tickets.

"AUTOCASHIER": Es el tipo de licencia que se utiliza para los cajeros automaticos.


- EJEMPLO DE RESPUESTA

```
{
    "status": "ok",
    "Authorization": "4782f0ea98485bf13ff23c87e24bbf5b",
    "code": 200
}
```
### Slave Ping

URL = /slave-ping

Cumple la funcion de mantener vivo el token generado por el slave-login, debera ser ejecutado durante el tiempo de vida de dicho token para la actualizacion del mismo

- EJEMPLO DE BODY

```
{
    "Authorization": "4782f0ea98485bf13ff23c87e24bbf5b",
    "device_id": "e0d55eb1300c",
    "type":"CASHIER"
}
```

"Authorization": Es el token generado por el slave-login

"device_id": id del dispositivo, debe ser el mismo que el utilizado en el endpoint anterior

"type": tipo de licencia que estas intentando renovar

- EJEMPLO DE RESPUESTA

```
{
    "status": "ok",
    "data": "2023-10-30T17:50:05.035186",
    "code": 200,
    "internalCode": "001"
}
```
## A considerar

Desde este punto a cada endpoint al que se le solicite una peticion sera necesario incluirle un header con las siguientes caracteristicas:

```
{
    "Authorization": "4782f0ea98485bf13ff23c87e24bbf5b CASHIER"
}
```
El mismo incluye el token de autorizacion generado, ademas del tipo de licencia para el que fue otorgado.


### User Login

URL = /login-user

Cumple la funcion de Logueo de usuario, a su vez tambien es el encargado de realizar la apertura de caja que otorgara un ID_Date, el cual es necesario y sera solicitado en endpoints posteriores para la identificacion de los registros de transacciones en el history.

- EJEMPLO DE BODY

```
{
    "username":"usuario",
    "password":"contraseña",
    "app":"CASHIER",
    "device_id":"e0d55eb1300c"
}
```

"username": Es el nombre de usuario

"password": Contraseña del usuario

"app": Tipo de licencia de dicho usuario.

"device_id": Mismo numero de idenficacion del dispositivo utilizado en anteriores endpoints

- EJEMPLO DE RESPUESTA

```
{
    "status": "ok",
    "data": {
        "credits": true,
        "bonus": true,
        "ticket": true,
        "quantity": true,
        "party": true,
        "timeCard": true,
        "cashierInfo": true,
        "redemption": true,
        "voucher": true
    },
    "id_date": "Nico10-11-2023_12:35:12",
    "languagesAvailables": [
        {
            "isoCode": "en",
            "language": "English"
        },
        {
            "isoCode": "es",
            "language": "Español"
        },
        {
            "isoCode": "cn",
            "language": "Chinese"
        }
    ],
    "code": 200
}
```

"data": En este apartado se incluiran todos los permisos del usuario, en caso de contar con alguno de estos permisos en False, notaras que no se te permitira el acceso a la carga del mismo, esto puede configurarse desde el backoffice del operador, o desde uno de sus respectivos endpoints.

"id_date": Este apartado es muy importante, debera ser incluido en todos los endpoints de carga de tarjetas, o de canjeo de tickets, puesto que por medio de este se identificaran las transacciones correspondientes a un usuario en un dia determinado

"languagesAvailables": Este apartado devolvera todos los idiomas disponibles a los que se puede acceder y configurar la aplicacion, generalmente se utiliza para el cargado de listas de configuracion.

### get Card 

Este endpoint de tipo get es el encargado de buscar la tarjeta en la base de datos, tanto lo local como en un servidor remoto. Para la utilizacion del mismo debera ingresarse como parametro el numero de dicha tarjeta.

- EJEMPLO DE URL


```
/getCard/4720013852718
```
- EJEMPLO DE RESPUESTA

`
{
    "status": "ok",
    "data": {
        "number": ";4720013852718",
        "type": "N",
        "balance": 148.0,
        "bonus": 2.0,
        "tickets": 0.0,
        "total": 0.0,
        "lastuse": "2023-10-23T13:36:58",
        "ocupied": null,
        "time": "2023/10/23 10:36:11",
        "lasttime": null,
        "device_id": "0",
        "party": 0,
        "nvtype": "N",
        "uses": 0,
        "lastq": null,
        "quantity": 0,
        "ocupiedcard": null
    },
    "api_datetime": "2023/11/10 09:55:33",
    "code": 200
}
`

"data": En este apartado se muestran todos los atributos y saldos de la tarjeta

"api_datetime": Es la hora del servidor

### getDefault

Este endpoint de tipo get es el encargado de traer todas las configuraciones presentes en el backoffice, es importante ejecutarlo antes de realizar cualquier carga, puesto que es por medio de este que dicha configuracion queda presente en memoria, caso contrario el propio endpoint de carga solicitara la ejecucion del mismo.

Aclaracion: Devuelve 0 para False y 1 para True.

- EJEMPLO DE URL


```
/getDefault
```
- EJEMPLO DE RESPUESTA

```
{
    "status": "ok",
    "data": {
        "VoucherOnOff": 1,
        "allowNewCardVoucher": 1,
        "CobrarNuevaVoucher": 1,
        "CobrarNuevaTimeCard": 1,
        "CobrarNuevaParty": 1,
        "CobrarNuevaQuantity": 1,
        "NewCardPrice": 2.0,
        "MaxCharge": 1000.0,
        "MontoConfir": 100.0,
        "MaxDowlTickets": 1000,
        "MaxChargeTickets": 1000,
        "allowNewCardTimeCard": 1,
        "allowNewCardParty": 1,
        "allowNewCardQuantity": 1,
        "CashierReport": false,
        "Timecard": [],
        "Party": [
            {
                "Charge": 100.0,
                "Time": "1",
                "PartyHours": {
                    "days": 0,
                    "hours": 0,
                    "minutes": 1
                }
            }
        ],
        "Quantity": []
    },
    "code": 200
}
```

"VouchersOnOff": Es la habilitacion de la carga de vouchers.

"allowNewCardVoucher": Es la habilitacion de vauchers para tarjetas nuevas

"CobrarNuevaVoucher": Define si la carga de voucher en tarjeta nueva incluye o no el precio de la tarjeta.

"CobrarNuevaTimeCard":  Define si la carga de TimeCard en tarjeta nueva incluye o no el precio de la tarjeta.

"CobrarnuevaParty":  Define si la carga de Party en tarjeta nueva incluye o no el precio de la tarjeta.

"CobrarNuevaQuantity":  Define si la carga de Quantity en tarjeta nueva incluye o no el precio de la tarjeta.

"NewCardPrice": Es el precio de la tarjeta nueva

"MaxCharge": Define el monto maximo que se puede realizar por carga

"MontoConfir": Define a partir de que monto se debe confirmar la operacion.

"MaxDowlTickets": Monto maximo de descarga de tickets.

"MaxChargeTickets": Monto maximo de carga de tickets,

"allowNewCardTimeCard": Es la habilitacion de TimeCard para tarjetas nuevas,

"allowNewCardParty": Es la habilitacion de Party para tarjetas nuevas,

"allowNewCardQuantity": Es la habilitacion de Quantity para tarjetas nuevas,

"CashierReport":  Es la habilitacion para la consulta del endpoint cashierInfo

"TimeCard, Party y Quantity" : Son las listas de las operaciones de cargas definidas en los backoffices para dichos saldos.


### Charge

URL = '/charge'

Este endpoint es el encargado de realizar las cargas de tarjetas, es importante haber ejecutado con anteriodad el getDefault y el User Login para la utilizacion del mismo. Se puede utilizar para la carga de Credito, Bonus, Tickets, TimeCard, Party y Quantity.

- EJEMPLO DE BODY

```
{
"cashier_id": 1,
"cashier_name": "Nico",
"charge": [{"balance_type": "credito", "amount": 150, "payAmount":"300"}],
"device_name": "front-magneticash",
"id_date": "eliasC08-02-2023_09:32:42",
"msj": "",
"payments": [{"currency": "Pesos", "payment_method": "efectivo", "amount": 150, "rate": 1}],
"total_payment_local_currency": 150,
"transaction_id": 0,
"transaction_type": "charge",
"vmpl": "4720013852718"
}
```

"cashier_id": Es el id del cajero

"cashier_name": Es el nombre del cajero

"charge": Debe ser una lista de objetos. Dichos objetos deben incluir los siguientes parametros:

- "balance_type": Puede ser "credito", "bonus", "tickets", "timeCard", "party" o "quantity",
- "amount": Este es el monto a cargar del saldo seleccionado. Por ejemplo al cargar 20 en timeCard, se estara cargando 20 minutos de TimeCard.
- "payAmount": Este es el monto real que fue abonado, y debe ser incluido en todas las cargas que no sean "tickets" y "bonus" puesto que esos tipos de saldo no se abonan.

"device_name": nombre del dispositivo de carga

"id_date": Es otogado por el user-login, debe ser incluido en este apartado

"msj": es una aclaracion de la transaccion, puede ser un string vacio

"payments": Es una lista de objetos, incluye el tipo de moneda de cambio, el metodo de pago, el monto abonado, y ratio que es el multiplo de la operacion, por lo general sera de 1.

"total_payment_local_currency": Este apartado esta pensado para el caso de que se realizara la carga de varias tarjetas a la vez, de momento no esta implementado, por lo que siempre sera igual al payAmount.

"transaction_id": id de la transaccion

"vmpl": Es el numero de identificacion de la tarjeta a cargar

- EJEMPLO DE RESPUESTA

```
{
    "status": "ok",
    "data": {
        "number": ";4720013852718",
        "type": "N",
        "balance": 448.0,
        "bonus": 2.0,
        "tickets": 0.0,
        "total": 0.0,
        "lastuse": "2023-10-23T13:36:58",
        "ocupied": null,
        "time": "2023/11/10 11:00:48",
        "lasttime": null,
        "device_id": "0",
        "party": 0,
        "nvtype": "N",
        "uses": 0,
        "lastq": null,
        "quantity": 0,
        "ocupiedcard": null
    },
    "promotions": {},
    "api_datetime": "2023/11/10 11:00:48",
    "code": 200
}
```

"data": En este apartado se encuentran los datos actualizados de la tarjeta, luego de realizada la carga.

"promotions": En caso de haberse aplicado alguna promocion, este campo contendra los datos de la misma.

"api_datetime": hora en la que se realizo la transaccion segun el servidor.

### info Card

Endpoint de tipo get que devuelve los registros de transacciones de una tarjeta.

-- EJEMPLO DE URL

```
/info/4880013852149?page=1
```

Se le brinda el numero de la tarjeta en la url y la pagina.

-- EJEMPLO DE RESPUESTA

```
{
    "status": "ok",
    "data": {
        "Historial": [
            {
                "DateHour": "06/10/2023 16:05:53",
                "Type": "charge",
                "Machine": "front-magneticash",
                "CashierName": "nico2",
                "Card": 0.0,
                "CreditsC": 0.0,
                "BonusC": 0.0,
                "TicketC": 0,
                "Credits": 48.0,
                "Bonus": 0.0,
                "Ticket": 0,
                "TimeCard": 1,
                "Party": 0,
                "Qty": 0.0,
                "PromoType": "TimeCard",
                "Promotions": "",
                "IdDate": "eliasC08/02/2023 09:32:42",
                "valorTransa": 300.0
            },
            {
                "DateHour": "04/10/2023 16:18:12",
                "Type": "charge",
                "Machine": "front-magneticash",
                "CashierName": "nico2",
                "Card": 2.0,
                "CreditsC": 48.0,
                "BonusC": 0.0,
                "TicketC": 0,
                "Credits": 48.0,
                "Bonus": 0.0,
                "Ticket": 0,
                "TimeCard": 0,
                "Party": 0,
                "Qty": 0.0,
                "PromoType": "Saldo",
                "Promotions": "",
                "IdDate": "eliasC08/02/2023 09:32:42",
                "valorTransa": 50.0
            }
        ],
        "pages": 1
    },
    "code": 200
}
```

"Historial": Devuelve el listado de registros relacionado a la tarjeta

"pages": Devuelve el numero de paginas totales.

### Cashier Info

Endpoint de tipo get, encargado de calcular el total de dinero ingresado en una caja desde el momento de la apertura de la misma, se brindara como parametro el id_date generado durante el logueo del usuario

- EJEMPLO DE URL

```
/cashierInfo/eliasC08-02-2023_09:32:42
```

- EJEMPLO DE RESPUESTA

```
{
    "Cashier": "Nico",
    "Total": 31685.43,
    "Credito": 5475.0,
    "TimeCard": 5398.0,
    "Party": 400.0,
    "Quantity": 300.0,
    "Bonus": 1102.0,
    "Voucher": 20198.43,
    "TicketsCharge": 2300,
    "TicketsDischarge": 0
}
```

"Cashier": Nombre del cajero

"Total": Total de dinero ingresado hasta el momento (No tiene en cuenta ni el bonus ni los tickets)

"Credito": Total de dinero por carga de credito

"TimeCard": Total de dinero por carga de TimeCard

"Party": Total de dinero por carga de Party

"Quantity": Total de dinero por carga de Quantity

"Bonus": Total de carga de Bonus

"Voucher": Total de dinero por carga de Voucher

"TicketsCharge": Total de carga de tickets

"TicketsDischarge": Total de descarga de tickets

### REDEMPTION

URL = /redemption

Endpoint encargado de realizar el canjeo de tickets por productos seleccionados, este tipo de transaccion esta estrechamente relacionada con el sistema de redemption.

- EJEMPLO DE BODY

```
{
    "totalTK":100,
    "select":[";3870005642136"],
    "selected":"3870005642136",
    "resultado":100,
    "cashier_name":"Carlos",
    "device_name":"Caja 1",
    "id_date":"nico201-06-2023_12:33:05",
    "tid":"505050.101050"
}
```

"totalTK": Es la cantidad de tickets que tiene la tarjeta al momento de realizar el canjeo

"select": Es una lista con el numero de identificacion de la tarjeta, es necesario incluir el ; adelante.

"selected": Es la tarjeta seleccionada para recibir la totalidad de tickets restantes en caso de que se utilice mas de una tarjeta para canjear el producto

"resultado": Es la cantidad de tickets totales que deberan ser descontados

"cashier_name": Nombre del cajero

"device_name": Nombre del dispositivo

"id_date": Es el id_date obtenido de realizar el logueo de usuario

"tid": Este es un numero de identificacion con el que se vinculan las transacciones realizadas entre el Master y el REDEMPTION, puesto que al realizarse en dos dispositivos diferentes, es importante poder vincular el registro del canjeo de producto, con el descuento de tickets, estos tid siempre deberan coincidir en ambas plataformas.

- EJEMPLO DE RESPUESTA

```
{
    "status": "ok",
    "data": "Succesfull",
    "code": 200,
    "internalCode": "001"
}
```

### Language

Endpoint Encargado de setear el idioma en el que se mostraran las respuestas de la api, actualmente estan disponibles en español e ingles.

- EJEMPLO DE URL

```
/language/en
```

- EJEMPLO DE RESPUESTA

```
{
    "status": "ok",
    "data": "Succesfull",
    "code": 200,
    "internalCode": "001"
}
```

### Node

Este endpoint funciona con una aplicacion externa. Se encarga de realizar los registros en la base de datos de las transacciones realizadas por el nodo.

### Get Voucher

Endpoint de tipo get encargado de devolver todos los vouchers disponibles en el store de uso.

- EJEMPLO DE URL

```
/vouchers
```

- EJEMPLO DE RESPUESTA

```
{
    "status": "ok",
    "data": [
        {
            "id": 1,
            "expiration": "2023-12-10T17:09:55.617373",
            "name": "Voucher1",
            "price": 10.99,
            "description": "Este es un voucher de descuento para productos"
        },
        {
            "id": 3,
            "expiration": "2024-01-09T17:09:55.617373",
            "name": "Voucher3",
            "price": 25.5,
            "description": "Este es un voucher especial para aniversarios"
        },
        {
            "id": 107,
            "expiration": "2024-02-08T17:09:55.617373",
            "name": "Competencia",
            "price": 2.0,
            "description": ""
        },
        {
            "id": 110,
            "expiration": "2024-02-08T17:09:55.617373",
            "name": "Paseos",
            "price": 12.0,
            "description": ""
        }
    ],
    "code": 200,
    "internalCode": "001"
}
```

### Get Vouchers Por Tarjeta

Endpoint tipo get encargado de devolver el listado de vouchers asociados a una tarjeta, asi como la cantidad disponibles de los mismos.

- EJEMPLO DE URL

```
/voucher/;5960013775785
```

recibe por parametro el numero de tarjeta.

- EJEMPLO DE RESPUESTA

```
{
    "status": "ok",
    "data": [
        {
            "expiration": "2024-01-09T16:03:30",
            "quantity": 4,
            "name": "Competencia",
            "price": 2.0,
            "description": ""
        }
    ],
    "code": 200,
    "internalCode": "001"
}
```

### post Voucher

URL = /voucher/;5150013852908

Endpoint encargado de la carga de vouchers, debe haberse ejecutado el getDefaults, y el userLogin para su utilizacion.

```
{
    "cashier_id": 1,
    "cashier_name": "Nico",
    "charge": [{"name": "Competencia", "amount": 1, "payAmount":100}],
    "device_name": "front-magneticash",
    "cardPrice":0,
    "id_date": "eliasC08-02-2023_09:32:42"
}
```

"cashier_id":Es el id del cajero

"cashier_name": Es el nombre del cajero

"charge": es una lista de objetos cuyos parametros son los siguientes:

- "name": el nombre del voucher a cargar

- "amount": cantidad de vouchers a cargar

- "payAmount": precio individual del voucher a cargar

"device_name": nombre del dispositivo

"cardPrice": Monto de la tarjeta nueva, en caso de que la tarjeta ya haya sido usada siempre debera ser 0, caso contrario debera usarse el valor obtenido por el defaults

"id_date": dato obtenido durante el logueo de usuario

- EJEMPLO DE RESPUESTA


```
{
    "status": "ok",
    "data": [
        {
            "expiration": "2024-02-08T17:25:22.166848",
            "quantity": 17,
            "name": "Competencia",
            "price": 2.0,
            "description": ""
        }
    ],
    "amount": 2.0,
    "payAmount": 2.0,
    "code": 200,
    "internalCode": "001"
}
```










