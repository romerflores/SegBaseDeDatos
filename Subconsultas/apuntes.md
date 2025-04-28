# CONSULTAS

## CONSULTA ESCALAR

Es un tipo de consulta que devuelve un unico valor

Ejemplo:

```sql
SELECT nombre, usuario
FROM usuarios
WHERE rol_id = (
SELECT id  FROM roles WHERE nombre_rol='Administrador'
);
```

## SUBCONSULTA DERIVADA

Se trata como una tabla temporal

- Seleccionar los usarios con mas de 5 intentos fallidos

Ejemplo, Seleccionar usuarios con mas de 1  intentos fallidos:

```sql
SELECT u.nombre, intentos
FROM (
    SELECT usuario_id, COUNT(*) AS intentos
    FROM logs_acceso
    WHERE exito = 0
    GROUP BY usuario_id
    ) AS fallos
JOIN usuarios u ON u.id = fallos.usuario_id
WHERE intentos > 1;
```

## SUBCONSULTA CORRELACIONADA

Depende de la fila externa, es como un campo

Ejemplo, encontrar la ultima vez que un usuario entro a su cuenta con exito

```sql
SELECT u.nombre, (                     
    SELECT max(fecha_acceso) FROM          
    logs_acceso l where l.usuario_id = u.id
    ) AS ultimo_acceso                     
FROM usuarios AS u;                    
```

## Consultas con EXISTS, y NOT EXISTS

Ejemplo: Optener los usuarios que aun no tienen accesos registrados

```sql
SELECT nombre  FROM usuarios AS u                
WHERE NOT EXISTS (                               
    SELECT 1 FROM logs_acceso WHERE usuario_id = u.id
);                                              
```

## Subconsulta con IN

Comprueba si un valore esta en una lista

Ejemplo Obtener los usuarios con roles que tienen el permiso ver auditoria

```sql
SELECT nombre FROM usuarios                             
WHERE rol_id in ( 
    SELECT rol_id FROM rol_permiso AS rp
    JOIN                                      
    permisos p ON rp.permiso_id = p.id        
    WHERE p.nombre_permiso = "Ver Auditoria"
);
```
