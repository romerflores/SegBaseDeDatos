# FUNCIONES EN MARIA DB

Existen dos tipos de funciones

1. **Funciones Integradas**
Son las que son parte del sistema, estan predefinidas, nos permiten hacer operaciones. Ej, concatenacion, substring
2. **Funciones Definidas por el Usuario**
Son instruciones logicas, que el usuario almacena y para usarlas y reutilizar codigo.

Estas funciones contribuyen a la seguridad, ya que nos permiten ocultar logica de codigo, ocultar complejidad, y control de ejecucion de funciones, es decir se puede controlar quiene ejecutan que funciones.

## Sintaxis

La sintaxis basica es la siguiente:

```sql
DELIMITER //
```

```sql
CREATE FUNCTION nombre_de_la_funcion (
    p_parametro1 INT, -- TIPO DE DATO
    p_parametro2 BOOLEAN,
    ...
)
RETURNS INT --(Tipo de dato de retorno)
[DETERMINISTIC | NOT DETERMINISTIC | SQL SECURITY {DEFINER | INVOKER}]
BEGIN
    --Declaracion de variables(opcional)
    DECLARE nombre_variable INT [DEFAULT 0];

    RETURN valor_a_retornar;
END//
```

```sql
DELIMITER ;
```

Ejemplo, una funcion que recibe dos numeros y devuelve su suma

```sql
DELIMITER //

CREATE FUNCTION suma(
    p_a INT,
    P_B INT
)
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE res INT DEFAULT 0;

    SET res=p_a+p_b;

    RETURN res;
END //

DELIMITER ;
```

```sql
--para usar
SELECT suma(5,4);
```

Para crear una funcion, el usuario en custion debe contar con el permiso **CREATE ROUTINE**, y si quiere ejecutarlas debera usar el permiso **EXECUTE**

## 1. Funciones de validacion de entrada

Ejemplo: crear una funcion que valide el formato de un correo electronico

```sql
DELIMITER //
CREATE FUNCTION validar_correo(
    p_correo VARCHAR(100)
)
RETURN BOOLEAN
DETERMINISTIC
BEGIN
    DECLARE res BOOLEAN;

    RETURN p_correo REGEXP '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$';
END //

DELIMITER ;

```

## 2. Funciones de cifrado y hasheo

### Funciones de Hash

```sql

SELECT SHA2('esto_es_una_contrasenia',256);

UPDATE customer 
SET password = SHA2('nueva_contrasenia123',512);
WHERE customer.id=1;

```

### Funciones de Cifrado

Aca vamos a observar el uso de la funcion AES_ENCRYPT Y AES_DECRYPT

```sql
ALTER TABLE customer
ADD COLUMN email1 VARBINARY(255);

UPDATE customer SET email1 = AES_ENCRYPT('elsa@mail.com', 'claveroja') WHERE customer_id = 600;

SELECT CAST(AES_DECRYPT(email1, 'claveroja') AS CHAR) AS email_descifrado
FROM sakila.customer WHERE customer_id = 600;
```

## Funciones de Gestion de usuarios y permisos

```sql
--para obtener el usuario actual
SELECT CURRENT_USER(),USER(),SESSION_USER(),SYSTEM_USER();

--verificar el rol actual
SELECT CURRENT_ROLE();
```

## Funciones de auditoria y Registro

### Funciones de informacion de conexion

```sql
SELECT CONNECTION_ID(), VERSION(), DATABASE();
```

### Registro de cambios
Para este registro de datos se van a usar TRIGGERS

```SQL
DELIMITER //

CREATE TRIGGER audit_customer_update
AFTER UPDATE ON customer
FOR EACH ROW
BEGIN
    INSERT INTO audit_log (user, table_name, action, old_value, new_value, change_date)
    VALUES (CURRENT_USER(), 'customer', 'UPDATE', OLD.email, NEW.email, NOW());
END //

DELIMTIER ;

```

## Funciones de mascara de datos

Estas funciones nos ayudad a enmascarar datos que podrian ser sensibles, de modo que solo usuarios con permisos parciales, puedan acceder parcialmente al dato en cuestion.

Ej: $romer141016@gmail.com$ -> $r------16@g----.com$

```sql
--enmascarar un email
SELECT customer_id,
CONCAT(LEFT(email,3),'***',SUBSTRING_INDEX(email,'@',-1))
as email_masked
FROM customer

--enmascarar una targeta
SELECT customer_id,
CONCAT(LEFT(card_number,3),'***-*****')
as masked_card
FROM customer
```