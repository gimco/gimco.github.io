---
layout: post
title: Multientidad con Hibernate 3
---

En una antigua aplicación se presentó la necesidad de agregar un nuevo conjunto de usuarios de forma que pudieran administrar sus datos y procedimientos de forma independiente al conjunto actual de usuarios. Estaríamos ante el clásico problema de la multientidad (multi-tenancy). 

El término multientidad es una arquitectura en la que una única instancia de la aplicación es capaz de dar servicio a varios clientes (entidades), muy común en sistemas SaaS ([Software as a Service][saas]). El problema que se intenta resolver es el aislamiento de la información entre las distintas entidades.

En un sistema con soporte multientidad podemos crear distintos conjunto de datos independientes que serían nuestra entidades. Cada entidad tendrá su propio conjunto de información, usuarios, roles y administradores independientes de las otras entidades. 

## Tipo de multientidad

La aplicación en cuestión no fue creada con esta capacidad desde el principio, por podríamos abordar este nuevo requisito de distintas formas:

- *Realizar un nuevo despliegue.* ¡Esto no dotaría a la aplicación de la capacidad de ser multientidad!, pero resolvería el problema forma puntual. Un nuevo despliegue no requeriría la modificación de la aplicación actual, ya que sólo tendríamos que crear un nuevo entorno (con una url diferente) y una nueva base de datos. Esta no es una opción escalable, puesto que conforme aumenten el número de entidades desperdiciamos recursos y aumentamos la complejidad del mantenimiento.
- *Usar una base de datos distinta por entidad.* En este caso modificaríamos la aplicación para que sea capaz de decidir qué conexión de base de datos debe utilizar para cada petición solicitada.
- *Usar un esquema de base de datos por entidad.* Este caso es similar al anterior pero sólo utilizamos una única base de datos (y un pool de conexiones común), y lo que hacemos es establecer el usuario o el esquema según cada entidad (utilizando `CONNECT` o `ALTER SESSION SET CURRENT_SCHEMA`).
- *Multientidad lógica.* En este caso se utiliza una única base de datos y esquema, y agregamos un campo discriminador en cada una de las tablas que necesitemos independizar por cada entidad. Es decir, agregamos una nueva columna entidad a ciertas tablas que nos indica a que entidad corresponde cada fila de datos.

![](/assets/multientidad-tipos.png)

Las opciones mas populares son las dos últimas, y la utilización de una u otra lo marcará las necesidades concretas. Por ejemplo si estamos creando una plataforma SaaS y queremos permitir que los usuarios puedan crear de forma autónoma nuevas entidades como parte del proceso de registro, probablemente usemos la última opción.

Si por el contrario, la creación de nuevas entidades no forma parte de la lógica principal, y se realiza de forma esporádica (mediante intervención del equipo de sistemas o DBAs) optaremos por la opción de un esquema por entidad. Esta opción además tiene la ventaja de simplificar las copias de seguridad y restauración de los datos de la entidad.

## Impacto

En el caso concreto de esta antigua aplicación web se decidió utilizar la opción del campo discriminador. La librería de persistencia utilizada era Hibernate, la cuál tiene distintos tipos de [soporte multientidad][hibernate-multi] a partir de la [versión 4.2][discriminador]. Desgraciadamente la versión utilizada era la 3.3 y el intento de actualización de las librerías desencadenaba la subida de versiones de otros módulos que tendría un gran impacto en el código existente (dependency hell). Por lo que se decidió implementar la multientidad con la versión de Hibernate utilizada.

El primer paso sería agregar el campo discriminador `entidad` en cada una de la tablas que deban tener subconjuntos de datos independientes. Posteriormente tendríamos que modificar todas y cada una de las consultas HQL de la aplicación para agregar una nueva condición de la entidad, y cada uno de los métodos para agregar el nuevo parámetro con el valor de la entidad que queremos consultar o modificar los datos. Por toda la aplicación estaríamos arrastrando el parámetro de la entidad activa para filtrar los datos adecuadamente. Haciendo esto estaríamos contaminando nuestro modelo para implementar un aspecto que debería ser transversal.

## Implementación

Lo primero que haremos es agregar las columnas que harán de discriminador en las tablas. Estas nuevas columnas pueden ser claves ajenas hacia una tabla donde almacenemos información adicional de la Entidad. Si alguna de las tablas tuviera claves únicas (por ejemplo alguna columna que fuera un código), tendríamos que modificar la condición del índice de unicidad para agregar la columna entidad. No es el caso de las claves primarias (y por tanto únicas) que suelen rellenarse a partir de un objeto secuencia.

![](/assets/multientidad.png)

Las columnas entidad solo serían necesarias en las tablas de primer nivel. Las tablas de primer nivel son aquella que no dependen de otras. Por ejemplo, en una aplicación que gestione facturas en la que todos los usuarios (de la misma entidad) puedan consultarlas indistintamente, la tabla `Facturas` serían un caso de tabla de primer nivel. Las tablas que almacenan el detalle de estas facturas (las líneas de facturas por ejemplo) no necesitaría un campo discriminador ya que siempre se obtendrían a partir de su tabla padre la cuál ya tendría asociada la entidad a la que pertenece. Es decir, la pertenencia del dato a la entidad se resuelve de forma transitiva a través de las las tablas a las que apuntan las claves ajenas.

Para implementar el soporte multientidad de forma transparente aprovecharemos la funcionalidad de [definición de filtros de Hibernate][filters] que nos permite activar o desactivar ciertas condiciones en la sesión de Hibernate para filtrar las consultas de datos. Después de realizar las modificaciones en el esquema de base de datos, crearemos un interfaz con los métodos getter y setter para este nuevo atributo discriminador. Cada clase de dominio de primer nivel que queramos independizar por entidad deberá implementar este interfaz.

~~~java
//src/main/java/com/foo/bar/domain/FiltrarPorEntidad.java
public interface FiltrarPorEntidad {
    public static String FILTRO = "FILTRAR_POR_ENTIDAD";
    public Integer getEntidad();
    public void setEntidad(Integer entidad);
}
~~~

Lo siguiente será definir la condición y los parámetros del filtro que utilizaremos para filtrar los datos, tan simple como agregar la condición `entidad = :entidad`. El filtro lo definimos a nivel de paquete para tenerlo disponible de forma global para todas las clases de dominio.

~~~java
//src/main/java/com/foo/bar/domain/package-info.java
@FilterDef(defaultCondition = "entidad = :entidad",
           name = FiltrarPorEntidad.FILTRO,
           parameters = @ParamDef(name = "entidad", type = "int"))
package com.foo.bar.domain;

import org.hibernate.annotations.FilterDef;
import org.hibernate.annotations.ParamDef;
~~~

Ahora tendremos que modificar cada una de las clases de dominio, indicando la disponibilidad del filtro por entidad, agregando el interfaz y el nuevo atributo:

~~~diff
 @Entity
 @Table(name="Nodo")
+@Filter(name=FiltrarPorEntidad.FILTRO)
-public class Nodo extends BaseEntidad
+public class Nodo extends BaseEntidad implements FiltrarPorEntidad 
+    private Integer entidad;
+
+    @Column (name = "ENTIDAD", nullable = false)
+    public Integer getEntidad() {
+        return entidad;
+    }
+
+    public void setEntidad(Integer entidad) {
+        this.entidad = entidad;
+    }
     ...
 }
~~~

## Filtrando datos en las consultas

Ya tendríamos marcadas todas las entidades que necesitamos filtrar. Ahora tendríamos que activar el filtro en cada consulta que realicemos. Es habitual que nuestras clases de servicio o DAOs hereden de una clase base que agregue cierta lógica común. Aprovecharemos esta clase para activar el filtro cada vez que obtengamos el objeto sesión de Hibernate con la que ejecutamos las sentencias HQL: 

~~~java
public class BaseDO {
    ...

    public Session getSession() {
        Session session = sessionFactory.getCurrentSession();
        session.enableFilter(FiltrarPorEntidad.FILTRO)
            .setParameter("entidad", getEntidadActiva());
        return session;
    }

    protected Query getNamedQueryPorEntidad(String queryName) {
        Session session = sessionFactory.getCurrentSession();
        Query query = session.getNamedQuery(queryName);
        query.setParameter("entidad", getEntidadActiva());
        return query;

    }

    static Integer getEntidadActiva () {
        // obtener la entidad activa actualmente
        // usando alguna clase utilidad o usando ThreadLocal 
    }

    ...
}
~~~

Una parte interesate es cómo saber qué entidad es la activa en cada momento. Esto dependerá de la arquitectura de cada proyecto. Lo mas habitual es crear algún tipo de Filter o Interceptor que se ejecute al principio de cada petición y que averigüe la entidad, ya sea viendo el dominio, url, algún parámetro de sesión o extrayéndolo del usuario autenticado. Una vez averiguado estableceremos el valor de la entidad activa en alguna clase utilidad u objeto ThreadLocal que podamos consultar desde `BaseDAO.getEntidadActiva()`.

Hasta aquí habríamos conseguido que de forma transparente cada vez que en una consulta aparezca alguna clase de dominio se agregue el filtro por entidad de forma automática.

## Entidad en nuevos objetos

Ya tendríamos las consulta de datos filtrada pero nos quedaría la última parte, que sería establecer la entidad en los objetos de nueva creación. No queremos que los desarrolladores deban acordarse de llamar a los métodos `setEntidad` cada vez que vayan a persistir un nuevo objeto. Utilizaremos un listener de Hibernate para las operaciones de guardar o actualizar datos que registraremos en el framework de Hibernate mediante la creación de una clase de integración. Comenzamos por crear un fichero `META-INF/services/org.hibernate.integrator.spi.Integrator` que contendrá el paquete y clase que implementará esta integración:

~~~
//src/main/resources/META-INF/services/org.hibernate.integrator.spi.Integrator
com.foo.bar.domain.MultiEntidadIntegrator
~~~

La clase integración registrará nuestro listener que establecerá el campo entidad en los nuevos objetos que queramos persistir. Ademas, en el caso de las actualizaciones, comprobaremos que estamos modificando solamente objetos de la entidad activa, evitando por tanto que por error modifiquemos objetos que no pertenecen a la entidad del usuario que realiza la operación.

~~~java
public class MultiEntidadIntegrator implements Integrator {

    @Override
    public void integrate(Configuration configuration,
                          SessionFactoryImplementor sessionFactory,
                          SessionFactoryServiceRegistry serviceRegistry) {

        final EventListenerRegistry eventListenerRegistry = serviceRegistry
            .getService(EventListenerRegistry.class);
        MultiEntidadListener el = new MultiEntidadListener();
        eventListenerRegistry.prependListeners(EventType.SAVE_UPDATE, el);
    }

    @Override
    public void integrate(MetadataImplementor metadata,
                          SessionFactoryImplementor sessionFactory,
                          SessionFactoryServiceRegistry serviceRegistry) {
    }

    @Override
    public void disintegrate(SessionFactoryImplementor sessionFactory,
                             SessionFactoryServiceRegistry serviceRegistry) {
    }
}

class MultiEntidadListener implements SaveOrUpdateEventListener {

    private static final long serialVersionUID = 1L;

    @Override
    public void onSaveOrUpdate(SaveOrUpdateEvent event) throws HibernateException {
        Object entity = event.getObject();
        if (entity instanceof FiltrarPorEntidad) {
            FiltrarPorEntidad e = (FiltrarPorEntidad) entity;
            Integer entidadActiva = BaseDAO.getEntidadActiva();
            if (e.getEntidad() == null) {
                e.setEntidad(entidadActiva);
            } else if (e.getEntidad() != entidadActiva) {
                String mensaje = String.format(
                    "Intento de cambiar un objeto %s de la entidad %d a la %d",
                    entity.getClass().getName(),
                    e.getEntidad(),
                    entidadActiva);
                throw new RuntimeException(mensaje);
            }
        }
    }
}
~~~

## Conclusión

Con esta técnica hemos conseguido agregar el soporte multientidad a una aplicación existente para dar servicio a nuevos clientes minimizando el impacto necesario para llevarlo a cabo. También deberemos agregar una gestión mínima de entidades accesible por un perfil superadministrador que nos permita la creación de nuevas entidades.

Existen otros enfoques que también podrían haber sido interesantes de abordar como una solución PL/SQL completamente, mediante el uso de vistas de las tablas que filtren los datos según algún parámetro de sesión de base de datos que pudiéramos establecer (`SYS_CONTEXT`).

Y por último algunos enlaces de interés:

* <http://docs.oracle.com/cd/B19306_01/server.102/b14200/functions165.htm>
* <https://stackoverflow.com/questions/25855260/global-hibernate-filter-on-all-database-queries>
* <https://www.mkyong.com/hibernate/hibernate-data-filter-example-xml-and-annotation/>
* <https://picodotdev.github.io/blog-bitix/2015/02/ejemplo-de-listener-de-eventos-de-hibernate/>
* <https://stackoverflow.com/questions/5150660/adding-listener-to-hibernate-session>

[saas]: https://en.wikipedia.org/wiki/Software_as_a_service
[filters]: https://docs.jboss.org/hibernate/orm/3.3/reference/en/html/filters.html
[discriminador]: https://hibernate.atlassian.net/browse/HHH-6054
[hibernate-multi]: https://docs.jboss.org/hibernate/orm/4.2/devguide/en-US/html/ch16.html