# postgresTrabajo
List of queries performed to postgres database

## SECURITY - TRIGGER AND FUNCTIONS
This excercise performs a trigger checking for apellido_persona, nombre_persona and domc_calle to
always be in uppercase. The trigger activates whenever the user tries to update or insert values

```sql
CREATE OR REPLACE FUNCTION controlarDatosPersona()
RETURNS TRIGGER
AS $$
	BEGIN
		NEW.apellido_persona := UPPER(NEW.apellido_persona);
		NEW.nombre_persona := UPPER(NEW.nombre_persona);
		NEW.dom_calle := UPPER(NEW.dom_calle);
		RETURN NEW;
	END;
$$
LANGUAGE 'plpgsql';

CREATE TRIGGER datospersona_enmayuscula
	BEFORE UPDATE OR INSERT
	ON persona
	FOR EACH ROW
	EXECUTE FUNCTION controlarDatosPersona();
	
SELECT * FROM persona WHERE cuit_persona = '20202322321';

UPDATE persona SET dom_calle='por ahí' WHERE cuit_persona = '20202322321';
```

The following query tries to prevent blank spaces to be typed intro certain attributes in the database

```sql
CREATE OR REPLACE FUNCTION quitarEspaciosEnBlanco()
RETURNS TRIGGER
AS $$
	BEGIN
		NEW.cuit_persona := TRIM(NEW.cuit_persona);
		NEW.apellido_persona := TRIM(NEW.apellido_persona);
		NEW.nombre_persona := TRIM(NEW.nombre_persona);
		NEW.dom_calle := TRIM(NEW.dom_calle);
		-- Esto te permite imprimir en consola
		--RAISE NOTICE 'acción performada';
		RETURN NEW;
	END;
$$
LANGUAGE 'plpgsql';

CREATE TRIGGER datos_personaSinEspacio
	BEFORE INSERT OR UPDATE
	ON persona
	FOR EACH ROW
	EXECUTE FUNCTION quitarEspaciosEnBlanco();
	
UPDATE persona SET nombre_persona=' APE' WHERE cuit_persona = '20202322321';
```

The code below is used to audit changes in a database. On INSERT, UPDATE AND DELETE a trigger is activated
which copies the values into a new database that we created. In this new one we also incorporate extra data
as the user in the DB that performed the action, the CURRENT_TIMESTAMP, the ip and the operation (TG_OP)

```sql
CREATE TABLE audi_persona(
	cuit_persona character(11) NOT NULL,
    apellido_persona character varying(50),
    nombre_persona character varying(50),
    sexo_persona character varying(1),
    fecha_nac_persona date,
    dom_calle character varying(40),
    dom_nro_calle smallint,
    dom_cod_localidad smallint,
    dom_cod_provincia smallint,
	id SERIAL PRIMARY KEY,
	tipo_operacion character VARYING(20),
	fec_operacion timestamp,
	usuario character VARYING(50),
	ippc inet
)

CREATE OR REPLACE FUNCTION auditoria_persona()
RETURNS TRIGGER
AS $$
	BEGIN
		IF TG_OP = 'UPDATE' OR TG_OP = 'INSERT' THEN
			INSERT INTO audi_persona (
			cuit_persona, apellido_persona, nombre_persona, sexo_persona, fecha_nac_persona, dom_calle, dom_nro_calle, dom_cod_localidad, dom_cod_provincia, tipo_operacion, fec_operacion, usuario, ippc)
			VALUES (NEW.cuit_persona, NEW.apellido_persona, NEW.nombre_persona, NEW.sexo_persona, NEW.fecha_nac_persona, NEW.dom_calle, NEW.dom_nro_calle, NEW.dom_cod_localidad, NEW.dom_cod_provincia, TG_OP, CURRENT_TIMESTAMP, CURRENT_USER, inet_client_addr());
		ELSE
			INSERT INTO audi_persona (
			cuit_persona, apellido_persona, nombre_persona, sexo_persona, fecha_nac_persona, dom_calle, dom_nro_calle, dom_cod_localidad, dom_cod_provincia, tipo_operacion, fec_operacion, usuario, ippc)
			VALUES (OLD.cuit_persona, OLD.apellido_persona, OLD.nombre_persona, OLD.sexo_persona, OLD.fecha_nac_persona, OLD.dom_calle, OLD.dom_nro_calle, OLD.dom_cod_localidad, OLD.dom_cod_provincia, TG_OP, CURRENT_TIMESTAMP, CURRENT_USER, inet_client_addr());
		END IF;
		RETURN NEW;
	END;
$$
LANGUAGE 'plpgsql';

CREATE TRIGGER auditarPersona
	AFTER UPDATE OR INSERT OR DELETE
	ON persona
	FOR EACH ROW
	EXECUTE FUNCTION auditoria_persona();
	
-- PROBAMOS SI FUNCIONA	
SELECT * FROM persona;

UPDATE persona SET apellido_persona = 'nuevo apellido' WHERE cuit_persona = '20202322321';

SELECT * FROM audi_persona;
```

Creating a functions that concats two attributes of a table. The function will return a table of type text:

````sql
CREATE OR REPLACE FUNCTION buscarpersona2(cuit VARCHAR)
RETURNS TABLE(concatenado text)
AS $$
	BEGIN
		RETURN QUERY
		SELECT CONCAT(p.apellido_persona, p.nombre_persona) FROM persona p WHERE p.cuit_persona = cuit;
	END;
$$
LANGUAGE 'plpgsql'

-- comprobacion
SELECT * FROM persona;
SELECT * FROM buscarpersona2('20202322321');
```
