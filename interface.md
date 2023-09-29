---
layout: page
title: Interface MC
permalink: /interface-mc/
---

# Comunication CA – Interface MC

## CASHIER-> INTERFACE MC

The interface receives connections via tcp/ip to the ip of the master pc with the “Magnetic Cash” system at port 4000. In this example we will connect to IP:10.0.0.2 PORT:4000.

### Cashier opening

<p>It fulfills the function of opening the box.Must always be called before making any charges and bring the price of the card that you should use in the charge.</p>
>>To load boxes of electronic cards, you must always send the "id_date"<<
                                                                           
**Required fields:**

- Transaction_type: It is the type of transaction, in this case it will be "opencash"
- Datetime_device: Actual Hour

```
{
    “transaction_type”: “opencash”,
    “cashier_name”:”Autocashier”,
    “cashier_id”:0, “device_name”: “Autocashier 1”,
    “datetime_device”: “31/12/2020 10:00:00”
}
```

**Succesful code:**

```
{"dateopen":"11/01/23 09:56:56","status":"ok","id_date": "cashier12023-09-28 19:04:26","card_price":"100"}
```

### Promotions

Brings all the promotions present in the database

**Required fields:**

- transaction_type: It is the type of transaction, in this case it will be "promotions”

**Example**

``` 
{
    ”transaction_type”:”promotions”
}
```

**Succesful code**

```
[
    {
        ”nombrepromo”:”Exact Charge”, “precio”:10, “bonus”:2, “credits”:0,
        ”tickets”:0, “tipopromo”:”Charge+”, “tks_value”: 0,01,
        ”total_gift_value”:2}{“nombrepromo”:Exact Charge”,”precio”:20,
        ”bonus”:4, “credits”:0, “tickets”:0, “tipopromo”:”Charge++”,
        ”tks_value”:0.01, “total_gift_value”:4
    }
]
```

**Response fields from this endpoint:**

- nombrepromo”: it is the name of the promotions.
- precio: It is the amount that must be loaded from credit for the promotion to be applied.
- tipopromo:tipopromo can have three variants:  
1. Free Card: with the charge of a certain amount the price of the card will be discounted.
2. Charge+: You have to charge exactly the amount specified for the application of the promotion.
3. Charge++: You must charge the specified amount or more.

### Search Card

**Required fields:**

- transaction_type: It is the type of transaction, in this case it will be "balance".
- mpl: It is the card number, it must be entered with a ';' as first character.

**Example**

```
{
    "transaction_type”:”balance",
    "mpl": ";4610000000050"
}
```

**Succesful code**

```
{“status”:true,“exists”:true,“Balances”:{“tipo”:“N”,“saldo”:1580,“bonus”:100,“tickets”:0,“quantity”:0}
{“status”:true,“exists”:false,“msg”:“card is not in database”}
```

**Error message**

```
{“status”:“error”,“msg”:“Rejected card!!!“} --> This error occurs when an invalid card is inserted or without the ";"
```

### Charge

**Required fields:**

- opencash : It is mandatory to open the box, before carrying out a load, so this field should always be id_date.
- transaction_type: It is the type of transaction, in this case it will be "charge".
- Mpl: It is the card number, it must be entered with a ';'as first character.
- Charge: It is an array where an object with the following fields will be found:
    1. Balance_type: what you will load
    2. amount : How much you will load
    3. payments: It is an array where an object with the following fields will be found:
        - currency:currency with which the customer paid
        - Amount: How much the client had to pay
        - rate: difference with local currency
- rate: difference with local currency
- buycard: If the card is new it should be set to 1, otherwise it should be 0, this parameter deducts the price of the card from the charge.
- promotions:Through this field, the bonus, tickets, and quantity will be loaded.It has the following fields:
    1. nombrepromo: It is the name that the promotion has in the database
    2. precio: It is the amount of credit that must be paid to activate the promotion
    3. bonus: How much bonus will be charged
    4. credits:How much credits will be charged
    5. tickets: How much tickets will be charged
    6. tipopromo: It's the type of promotion, can have three variants:  
        -  Free Card: with the charge of a certain amount the price of the card will be discounted.
        - Charge+: You have to charge exactly the amount specified for the application of the promotion.
        - Charge++: You must charge the specified amount or more.

**Example Charge with promotions**

```

{
    “opencash”:"cashier12023-09-28 19:04:26",
    “transaction_type”: “charge”,
    “mpl”: “;4190013769052”,
    “msj”: “”,
    “transaction_id”: “0123456786”,
    “charge”: [{“balance_type”: “Pesos”,“amount”: 300,“cashier_name”: “Autocashier”,“cashier_id”: 0,
    “device_name”: “Autocashier 1",“datetime_device”:“09/01/2023 16:00:00",
    “payments”: [{“currency”: “Pesos”,“payment_method”: “efectivo”,“Amount”: 30,“rate”: 1.0 }],
    “total_payment_local_currency”: 300,
    “promotions”:[{“nombrepromo”:“500 da 700”,“precio”:500, “bonus”:200, “credits”:0,“tickets”:0, “tipopromo”:“Charge+”}]}] ,
    “card_price”:100,
    “buycard”:1
}
```

### Consultation of last movements

You can request the last 20 movements of a card by requesting a call with the following instruction:

**Required fields:**

- Transaction_type: It is the type of transaction,in this case it will be "movements".
- Mpl: It is the card number, it must be entered with a ';' as first character.

**Example**

```
{
    "transaction_type":"movements",
    ”mpl”:";4190013769052",
    ”cashier_name”:”Autocashier”,
    ”Device_Name”:”Autocashier 1”
}
```

**Succesful code**

```
[["2023-09-26 13:07:45", "Autocashier", "charge", "credits", "10.00"], ["2023-09-26 13:07:46", "cash_debit", "debit", "credits", "-1.50"]]
```

**Error message**

- [] => It means that I did not find any record of the entered card

### Debit

Allows you to debit credit

**Required fields:**

- Transaction_type: It is the type of transaction,in this case it will be "movements".
- Mpl: It is the card number, it must be entered with a ';' as first character.
- debit: In this field you will define the card debit data

**Example**

```
{"transaction_type": "debit","mpl": ";8350009433897","vmpl": "","msj": "","transaction_id": "","debit":[{"balance_type": "usd","amount": 15.25,"device_name": "store1"}]}

```

**Succesful code**

```
{"status":"ok","debit_time":"02-Feb-23 15:39:42"}

```

**Error message**

```
{"status":"error","debit_time":"02-Feb-23 15:39:42"}
```

### Vouchers List

Show all voucher in list

**Required fields:**

- Transaction_type: It is the type of transaction,in this case it will be "listvouchers".

**Example**

```
{"transaction_type":"listvouchers"}

```

**Succesful code**

```
[{"id": "1", "name": "Voucher 1", "description": "", "expiration_days_default": "30", "is_active": "1", "deleted": "0", "price": "10.50"}, {"id": "2", "name": "Voucher 2", "description": "", "expiration_days_default": "45", "is_active": "1", "deleted": "0", "price": "15.75"}, {"id": "3", "name": "Voucher 3", "description": "", "expiration_days_default": "25", "is_active": "1", "deleted": "0", "price": "8.75"}]

```

**Error message**


### Get Vouchers Cards

Show all voucher in cards

**Required fields:**

- Transaction_type: It is the type of transaction,in this case it will be "listvouchers".

**Example**

```
{"transaction_type":"getvouchers","mpl": ";7370013852930"}

```

**Succesful code**

```
{"status": true, "vouchers": [{"mpl": ";7370013852930", "vouchers_id": "1", "expiration_date": "2023-12-31 23:59:59", "quantity": "5", "voucher_name": "Voucher 1", "price": "10.5"}, {"mpl": ";7370013852930", "vouchers_id": "2", "expiration_date": "2023-12-31 23:59:59", "quantity": "3", "voucher_name": "Voucher 2", "price": "15.75"}]}

```

**Error message**

```
{"status": error, "vouchers": []}
```

### Vouchers Charge

Charge vouchers for cards

**Required fields:**

- Transaction_type: It is the type of transaction,in this case it will be "chargevoucher".
- mpl: card number.
- buycard: 1 for new card, 0 for client card.
- charge:list of voucher object.
- id_date: open cashier date id.

**Example**

```
{"cashier_id": 1,"mpl": ";7370013852930","buycard": 0,"transaction_type":"chargevouchers","cashier_name": "nico2","charge": [{"name": "Daytona super voucher", "amount": 1, "payAmount":100}],"device_name": "front-magneticash","cardPrice":0,"id_date": "cashier12023-09-28 19:04:26.988427"}

```

**Succesful code**

```
{"status": true, "vouchers": [{"mpl": ";4190013769052", "vouchers_id": "1", "expiration_date": "2023-10-28 20:14:59", "quantity": "4", "voucher_name": "Daytona super voucher", "price": "100.0"}]}

```

**Error message**

```
{"status": false, "vouchers": []}
```


