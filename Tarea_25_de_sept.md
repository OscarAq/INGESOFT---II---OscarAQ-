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
