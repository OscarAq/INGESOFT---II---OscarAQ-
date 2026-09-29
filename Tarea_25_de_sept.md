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

Las entidades de software deben estar abiertas para extensión, pero cerradas para modificación.

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
