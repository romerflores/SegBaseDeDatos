# PROCEDIMEINTOS ALMACENADOS

Son conjuntos de instrucciones $SQL$ predefinidas, que se almacenan en el servidor, y pueden ser usandos mediante una llamada

Se pueden mencionar a los siguientes:

1. Permisos granulares
2. Ocultamiendo de la logica de negocio
3. Prevencion de inyeccion SQL
4. Auditoria y el mantenimento

## Delimitador

Ej:

```sql
DELIMITER //, $$, %%
SELECT * FROM usuarios//
--El codigo finalizara con el nuevo delimitador

DELIMITER;
```

Creando un procedimiento alamcenado

```sql

DELIMITER //
--Los delimiters deben hacerce por separado
CREATE PROCEDURE insertar_usuario(
    IN usuario VARCHAR(100),
    IN contrasena VARCHAR(255),
    IN rol_usuario ENUM('admin','vendedor')
)
BEGIN
    DECLARE existe INT;
    SELECT COUNT(*) INTO existe
    FROM usuarios
    WHERE nombre_usuario=usuario;

    IF existe = 0
    THEN
        INSERT INTO usuarios(nombre_usuario,contrasena_hash,rol)
        VALUES (usuario,sha2(contrasena,256),rol);
    ELSE 
        --Asigna un codigo y un mensage de error
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Nombre de Usuario ya existe';
    END IF;
END;
//

DELIMITER ;


```

Y se llama de la siguiente forma:

```sql
CALL insertar_usuario('usuario','Tomate25+','vendedor');
```

<span style="color:GREEN">Esto es Color verde</span>
