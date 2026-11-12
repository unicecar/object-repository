# Guía de uso de ObjectRepository en NetBeans

**Público:** estudiantes de segundo semestre de Ingeniería de Sistemas\
**Entorno de trabajo:** Apache NetBeans, proyecto Java con Ant y JDK 21.\
**Versión de la clase:** 2\
**Paquete de la biblioteca:** `co.edu.cecar.objectrepository`\
**Autor:** José David Ríos Pacheco ([LinkedIn](https://www.linkedin.com/in/joseriospacheco/))\
**Contribuidor del proyecto:** Namuel Francisco Solórzano Peralta ([LinkedIn](https://www.linkedin.com/in/namuel-francisco-sol%C3%B3rzano-peralta-a728a240/))\
**Archivo de referencia:** [ObjectRepository.java](src/co/edu/cecar/objectrepository/ObjectRepository.java)\
**Clase compilada (Java 21 o superior):** [object-repository-2.0.0.jar](dist/object-repository-2.0.0.jar)

## Institución

[Corporación Universitaria del Caribe — CECAR](https://cecar.edu.co/)

## Objetivo

El objetivo de `ObjectRepository` es **abstraer la lógica de persistencia para que los estudiantes concentren su esfuerzo en la lógica de negocio y no en implementar el almacenamiento de datos**.

Abstraer la persistencia significa ofrecer métodos como `save()`, `findById()` y `update()` que resuelven los detalles de guardar y recuperar información. Así, puedes dedicarte a las reglas y tareas de tu aplicación: validar un promedio, registrar un estudiante o determinar quién cumple los requisitos de una matrícula.

En esta guía aprenderás a utilizar esos métodos desde la capa de servicio, aplicando el patrón repositorio. Los ejemplos usan la entidad `Estudiante` para practicar consultas, actualizaciones, filtros y paginación mientras desarrollas la lógica de negocio.

Se recomienda conocer clases, objetos, atributos, métodos, constructores, listas, condiciones `if` y ciclos `for`. La guía explica los conceptos nuevos a medida que aparecen.

## 1. ¿Qué hace la clase?

Una lista que existe solo en memoria desaparece cuando se cierra el programa. `ObjectRepository` guarda una colección de objetos en un archivo binario y permite recuperarla después.

La clase se inspira en los repositorios de JPA, Hibernate y Spring Data. Este ejemplo usa archivos locales; no requiere una base de datos.

| Concepto | Explicación |
|---|---|
| Entidad | Objeto con información que queremos guardar, como un estudiante. |
| Repositorio | Clase que ofrece operaciones para guardar y buscar entidades. |
| Identificador o ID | Dato único para encontrar una entidad. Usaremos el código estudiantil. |
| Serialización | Proceso que escribe un objeto Java en un formato que puede guardarse en un archivo. |
| CRUD | Crear, leer, actualizar y eliminar información. |

### El patrón repositorio: organizar antes de programar

**Usa el patrón repositorio en tus prácticas.** Este patrón reúne las operaciones para guardar y consultar objetos en una clase dedicada. Así, cuando necesites registrar un estudiante, podrás llamar a `save()` sin escribir de nuevo las instrucciones para abrir y modificar un archivo.

Organiza la aplicación en cuatro partes. Cada una tiene una responsabilidad:

| Parte | Carpeta o paquete | Responsabilidad | Ejemplo |
|---|---|---|---|
| UI: interfaz de usuario | `ui` | Recibe datos y muestra mensajes o resultados. Puede ser una consola o una ventana. | `App` |
| Entities: entidades | `entities` | Representa los datos y valida los valores propios de cada objeto. | `Estudiante` |
| Servicio | `service` | Resuelve una tarea de la aplicación y aplica sus reglas usando el repositorio. | `EstudianteService` |
| Repositorio | `repository` | Guarda, busca, actualiza y elimina entidades. | `EstudianteRepository` y `ObjectRepository` |

Por ejemplo, para cambiar un promedio:

1. La **UI** recibe el código del estudiante y el nuevo promedio.
2. El **servicio** busca al estudiante, cambia el promedio y solicita guardar el cambio.
3. La **entidad** valida que el promedio esté entre 0 y 5.
4. El **repositorio** escribe la información actualizada en el archivo.
5. La **UI** muestra el resultado de la operación.

```text
UI → Servicio → Repositorio → Archivo
       Las entidades contienen los datos que se intercambian.
```

Las entidades no son un paso adicional después del repositorio: son los objetos con los que trabajan las otras partes. Mantén las lecturas de teclado y los mensajes en la UI. Coloca en el servicio tareas como registrar estudiantes o cambiar su promedio, y deja el acceso al archivo a cargo del repositorio.

Esta organización permite reutilizar el mismo servicio desde una consola o una ventana. Si cambia la forma de almacenar los datos, concentras los cambios de persistencia en el repositorio. Para esta práctica basta con clases sencillas; no necesitas configurar un framework.

## 2. Preparar el proyecto en NetBeans

Trabajaremos desde las ventanas y menús de **NetBeans**. Ten instalado **JDK 21** para el proyecto. Los nombres de los menús se muestran en inglés, como aparecen habitualmente en el IDE.

La carpeta `D:\Desarrollos\Java\object-repository` contiene esta guía, el código fuente y el compilado JAR de la clase. Crearás tu aplicación de práctica con el nombre **PracticaEstudiantes**.

### 2.1. Crear el proyecto

1. Abre NetBeans y selecciona **File → New Project**.
2. Elige **Java with Ant → Java Application** y pulsa **Next**.
3. En **Project Name**, escribe `PracticaEstudiantes`.
4. En **Project Location**, selecciona `D:\Desarrollos\Java`. El proyecto quedará en `D:\Desarrollos\Java\PracticaEstudiantes`.
5. En **Create Main Class**, escribe `co.edu.cecar.app.ui.App` y deja activada esa opción.
6. Pulsa **Finish**. Verás el proyecto en la ventana **Projects** y la clase `App` en el editor.

La clase principal es la que contiene `main()`: desde ella empieza la ejecución. [Referencia: creación de aplicaciones Java en NetBeans](https://netbeans.apache.org/tutorial/main/kb/docs/java/javase-intro/).

### 2.2. Seleccionar Java 21

Haz clic derecho en **PracticaEstudiantes → Properties → Libraries** y selecciona **JDK 21** en **Java Platform**. En **Sources**, elige **21** como formato de código fuente (**Source/Binary Format**).

Si JDK 21 no aparece, agrégalo desde **Tools → Java Platforms → Add Platform**, seleccionando su carpeta de instalación. Luego vuelve a las propiedades del proyecto y selecciónalo. [Referencia: configuración de la plataforma Java](https://docs.oracle.com/javase/tutorial/getStarted/cupojava/netbeans.html).

### 2.3. Crear los paquetes

Un paquete agrupa clases relacionadas. En **Projects**, despliega **Source Packages**. El paquete `co.edu.cecar.app.ui` ya contiene `App`.

Para cada paquete restante, haz clic derecho sobre **Source Packages → New → Java Package**, escribe el nombre y pulsa **Finish**:

- `co.edu.cecar.app.entities`
- `co.edu.cecar.app.service`
- `co.edu.cecar.app.repository`

### 2.4. Elegir cómo agregar ObjectRepository

Puedes usar la clase de dos formas. **Elige una sola opción para tu proyecto de práctica.**

| Opción | Cuándo usarla |
|---|---|
| **A. Agregar el JAR compilado** | Recomendada para empezar a trabajar en la lógica de negocio. |
| **B. Copiar el código fuente** | Para estudiar cómo funciona la persistencia o proponer mejoras a la clase. |

Un **JAR** es un archivo que agrupa clases Java ya compiladas. Este proyecto incluye [object-repository-2.0.0.jar](dist/object-repository-2.0.0.jar), preparado para **Java 21 o superior**, con `ObjectRepository`, sus excepciones, filtros y páginas. Se utiliza como biblioteca desde tu aplicación; no se ejecuta con doble clic.

#### Opción A. Agregar el JAR al proyecto de NetBeans

1. Obtén el archivo [object-repository-2.0.0.jar](dist/object-repository-2.0.0.jar). Si consultas la guía en GitHub, abre ese enlace y usa **Download raw file** para descargarlo. También viene en `dist` si descargaste el repositorio completo.
2. Dentro de la carpeta de tu proyecto **PracticaEstudiantes**, crea una carpeta llamada `lib` usando el Explorador de archivos y copia allí el JAR. Así la biblioteca queda junto a tu proyecto.
3. En NetBeans, haz clic derecho sobre **PracticaEstudiantes → Properties → Libraries**.
4. En **Compile**, selecciona **Classpath** si aparece esa lista, y pulsa **Add JAR/Folder**. Selecciona el JAR que copiaste a `PracticaEstudiantes/lib`.
5. Confirma con **Open** y **OK**. La biblioteca aparecerá bajo **Libraries** en **Projects**.

Ya puedes continuar con las clases de esta guía. **No crees una clase `ObjectRepository.java` en tu proyecto cuando elijas el JAR**: NetBeans encontrará esa clase en la biblioteca. Sí debes crear `EstudianteRepository`, que la extiende.

El paquete de la biblioteca es `co.edu.cecar.objectrepository`. Para utilizar la clase, importa también su nombre:

```java
import co.edu.cecar.objectrepository.ObjectRepository;
```

`co.edu.cecar.objectrepository` es el nombre del paquete; `ObjectRepository` es la clase que vas a utilizar. Por eso debes escribir el import completo que aparece arriba.

Tu clase `EstudianteRepository` sigue en `co.edu.cecar.app.repository` y utiliza la biblioteca mediante ese import. Las entidades, el servicio y la UI mantienen sus paquetes de la aplicación.

**Si ya habías agregado el JAR anterior:** reemplaza el archivo dentro de `PracticaEstudiantes/lib`. Si NetBeans conserva una referencia al anterior, retírala desde **Properties → Libraries** y agrega el JAR actualizado. Cambia los imports al nuevo paquete y ejecuta **Clean and Build** antes de volver a ejecutar `App`.

También están disponibles `ObjectRepository.Page` y sus excepciones. Al seguir esta opción no necesitas agregar dependencias adicionales a las del JDK. [Referencia: bibliotecas y classpath en proyectos de NetBeans](https://docs.oracle.com/netbeans/nb82/netbeans/NBDAG/create_japps.htm).

#### Opción B. Incorporar el código fuente

Crea un paquete llamado `co.edu.cecar.objectrepository` y, dentro de él, una clase llamada **ObjectRepository** mediante **New → Java Class**. Abre la [clase proporcionada](src/co/edu/cecar/objectrepository/ObjectRepository.java), copia todo su contenido y pégalo reemplazando el contenido generado por NetBeans. Guarda el archivo. En esta opción no agregues el JAR de la opción A.

Si modificas el código fuente, el JAR distribuido no cambia automáticamente: para distribuir tus cambios necesitarás generar una nueva compilación.

### 2.5. Crear las clases de tu aplicación

Haz clic derecho sobre cada paquete y selecciona **New → Java Class**. Escribe el nombre de la clase sin `.java` y pulsa **Finish**.

| Paquete | Clases |
|---|---|
| `co.edu.cecar.app.ui` | `App` — creada por el asistente |
| `co.edu.cecar.app.entities` | `Estudiante` |
| `co.edu.cecar.app.service` | `EstudianteService` |
| `co.edu.cecar.app.repository` | `EstudianteRepository` |
| `co.edu.cecar.objectrepository` | `ObjectRepository`, solo si elegiste copiar el código fuente |

Completa las clases con los ejemplos de las siguientes secciones. Cada bloque incluye `package`, los imports y la clase completa: reemplaza el contenido generado por NetBeans para evitar repetir esas partes. Es normal que aparezcan errores de referencias mientras todavía falten clases por completar.

Con la opción del JAR, tu proyecto tendrá esta organización:

```text
PracticaEstudiantes
├── Source Packages
│   ├── co.edu.cecar.app.ui
│   │   └── App.java
│   ├── co.edu.cecar.app.entities
│   │   └── Estudiante.java
│   ├── co.edu.cecar.app.service
│   │   └── EstudianteService.java
│   └── co.edu.cecar.app.repository
│       └── EstudianteRepository.java
└── Libraries
    └── object-repository-2.0.0.jar
```

Si elegiste el código fuente, verás además el paquete `co.edu.cecar.objectrepository` con `ObjectRepository.java`. El repositorio de estudiantes permanece en `co.edu.cecar.app.repository`. NetBeans crea las carpetas correspondientes a cada paquete. Utiliza **Projects** para navegar entre las clases y **Files** para ver los archivos del proyecto.

## 3. Crear la entidad `Estudiante`

En NetBeans, abre **co.edu.cecar.app.entities → Estudiante.java** y reemplaza su contenido por este código:

```java
package co.edu.cecar.app.entities;

import java.io.Serializable;

public class Estudiante implements Serializable {
    private static final long serialVersionUID = 1L;

    private final Integer codigo;
    private final String nombre;
    private final int semestre;
    private double promedio;

    public Estudiante(Integer codigo, String nombre, int semestre, double promedio) {
        if (codigo == null || codigo <= 0) {
            throw new IllegalArgumentException("El codigo debe ser positivo.");
        }
        if (nombre == null || nombre.isBlank()) {
            throw new IllegalArgumentException("El nombre es obligatorio.");
        }
        if (semestre < 1) {
            throw new IllegalArgumentException("El semestre debe ser positivo.");
        }
        this.codigo = codigo;
        this.nombre = nombre;
        this.semestre = semestre;
        setPromedio(promedio);
    }

    public Integer getCodigo() { return codigo; }
    public String getNombre() { return nombre; }
    public int getSemestre() { return semestre; }
    public double getPromedio() { return promedio; }

    public void setPromedio(double promedio) {
        if (!Double.isFinite(promedio) || promedio < 0 || promedio > 5) {
            throw new IllegalArgumentException("El promedio debe estar entre 0 y 5.");
        }
        this.promedio = promedio;
    }

    @Override
    public String toString() {
        return codigo + " | " + nombre + " | semestre " + semestre
                + " | promedio " + promedio;
    }
}
```

La entidad implementa `Serializable`, requisito para escribirla mediante serialización. `serialVersionUID` identifica su versión de serialización. El código es `final` porque identifica al estudiante y no debe cambiar al actualizar el promedio.

Usamos `Integer` para el código porque los parámetros genéricos de Java reciben tipos de referencia como `Integer` y `String`. En esta clase, `T` será `Estudiante` y `ID` será `Integer`.

## 4. Crear el repositorio concreto

Abre **co.edu.cecar.app.repository → EstudianteRepository.java** y reemplaza su contenido por este código:

```java
package co.edu.cecar.app.repository;

import co.edu.cecar.app.entities.Estudiante;
import co.edu.cecar.objectrepository.ObjectRepository;

public class EstudianteRepository extends ObjectRepository<Estudiante, Integer> {
    public EstudianteRepository() {
        this("data/estudiantes.bin");
    }

    public EstudianteRepository(String ruta) {
        super(ruta, Estudiante::getCodigo);
    }
}
```

`extends` indica que `EstudianteRepository` hereda los métodos de `ObjectRepository`. El constructor sin argumentos usa el archivo habitual. El constructor que recibe una ruta permite elegir otro archivo para practicar.

`super(...)` entrega a la clase padre la ruta del archivo y la forma de obtener el ID. `Estudiante::getCodigo` es una referencia a método; aquí equivale a `estudiante -> estudiante.getCodigo()`.

Este repositorio se dedica a almacenar estudiantes. No pide datos por teclado ni imprime mensajes al usuario.

## 5. Conectar el servicio y la UI

### 5.1. Crear el servicio

Abre **co.edu.cecar.app.service → EstudianteService.java** y reemplaza su contenido por este código:

```java
package co.edu.cecar.app.service;

import co.edu.cecar.app.entities.Estudiante;
import co.edu.cecar.app.repository.EstudianteRepository;
import java.util.List;

public class EstudianteService {
    private final EstudianteRepository repo = new EstudianteRepository();

    public void registrarEstudiante(Estudiante estudiante) {
        repo.save(estudiante);
    }

    // Se usa para cargar los datos de ejemplo al iniciar esta práctica.
    public void registrarSiNoExiste(Estudiante estudiante) {
        if (!repo.existsById(estudiante.getCodigo())) {
            registrarEstudiante(estudiante);
        }
    }

    public List<Estudiante> listarEstudiantes() {
        return repo.findAll();
    }

    public long contarEstudiantes() {
        return repo.count();
    }

    public List<Estudiante> buscarDestacadosDeSegundo() {
        return repo.filter(e -> e.getSemestre() == 2)
                .and(e -> e.getPromedio() >= 4.0)
                .get();
    }

    public void cambiarPromedio(Integer codigo, double nuevoPromedio) {
        Estudiante estudiante = repo.getById(codigo);
        estudiante.setPromedio(nuevoPromedio);
        repo.update(estudiante);
    }
}
```

El servicio expresa tareas que entiende la aplicación. En este ejercicio consideramos destacados a los estudiantes de segundo semestre cuyo promedio es al menos 4.0. Esa decisión aparece en `buscarDestacadosDeSegundo()`.

`cambiarPromedio()` reúne tres pasos: buscar, modificar y guardar. La entidad comprueba que el promedio sea válido. Si la búsqueda o el guardado fallan, la excepción llega a la UI, donde se decide qué mensaje mostrar.

### 5.2. Crear la UI de consola

Abre **co.edu.cecar.app.ui → App.java** y reemplaza su contenido por este código:

```java
package co.edu.cecar.app.ui;

import co.edu.cecar.app.entities.Estudiante;
import co.edu.cecar.app.service.EstudianteService;
import co.edu.cecar.objectrepository.ObjectRepository.DuplicateEntityException;
import co.edu.cecar.objectrepository.ObjectRepository.StorageException;

public class App {
    public static void main(String[] args) {
        EstudianteService servicio = new EstudianteService();

        try {
            servicio.registrarSiNoExiste(new Estudiante(1001, "Ana", 2, 4.5));
            servicio.registrarSiNoExiste(new Estudiante(1002, "Luis", 2, 3.2));
            servicio.registrarSiNoExiste(new Estudiante(1003, "Maria", 1, 4.8));
            servicio.registrarSiNoExiste(new Estudiante(1004, "Carlos", 2, 2.9));

            System.out.println("Total: " + servicio.contarEstudiantes());
            for (Estudiante estudiante : servicio.listarEstudiantes()) {
                System.out.println(estudiante);
            }

            System.out.println("Destacados de segundo semestre:");
            for (Estudiante estudiante : servicio.buscarDestacadosDeSegundo()) {
                System.out.println(estudiante.getNombre());
            }
        } catch (DuplicateEntityException ex) {
            System.out.println("El codigo ya esta registrado.");
        } catch (IllegalArgumentException ex) {
            System.out.println("Revisa los datos: " + ex.getMessage());
        } catch (StorageException ex) {
            System.err.println("Error al acceder a los datos: " + ex.getMessage());
        }
    }
}
```

En esta primera UI los datos están escritos en el código. Más adelante puedes leerlos por teclado. Observa que `App` llama al **servicio** para realizar las operaciones; el servicio utiliza el **repositorio**.

### 5.3. Ejecutar en NetBeans

1. Selecciona **File → Save All** para guardar las clases.
2. En **Projects**, haz clic derecho en **PracticaEstudiantes → Run**. Si el menú muestra **Run Project**, elige esa opción.
3. Si NetBeans solicita la clase principal, selecciona `co.edu.cecar.app.ui.App`.
4. Lee los resultados en la ventana **Output** que aparece en la parte inferior. Si no está visible, ábrela desde **Window → Output**.

También puedes abrir `App.java` y usar **Run File** (`Shift + F6`). NetBeans compila y ejecuta el código. [Referencia: ejecutar proyectos y archivos Java](https://docs.oracle.com/netbeans/nb82/netbeans/NBDAG/run_debug_japps.htm).

Con el archivo de estudiantes nuevo, verás:

```text
Total: 4
1001 | Ana | semestre 2 | promedio 4.5
1002 | Luis | semestre 2 | promedio 3.2
1003 | Maria | semestre 1 | promedio 4.8
1004 | Carlos | semestre 2 | promedio 2.9
Destacados de segundo semestre:
Ana
```

Ejecuta otra vez: los cuatro estudiantes se conservan y no se duplican. El servicio usa `registrarSiNoExiste()` para cargar los datos iniciales. Esta comprobación y el guardado son operaciones separadas; por eso la UI también maneja un posible duplicado.

### 5.4. Encontrar los datos guardados

En **Files**, busca la carpeta `data` dentro de **PracticaEstudiantes**. Ahí encontrarás `estudiantes.bin` y `estudiantes.bin.lock` al usar la configuración habitual del proyecto. Conserva el archivo `.lock`, utilizado para coordinar el acceso.

La ruta `data/estudiantes.bin` depende de la carpeta de trabajo. Si los archivos aparecen en otro lugar, revisa **Properties → Run → Working Directory** y establece la carpeta `D:\Desarrollos\Java\PracticaEstudiantes`. Usa esa misma carpeta en las siguientes ejecuciones para consultar los mismos datos.

### 5.5. Si el ejemplo no se ejecuta

| Lo que ves en NetBeans | Qué revisar |
|---|---|
| `No main classes found` | Comprueba que `App` tenga el método `public static void main(String[] args)` completo. |
| `package ... does not exist` | Comprueba los paquetes e imports. Si elegiste el JAR, revisa que esté agregado en Libraries; si elegiste el código fuente, comprueba que copiaste la clase completa. |
| `cannot find symbol` | Revisa el nombre de la clase o método señalado y que su import esté escrito al inicio del archivo. |
| Error de versión de Java | Revisa que el proyecto tenga JDK 21 y formato de código fuente 21 en sus propiedades. |
| Se ejecuta otra clase | En **Properties → Run → Main Class**, selecciona `co.edu.cecar.app.ui.App`. |

Lee el primer error de **Output**, corrígelo y vuelve a ejecutar. Para reconstruir el proyecto desde el IDE, utiliza **Clean and Build** en el menú del proyecto.

## 6. Operaciones CRUD

Las secciones 6 a 9 explican los métodos que puedes utilizar **dentro de los métodos de `EstudianteService`**, donde ya existe el campo `repo`. Son fragmentos para estudiar uno por uno: adapta su resultado con `return` y recíbelo en la UI. Los `System.out.println` de estos fragmentos ilustran qué información se puede mostrar; en tu aplicación colócalos en `App`.

En NetBeans, edita los métodos desde la clase abierta en el editor. Los imports van al principio del archivo Java, después de `package`, nunca dentro de un método. Según el fragmento, necesitarás `java.util.List`, `java.util.Optional`, `java.util.Comparator` y `co.edu.cecar.objectrepository.ObjectRepository.Page`.

Por ejemplo, el servicio puede exponer una tarea para consultar un estudiante:

```java
public Optional<Estudiante> buscarEstudiante(Integer codigo) {
    return repo.findById(codigo);
}
```

Después la UI llama a `servicio.buscarEstudiante(1001)` y muestra el resultado. Conserva esta distribución al implementar las demás tareas. Las operaciones de escritura modifican el archivo de datos.

### Guardar

```java
Estudiante nuevo = new Estudiante(1005, "Sofia", 2, 4.1);
repo.save(nuevo);
```

`save()` inserta un ID nuevo y devuelve `void`. Si el código ya existe, lanza `DuplicateEntityException`. No se usa `save()` para actualizar.

### Listar y contar

```java
List<Estudiante> estudiantes = repo.findAll();
for (Estudiante estudiante : estudiantes) {
    System.out.println(estudiante);
}
System.out.println("Cantidad: " + repo.count());
```

`findAll()` devuelve una lista. Si no hay elementos, la lista está vacía. `getAll()` es otro nombre para obtener todos los registros. `count()` devuelve la cantidad.

### Buscar por ID con Optional

```java
import java.util.Optional;

Optional<Estudiante> resultado = repo.findById(1001);
if (resultado.isPresent()) {
    System.out.println(resultado.get().getNombre());
} else {
    System.out.println("No existe ese codigo.");
}
```

`Optional` representa un resultado que puede estar presente o vacío. Revisa `isPresent()` antes de llamar a `get()` para evitar intentar leer un valor inexistente.

`getById(1001)` devuelve directamente el estudiante, pero lanza `EntityNotFoundException` si el ID no existe.

### Actualizar

```java
Optional<Estudiante> resultado = repo.findById(1002);
if (resultado.isPresent()) {
    Estudiante estudiante = resultado.get();
    estudiante.setPromedio(4.0);
    repo.update(estudiante);
}
```

Cambiar el promedio solo modifica el objeto en memoria. `update()` escribe el cambio en el archivo y requiere que ese código ya exista. Si no existe, lanza `EntityNotFoundException`.

### Eliminar

```java
boolean eliminado = repo.deleteById(1004);
System.out.println("Se elimino: " + eliminado);
```

`deleteById()` devuelve `true` si eliminó el registro o `false` si ese código no existía. `delete(estudiante)` también elimina por ID, pero lanza `EntityNotFoundException` cuando falta el registro.

`deleteAll(lista)` elimina una lista de entidades si todas existen. `deleteAll()` sin argumentos elimina todos los registros; úsalo con cuidado.

## 7. Filtrar información

Una lambda es una forma breve de escribir una acción. Por ejemplo:

```java
e -> e.getPromedio() >= 4.0
```

Se lee «recibe un estudiante `e` y responde si su promedio es al menos 4.0». La respuesta es `true` o `false`. La condición se entrega a `filter()`:

```java
List<Estudiante> conBuenPromedio = repo
        .filter(e -> e.getPromedio() >= 4.0)
        .get();
```

`filter()` construye la búsqueda y `get()` devuelve una lista con los resultados. Con los datos iniciales, son Ana y Maria.

### Combinar condiciones

```java
List<Estudiante> segundoConBuenPromedio = repo
        .filter(e -> e.getSemestre() == 2)
        .and(e -> e.getPromedio() >= 4.0)
        .get();
```

`and()` significa que ambas condiciones deben cumplirse. El resultado inicial es Ana.

```java
List<Estudiante> primerSemestreOCarlos = repo
        .filter(e -> e.getSemestre() == 1)
        .or(e -> e.getNombre().equals("Carlos"))
        .get();
```

`or()` acepta los estudiantes que cumplan al menos una condición. `not()` niega toda la condición acumulada:

```java
List<Estudiante> noSonDeSegundo = repo
        .filter(e -> e.getSemestre() == 2)
        .not()
        .get();
```

`and()`, `or()` y `not()` modifican el filtro actual. Para búsquedas independientes, comienza cada una con un nuevo `repo.filter(...)`. El filtro carga los datos cuando se crea; crea uno nuevo para incluir registros guardados después.

### Elegir el resultado

| Método | Resultado |
|---|---|
| `get()` | Lista de coincidencias. |
| `first()` | Primer resultado como `Optional<T>`. |
| `count()` | Cantidad de coincidencias como `long`. |
| `exists()` | `true` si hay coincidencias. |
| `limit(n)` | Lista de hasta `n` coincidencias. |
| `page(numero, tamaño)` | Una página de coincidencias. |
| `map(funcion)` | Lista transformada, por ejemplo, solo los nombres. |

```java
long cantidadSegundo = repo.filter(e -> e.getSemestre() == 2).count();
boolean hayReprobados = repo.filter(e -> e.getPromedio() < 3.0).exists();
List<String> nombres = repo.filter(e -> e.getSemestre() == 2)
        .map(e -> e.getNombre());
```

`limit(2)` ya devuelve una lista; no se le agrega `.get()`. La condición puede escribirse también como lambda con llaves, pero debe terminar con `return true;` o `return false;`.

## 8. Ordenar y paginar

Ordena por nombre con un `Comparator`:

```java
List<Estudiante> porNombre = repo.findAll(
        Comparator.comparing(Estudiante::getNombre));
```

Ordena por promedio de mayor a menor:

```java
List<Estudiante> porPromedio = repo.findAll(
        Comparator.comparingDouble(Estudiante::getPromedio).reversed());
```

Consulta una página ordenada por código:

```java
import co.edu.cecar.objectrepository.ObjectRepository.Page;

Page<Estudiante> pagina = repo.findAll(
        0, 2, Comparator.comparing(Estudiante::getCodigo));
for (Estudiante estudiante : pagina.getContent()) {
    System.out.println(estudiante);
}
System.out.println("Paginas en total: " + pagina.getTotalPages());
System.out.println("Hay siguiente pagina: " + pagina.hasNext());
```

La página inicial es `0`, la siguiente es `1`, y así sucesivamente. El tamaño debe ser mayor que cero. Con cuatro registros y tamaño `2`, la página `0` muestra los dos primeros códigos y la página `1` los otros dos. Una página fuera del rango devuelve contenido vacío.

También puedes filtrar y después paginar:

```java
Page<Estudiante> paginaFiltrada = repo
        .filter(e -> e.getSemestre() == 2)
        .page(0, 2);
```

El filtro no ordena los resultados automáticamente. La paginación trabaja en memoria con la colección cargada del archivo. `Page` ofrece también `getContent()`, `getTotalElements()`, `getPageNumber()`, `getPageSize()`, `hasPrevious()` e `isEmpty()`.

## 9. Varios registros y archivos de práctica

`saveAll()` inserta un lote y valida todos sus IDs antes de escribir. Usa una ruta separada para esta práctica:

```java
EstudianteRepository practica =
        new EstudianteRepository("data/practica.bin");

List<Estudiante> lote = List.of(
        new Estudiante(2001, "Elena", 2, 4.2),
        new Estudiante(2002, "Pedro", 1, 3.8));
practica.saveAll(lote);
```

`List.of(...)` crea una lista con los elementos indicados. Si algún código ya existe, `saveAll()` rechaza el lote completo con `DuplicateEntityException`.

Para borrar una lista, comprueba que todos los códigos existan:

```java
List<Estudiante> aEliminar = List.of(
        practica.getById(2001), practica.getById(2002));
practica.deleteAll(aEliminar);
```

`deleteAll()` sin lista borra todo el contenido de ese repositorio. Usa esta operación solamente cuando quieras vaciar el archivo.

## 10. Errores que puedes encontrar

Las excepciones propias se importan desde `ObjectRepository`:

```java
import co.edu.cecar.objectrepository.ObjectRepository.DuplicateEntityException;
import co.edu.cecar.objectrepository.ObjectRepository.EntityNotFoundException;
import co.edu.cecar.objectrepository.ObjectRepository.StorageException;
```

Ejemplo de manejo de un ID duplicado en la UI, donde ya existe la variable `servicio`:

```java
try {
    servicio.registrarEstudiante(new Estudiante(1001, "Otro nombre", 2, 3.5));
} catch (DuplicateEntityException ex) {
    System.out.println("Ese codigo ya esta registrado.");
} catch (StorageException ex) {
    System.err.println("Problema al acceder al archivo: " + ex.getMessage());
}
```

| Excepción o resultado | Causa habitual | Qué revisar |
|---|---|---|
| `DuplicateEntityException` | El código ya existe al guardar. | Usa un código nuevo o `update()` para modificar. |
| `EntityNotFoundException` | Se solicitó actualizar o eliminar un código ausente. | Comprueba el ID con `findById()`. |
| `StorageException` | No se pudo leer o escribir, o los datos son inválidos. | Revisa la ruta, los permisos y el archivo. |
| `IllegalArgumentException` | Página, tamaño u otro argumento fuera de rango. | Revisa los valores enviados. |
| `Optional.empty()` | La búsqueda no encontró datos. | Es un resultado normal; decide qué mensaje mostrar. |

No imprimas un mensaje de éxito si ocurrió una excepción. Si un archivo binario está dañado, el repositorio informa el error; no lo edites con un editor de texto.

## 11. Métodos principales

| Método | Qué hace |
|---|---|
| `save(entidad)` | Inserta; rechaza un ID duplicado. |
| `saveAll(lista)` | Inserta un lote validado. |
| `findById(id)` | Busca y devuelve `Optional<T>`. |
| `getById(id)` | Busca y falla si no encuentra la entidad. |
| `existsById(id)` | Comprueba si el ID existe. |
| `findAll()` / `getAll()` | Devuelve todos los objetos. |
| `findAll(comparador)` | Devuelve los objetos ordenados. |
| `findAll(pagina, tamaño)` | Devuelve una página. |
| `count()` | Cuenta todos los registros. |
| `find(condicion)` | Devuelve el primer objeto que cumple la condición. |
| `filter(condicion)` | Comienza una consulta con filtros. |
| `update(entidad)` | Reemplaza un objeto ya guardado con ese ID. |
| `deleteById(id)` | Elimina por ID y devuelve `boolean`. |
| `delete(entidad)` | Elimina por ID o falla si no existe. |
| `deleteAll(lista)` | Elimina un lote cuando todos sus IDs existen. |
| `deleteAll()` | Vacía el repositorio. |

## 12. Buenas prácticas y límites

- Aplica el patrón repositorio: la UI llama al servicio, y el servicio usa el repositorio. Las entidades representan los datos y sus validaciones propias.

- Mantén estable el ID: el repositorio lo usa en las búsquedas y actualizaciones.
- Cada tipo de entidad debe tener su propio archivo de datos.
- Conserva el archivo `.lock`; permite coordinar el acceso concurrente.
- El directorio requiere permisos de escritura incluso para consultar, porque se crea y usa el archivo de bloqueo.
- La lista se carga completa en memoria para filtrar y paginar. La clase es apropiada para aprender y para conjuntos pequeños de datos.
- Las operaciones individuales protegen la integridad del archivo. Varias llamadas consecutivas no forman una transacción; por ejemplo, si guardas datos en dos repositorios y falla la segunda escritura, la primera no se revierte automáticamente.
- Cambiar la estructura de una clase serializable puede afectar la lectura de archivos creados con una versión anterior. Conserva copias de los datos importantes.

## 13. Ejercicios

Prueba cada ejercicio con los cuatro estudiantes originales. Implementa la tarea en el servicio y muestra su resultado desde la UI:

1. Busca el código `1003` y luego `9999` con `findById()`. Muestra el nombre o un mensaje apropiado.
2. Filtra a quienes estén en segundo semestre. Resultado esperado: Ana, Luis y Carlos.
3. Filtra a quienes estén en segundo semestre y tengan promedio entre `3.0` y `4.0`. Resultado esperado: Luis.
4. Usa `map()` para obtener los nombres de estudiantes con promedio de al menos `4.0`. Resultado esperado: Ana y Maria.
5. Ordena por nombre y muestra la página `1` de tamaño `2`.
6. Busca a Carlos, cambia su promedio a `3.5` y llama a `update()`. Ejecuta el programa otra vez para comprobar la persistencia.
7. Intenta guardar otro estudiante con código `1001` y maneja `DuplicateEntityException`.
8. Crea una entidad `Libro` con código, título y páginas. Diseña su repositorio en otro archivo y filtra libros de más de 200 páginas.

## 14. Preguntas de repaso

1. ¿Qué diferencia hay entre `save()` y `update()`?
2. ¿Por qué revisamos `isPresent()` antes de usar `Optional.get()`?
3. ¿Qué representa el número `0` en `findAll(0, 2)`?
4. ¿Qué diferencia hay entre `repo.count()` y `repo.filter(condicion).count()`?
5. ¿Por qué el repositorio no puede guardar correctamente dos estudiantes con el mismo código?

<details>
<summary>Respuestas breves</summary>

1. `save()` inserta un ID nuevo; `update()` reemplaza un ID que ya existe.
2. Porque el `Optional` podría estar vacío.
3. Es el índice de la primera página.
4. El primero cuenta todos los registros y el segundo, solo los que cumplen la condición.
5. El código identifica al estudiante y debe ser único.

</details>
