# Api mid
En esta sección se mostrará como consolidar adecuadamente el Api mid

## Orientado a Funciones
Las **api_mid** están orientados a *interoperar con otras apis "consultar, agrupar, ordenar y exponer la información ya procesada que fue obtenida de las demás apis"*, Por ende, es apropiado **establecer al maximo funciones dedicadas** que luego seran expuesta en los servicios del mid.

## Ejemplo

*Se desea Crear un servicio que ingresando un id, retorna este id y un saludo.*

**Opción 1**

Lo más normal para esta solicitud es crear un servicio y en este crear el saludo

```golang
// Saludo ...
// @Title Saludo
// @Description get Saludo by id
// @Param	id		path 	string	true		"The key for staticblock"
// @Success 200 {object} models
// @Failure 404 not found resource
// @router saludo/:id [get]
func (c *UsuarioController) Saludo() {
	idStr := c.Ctx.Input.Param(":id")
	id, _ := strconv.Atoi(idStr)
	c.Data["json"] = map[string]interface{}{"Code": id, "Body": "hola que hace"}
	c.ServeJSON()
}
```

Pero qué pasa si queremos extender las funcionalidades del saludo, o queremos implementar en otra servicio el saludo. No vale la pena replicar código, para esto se desarrolla una función que se encargue de generar el saludo y al controlador solo lo dejamos como el puente que expone la información.

**Opción 2 (La correcta)**
```golang
// Saludo ...
// @Title Saludo
// @Description get Saludo by id
// @Param	id		path 	string	true		"The key for staticblock"
// @Success 200 {object} models
// @Failure 404 not found resource
// @router saludo/:id [get]
func (c *UsuarioController) Saludo() {
	idStr := c.Ctx.Input.Param(":id")
	id, _ := strconv.Atoi(idStr)
	c.Data["json"] = map[string]interface{}{"Code": id, "Body": GeneradorSaludo()}
	c.ServeJSON()
}

// GeneradorSaludo ...
func GeneradorSaludo() (result string) {
	result = "Hola a todos"
	return
}
```

De esta forma sucede dos cosas:
1. Se puede reutilizar la funcion GeneradorSaludo() en otros servicios
2. Se puede realizar pruebas unitarias a la fucnion GeneradorSaludo()
