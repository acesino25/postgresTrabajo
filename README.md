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
