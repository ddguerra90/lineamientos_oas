# Refactorizar API CRUD

En está sección se especificarán los ajustes pertinentes para que las API creadas en el framewor Beego respondan en JSON conforme a los estandares de la Oficina Asesora de Sistemas; esto con el fin de que no genere problemas al intregarse con el administrador de servicios WSO2.

>#### [:book: Documento de Sustentación :book:](https://docs.google.com/document/d/1wxf8QB-qZ3c5H2irR6kV6SoVQMw5LNNkp3aFLo9nooI/edit?usp=sharing)

## Requerimientos Previos
1. [Generar API Beego](generar_api.md)


## Procedimiento

### 1 Configurar paquete utils_oas

#### 1.1 Agregar las plantillas de errores [utils_oas](https://github.com/udistrital/utils_oas)

##### 1.1.1 Importar paquete:
Para esto Editar el `main.go` de la API a Ajustar.
```golang
import (
  "github.com/udistrital/utils_oas/customerror"
)
```
##### 1.1.2 Implementación en `func main()`
```golang
beego.ErrorController(&customerror.CustomErrorController{})
```

##### El **main.go** Lucirá de la siguiente forma:
```golang
package main

import (
    "github.com/astaxie/beego"
    "github.com/astaxie/beego/orm"
    "github.com/astaxie/beego/plugins/cors"
    _ "github.com/jotavargas/debug_beego_request/routers"
    _ "github.com/lib/pq"
    "github.com/udistrital/utils_oas/customerror"
)

func init() {
    orm.RegisterDataBase("default", "postgres", "postgres://postgres:postgres@127.0.0.1/test?sslmode=disable")
}

func main() {
    if beego.BConfig.RunMode == "dev" {
        beego.BConfig.WebConfig.DirectoryIndex = true
        beego.BConfig.WebConfig.StaticDir["/swagger"] = "swagger"
    }

    beego.InsertFilter("*", beego.BeforeRouter, cors.Allow(&cors.Options{
        AllowOrigins: []string{"*"},
        AllowMethods: []string{"PUT", "PATCH", "GET", "POST", "OPTIONS", "DELETE"},
        AllowHeaders: []string{"Origin", "x-requested-with",
            "content-type",
            "accept",
            "origin",
            "authorization",
            "x-csrftoken"},
        ExposeHeaders:    []string{"Content-Length"},
        AllowCredentials: true,
    }))
    beego.ErrorController(&customerror.CustomErrorController{})
    beego.Run()
}
```

### 2 Ejecución de Script para Refactorizar los Controladores
Se desarrolló un script en python 2.7 para realizar los ajustes de los micro servicios desarrolladon en el framework Beego de forma masiva y automática.   
la única restricción que existe, es que **solo realiza los ajustes en micro servicios que nos se han personalizado o modificado en sus líneas**. [refactor_controller](https://github.com/udistrital/refactor_controller)

#### 2.1 Implementar el [refactor_controller](https://github.com/udistrital/refactor_controller)

##### 2.1.1  Clonar repositorio
```golang
git clone https://github.com/jotavargas/refactor_controller.git
```
##### 2.1.2 Ejecución del script
```bash
#ir al proyecto
cd refactor_controller

#como ejecutar (con python 2.7)
python2.7 main.py -F ruta_controladores_del_api_a_refactoring

#Ejemplo:
python2.7 main.py -F /home/jjvargass/go/src/github.com/udistrital/api_financiera/controllers
```
##### 2.1.3 Indentar e importar package en Controladores *.go
```bash
cd ruta_controladores_del_api_a_refactoring
gofmt -w *.go
goimports -w *.go
```

### 3 Conparación de cambios (Antes y Despues)
Los cambios específicos en cada uno de los microservicios se definirán a continuación.

#### 3.1 Solicitud POST
A la izquierda el método por defecto creados por el Framewrok.   
A la derecha el refactor por el Script [refactor_controller](https://github.com/udistrital/refactor_controller)

![Refactor Metodo Post](/generacion_de_apis/img/post.png)


#### 3.2 Solicitud GETONE
A la izquierda el método por defecto creados por el Framewrok.   
A la derecha el refactor por el Script [refactor_controller](https://github.com/udistrital/refactor_controller)

![Refactor Metodo GetOne](/generacion_de_apis/img/getone.png)


#### 3.3 Solicitud GETALL
A la izquierda el método por defecto creados por el Framewrok.   
A la derecha el refactor por el Script [refactor_controller](https://github.com/udistrital/refactor_controller)

![Refactor Metodo GetAll](/generacion_de_apis/img/getall1.png)

![Refactor Metodo GetAll](/generacion_de_apis/img/getall2.png)

#### 3.4 Solicitud PUT
A la izquierda el método por defecto creados por el Framewrok.   
A la derecha el refactor por el Script [refactor_controller](https://github.com/udistrital/refactor_controller)

![Refactor Metodo Post](/generacion_de_apis/img/put.png)

#### 3.5 Solicitud DELETE
A la izquierda el método por defecto creados por el Framewrok.   
A la derecha el refactor por el Script [refactor_controller](https://github.com/udistrital/refactor_controller)

![Refactor Metodo Post](/generacion_de_apis/img/delete.png)

### 4 Estructura de Respuesas JSON
Obtenemos la siguiente estructura cuando el framework a controlado un error de bd

![Refactor Metodo Post](/generacion_de_apis/img/json01.png)

Obtenemos este Json cuando es desarrollador ha personalizado el servicio y estructura del error en el atributo development

![Refactor Metodo Post](/generacion_de_apis/img/json02.png)

Obtenemos este Json cuando ingresamos a una servicio  que no existe

![Refactor Metodo Post](/generacion_de_apis/img/json03.png)


## Testing con JMeter

![Refactor Metodo GetAll](/generacion_de_apis/img/test_01.png)

#### [Link Testing Jmeter](/generacion_de_apis/src/beegoTodasLasSolicitudes.jmx)
