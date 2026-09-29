# Laboratorio: arregla el código (SOLID)

## Lenguaje elegido: Java

Se usó Java porque los ejercicios están planteados en ese lenguaje y permite representar claramente interfaces, herencia y dependencias.

En cada ejercicio se identifica el problema, se muestra el código original del ejemplo en Java, se propone una corrección siguiendo el principio SOLID correspondiente, y se presenta una evidencia de ejecución.

---

## Ejercicio S — Single Responsibility Principle (SRP)

### Preguntas Guías
#### ¿Qué hace esta clase en una sola frase?

La clase representa un estudiante, calcula sus notas, guarda sus datos, imprime su boletín y envía información al acudiente.

#### ¿Cuantas "y" tiene?

Tiene 5 "y"

#### Si el colegio cambia el formato del boletín, ¿qué clase se toca?

En el código original se debe modificar `Estudiante`, aunque el cambio solo corresponde a la presentación del boletín. En la solución corregida se modifica únicamente `BoletinService`.

#### ¿Y si cambian el archivo por una base de datos?

En el código original también se modifica `Estudiante`. En la solución corregida se cambia o reemplaza `EstudianteRepository`, sin tocar la entidad ni el cálculo del promedio.

### Problema identificado
La clase `Estudiante` hace varias cosas a la vez:
- representa datos del estudiante,
- calcula promedios,
- guarda información en archivo,
- imprime boletín,
- decide enviar correo.

Esto viola SRP porque tiene más de una razón para cambiar: si cambia el formato del boletín, si cambia el almacenamiento o si cambia la forma de envío, la misma clase se modifica.

### Código original
```java
public class Estudiante {
    private String nombre;
    private double[] notas;

    public Estudiante(String nombre, double[] notas) {
        this.nombre = nombre;
        this.notas = notas;
    }

    public double calcularPromedio() {
        double suma = 0;
        for (double n : notas) suma += n;
        return suma / notas.length;
    }

    public void guardarEnArchivo() {
        System.out.println("Guardando " + nombre + " en estudiantes.txt...");
    }

    public void imprimirBoletin() {
        System.out.println("=== BOLETÍN ===");
        System.out.println("Nombre: " + nombre);
        System.out.println("Promedio: " + calcularPromedio());
    }

    public void enviarCorreoAlAcudiente() {
        System.out.println("Enviando boletín por correo al acudiente de " + nombre);
    }
}
```

### Código corregido
```java
import java.util.Arrays;

class Estudiante {
    private final String nombre;
    private final double[] notas;

    public Estudiante(String nombre, double[] notas) {
        this.nombre = nombre;
        this.notas = Arrays.copyOf(notas, notas.length);
    }

    public String getNombre() {
        return nombre;
    }

    public double[] getNotas() {
        return Arrays.copyOf(notas, notas.length);
    }

    public double calcularPromedio() {
        double suma = 0;
        for (double n : notas) suma += n;
        return notas.length == 0 ? 0 : suma / notas.length;
    }
}

class BoletinService {
    public void imprimirBoletin(Estudiante estudiante) {
        System.out.println("=== BOLETÍN ===");
        System.out.println("Nombre: " + estudiante.getNombre());
        System.out.println("Promedio: " + estudiante.calcularPromedio());
    }
}

class EstudianteRepository {
    public void guardar(Estudiante estudiante) {
        System.out.println("Guardando " + estudiante.getNombre() + " en estudiantes.txt...");
    }
}

class AcudienteService {
    public void enviarCorreoAlAcudiente(Estudiante estudiante) {
        System.out.println("Enviando boletín por correo al acudiente de " + estudiante.getNombre());
    }
}

public class DemoSRP {
    public static void main(String[] args) {
        Estudiante e = new Estudiante("Ana", new double[]{4.5, 4.8, 4.2});

        BoletinService boletinService = new BoletinService();
        EstudianteRepository repository = new EstudianteRepository();
        AcudienteService acudienteService = new AcudienteService();

        boletinService.imprimirBoletin(e);
        repository.guardar(e);
        acudienteService.enviarCorreoAlAcudiente(e);
    }
}
```


### Evidencia
```text
=== BOLETÍN ===
Nombre: Ana
Promedio: 4.5
Guardando Ana en estudiantes.txt...
Enviando boletín por correo al acudiente de Ana
```
---

## Ejercicio O — Open/Closed Principle (OCP)


### Problema identificado

`CalculadoraEnvio` contiene una cadena de condiciones `if/else`. Cada nuevo tipo de envío obliga a modificar la clase. Si existen muchos tipos, la clase se vuelve larga, difícil de probar y propega errores.

### Preguntas guía

#### La empresa quiere agregar el envío `MISMO_DIA`. ¿que modifico?

En el código original deben modificar `CalculadoraEnvio` y agregar otro bloque `else if`. Esto no va con OCP porque una nueva funcionalidad requiere modificar código existente.

En la solución corregida se crea una nueva clase `EnvioMismoDia` y se registra mediante `registrar`, sin modificar la lógica de cálculo de `CalculadoraEnvio`.

#### ¿Qué pasa con esta clase si en un año hay 15 tipos de envío?

La cadena de condiciones crecería demasiado. La clase tendría muchas responsabilidades, sería difícil de leer y cualquier modificación podría afectar los tipos existentes.

### Código corregido

```java
import java.util.HashMap;
import java.util.Map;

interface EstrategiaEnvio {
    double calcular(double peso);
}

class EnvioNormal implements EstrategiaEnvio {
    public double calcular(double peso) {
        return peso * 2000;
    }
}

class EnvioExpress implements EstrategiaEnvio {
    public double calcular(double peso) {
        return peso * 5000 + 10000;
    }
}

class EnvioInternacional implements EstrategiaEnvio {
    public double calcular(double peso) {
        return peso * 15000 + 50000;
    }
}

// Nueva extensión: no es necesario modificar CalculadoraEnvio.
class EnvioMismoDia implements EstrategiaEnvio {
    public double calcular(double peso) {
        return peso * 8000 + 20000;
    }
}

class CalculadoraEnvio {
    private final Map<String, EstrategiaEnvio> estrategias = new HashMap<>();

    public CalculadoraEnvio() {
        estrategias.put("NORMAL", new EnvioNormal());
        estrategias.put("EXPRESS", new EnvioExpress());
        estrategias.put("INTERNACIONAL", new EnvioInternacional());
    }

    public void registrar(String tipo, EstrategiaEnvio estrategia) {
        estrategias.put(tipo, estrategia);
    }

    public double calcular(String tipo, double peso) {
        EstrategiaEnvio estrategia = estrategias.get(tipo);
        if (estrategia == null) {
            throw new IllegalArgumentException("Tipo de envío no soportado");
        }
        return estrategia.calcular(peso);
    }
}

public class DemoOCP {
    public static void main(String[] args) {
        CalculadoraEnvio calculadora = new CalculadoraEnvio();
        calculadora.registrar("MISMO_DIA", new EnvioMismoDia());

        System.out.println(calculadora.calcular("NORMAL", 3));
        System.out.println(calculadora.calcular("EXPRESS", 3));
        System.out.println(calculadora.calcular("MISMO_DIA", 3));
    }
}
```

### Reto extra

Para un peso de `3`, el envío `MISMO_DIA` cuesta:

```text
3 * 8000 + 20000 = 44000
```

### Justificación

Cada tipo de envío se representa mediante una manera independiente. Agregar una modalidad nueva lleva en crear otra implementación y registrarla, sin modificar la clase que coordina los cálculos.

### Evidencia

```text
6000.0
25000.0
44000.0
```
---

## Ejercicio L — Liskov Substitution Principle (LSP)


### Problema identificado

`ArchivoSoloLectura` hereda de `Archivo`, pero no puede cumplir correctamente el método `escribir`: lanza una excepción. Por eso, un objeto que parece ser un `Archivo` no puede utilizarse en todos los lugares donde se espera un archivo escribible.

### Preguntas guía

#### ¿Qué pasa al ejecutar `agregarFirma` con un `Archivo` y un `ArchivoSoloLectura`?

El método funciona con el primer archivo escribible, pero cuando llega al archivo de solo lectura lanza `UnsupportedOperationException`. La operación se interrumpe y el programa falla.

#### ¿Por qué agregar `if (!(a instanceof ArchivoSoloLectura))` es un parche?

Porque el cliente debe conocer una clase concreta y preguntar qué tipo de objeto recibió. Eso aumenta el acoplamiento y no corrige el contrato incorrecto. Además, habría que agregar más condiciones cada vez que aparezca otro tipo de archivo no escribible.

#### ¿Un archivo de solo lectura realmente “es un” archivo que se puede escribir?

No. Es un archivo que se puede leer, pero no cumple el contrato de escritura. Por eso no debe heredar de una abstracción que obligue a escribir.

### Código corregido

```java
import java.util.ArrayList;
import java.util.List;

interface Archivo {
    String leer();
}

interface ArchivoEscribible extends Archivo {
    void escribir(String texto);
}

class ArchivoNormal implements ArchivoEscribible {
    private String contenido = "";

    public String leer() {
        return contenido;
    }

    public void escribir(String texto) {
        contenido += texto;
    }
}

class ArchivoSoloLectura implements Archivo {
    private final String contenido;

    public ArchivoSoloLectura(String contenido) {
        this.contenido = contenido;
    }

    public String leer() {
        return contenido;
    }
}

class Editor {
    public void agregarFirma(List<ArchivoEscribible> archivos) {
        for (ArchivoEscribible archivo : archivos) {
            archivo.escribir("\n-- Firmado por el sistema");
        }
    }
}

public class DemoLSP {
    public static void main(String[] args) {
        ArchivoNormal archivo1 = new ArchivoNormal();
        ArchivoNormal archivo2 = new ArchivoNormal();

        List<ArchivoEscribible> archivos = new ArrayList<>();
        archivos.add(archivo1);
        archivos.add(archivo2);

        new Editor().agregarFirma(archivos);

        System.out.println(archivo1.leer());
        System.out.println(archivo2.leer());
    }
}
```

### Justificación

Se separan los contratos de lectura y escritura. `Editor` recibe exclusivamente archivos que garantizan la operación `escribir`, mientras que `ArchivoSoloLectura` solo implementa el contrato que realmente puede cumplir.

### Evidencia

```text
-- Firmado por el sistema
-- Firmado por el sistema
```
---

## Ejercicio I — Interface Segregation Principle (ISP)

### Problema identificado

La interfaz `Dispositivo` es demasiado grande. Una impresora básica está obligada a implementar escaneo, fax y fotocopia, aunque no puede realizar esas operaciones. Los métodos vacíos ocultan el error al cliente.

### Preguntas guía

#### Si alguien llama `impresoraBasica.escanear("contrato")`, ¿qué pasa?

En el código original el método no hace nada y tampoco informa que la operación no está disponible. El cliente puede creer que el documento fue escaneado, aunque realmente no ocurrió nada.

#### Si se agrega `enviarPorCorreo` a la interfaz, ¿cuántas clases hay que modificar?

En el diseño original habría que modificar todas las clases que implementan `Dispositivo`, incluso las que no tienen esa capacidad. Esto demuestra que la interfaz es demasiado amplia.

### Código corregido

```java
interface Impresora {
    void imprimir(String documento);
}

interface Escaner {
    void escanear(String documento);
}

interface Fax {
    void enviarFax(String documento);
}

interface Fotocopiadora {
    void fotocopiar(String documento);
}

class ImpresoraMultifuncional
        implements Impresora, Escaner, Fax, Fotocopiadora {

    public void imprimir(String documento) {
        System.out.println("Imprimiendo " + documento);
    }

    public void escanear(String documento) {
        System.out.println("Escaneando " + documento);
    }

    public void enviarFax(String documento) {
        System.out.println("Enviando fax " + documento);
    }

    public void fotocopiar(String documento) {
        System.out.println("Fotocopiando " + documento);
    }
}

class ImpresoraBasica implements Impresora {
    public void imprimir(String documento) {
        System.out.println("Imprimiendo " + documento);
    }
}

// Reto extra: dispositivo que solo escanea.
class EscanerSimple implements Escaner {
    public void escanear(String documento) {
        System.out.println("Escaneando " + documento);
    }
}

public class DemoISP {
    public static void main(String[] args) {
        Impresora impresora = new ImpresoraBasica();
        Escaner escaner = new EscanerSimple();

        impresora.imprimir("contrato.pdf");
        escaner.escanear("contrato.pdf");
    }
}
```

### Justificación

Las capacidades se dividen en interfaces pequeñas. Cada clase implementa solamente las operaciones que puede ofrecer, evitando métodos vacíos y haciendo que el contrato sea claro para el cliente.

### Evidencia

```text
Imprimiendo contrato.pdf
Escaneando contrato.pdf
```
---

## Ejercicio D — Dependency Inversion Principle (DIP)

> Los módulos de alto nivel no deben depender de módulos de bajo nivel; ambos deben depender de abstracciones.

### Problema identificado

`ServicioUsuarios` crea directamente un objeto `MySQLDatabase`. La lógica de negocio queda acoplada a una tecnología específica, lo que dificulta migrar a MongoDB y probar el servicio sin una base de datos real.

### Preguntas guía

#### La empresa decide migrar a MongoDB. ¿Qué tienen que modificar en `ServicioUsuarios`?

En el código original hay que modificar `ServicioUsuarios`, porque contiene directamente la creación y el tipo `MySQLDatabase`. En la solución corregida solo se crea una implementación diferente de `BaseDatos` y se inyecta desde afuera.

#### ¿Cómo harían una prueba de `registrar` sin conectarse a una base de datos real?

Se utiliza `BaseDatosEnMemoria`, una implementación de `BaseDatos` que almacena los datos en una lista. Así se puede verificar el resultado sin infraestructura externa.

#### ¿Quién decide qué base de datos se usa: `ServicioUsuarios` o alguien de afuera?

Debe decidirlo el código externo, por ejemplo `main`, una fábrica o un contenedor de dependencias. `ServicioUsuarios` solo conoce la abstracción `BaseDatos`.

### Código corregido

```java
import java.util.ArrayList;
import java.util.List;

interface BaseDatos {
    void guardar(String dato);
}

class MySQLDatabase implements BaseDatos {
    public void guardar(String dato) {
        System.out.println("[MySQL] Guardando: " + dato);
    }
}

class BaseDatosEnMemoria implements BaseDatos {
    private final List<String> datos = new ArrayList<>();

    public void guardar(String dato) {
        datos.add(dato);
    }

    public List<String> getDatos() {
        return new ArrayList<>(datos);
    }
}

class ServicioUsuarios {
    private final BaseDatos db;

    public ServicioUsuarios(BaseDatos db) {
        this.db = db;
    }

    public void registrar(String nombreUsuario) {
        if (nombreUsuario == null || nombreUsuario.isBlank()) {
            throw new IllegalArgumentException("Nombre inválido");
        }
        db.guardar(nombreUsuario);
    }
}

public class DemoDIP {
    public static void main(String[] args) {
        BaseDatosEnMemoria baseDatos = new BaseDatosEnMemoria();
        ServicioUsuarios servicio = new ServicioUsuarios(baseDatos);

        servicio.registrar("ana");
        servicio.registrar("lucia");

        System.out.println(baseDatos.getDatos());
    }
}
```

### Justificación

El servicio depende de la abstracción `BaseDatos`, no de MySQL. Esto permite cambiar la infraestructura o usar una implementación en memoria sin modificar la lógica de negocio.

### Evidencia

```text
[ana, lucia]
```

---

## Conclusión

Las preguntas guía muestran el efecto práctico de cada principio:

- **SRP:** una clase con varias responsabilidades cambia por muchos motivos.
- **OCP:** una cadena de condiciones obliga a modificar código existente para extenderlo.
- **LSP:** una subclase que no cumple el contrato de su padre rompe a los clientes.
- **ISP:** una interfaz grande obliga a implementar operaciones que no corresponden.
- **DIP:** una clase de negocio acoplada a MySQL es difícil de cambiar y probar.

Las correcciones no consisten únicamente en cambiar nombres: modifican los contratos y las dependencias para que el diseño sea más mantenible, extensible y comprobable
