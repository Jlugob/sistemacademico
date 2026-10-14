# Arquitectura (visión conceptual)

Una descripción de cómo está pensada la plataforma, sin entrar en detalles de implementación.

## Una plataforma, muchos colegios

SistemAcadémico es **multi-colegio** (*multi-tenant*): una sola aplicación atiende a varios colegios,
y cada uno vive en un espacio propio.

```
        ┌──────────────────────────────┐
        │     Administración central   │   gestiona colegios, módulos y marca
        └──────────────┬───────────────┘
                       │
     ┌─────────────────┼─────────────────┐
     ▼                 ▼                 ▼
 ┌────────┐       ┌────────┐       ┌────────┐
 │Colegio │       │Colegio │       │Colegio │   cada colegio:
 │   A    │       │   B    │       │   C    │   su base de datos,
 └────────┘       └────────┘       └────────┘   su marca, sus usuarios
```

Dos contextos separados:

- **Central:** administra la plataforma (altas de colegios, módulos habilitados, marca global).
- **Colegio:** un colegio aislado, con su propia base de datos. Se accede por una dirección propia del
  colegio.

## Aislamiento de datos

La decisión más importante: **cada colegio tiene su propia base de datos**. Los datos de un colegio
nunca comparten tablas con los de otro, de modo que un error o una consulta mal hecha no puede
mezclar información entre instituciones.

Si un colegio se suspende, su acceso se bloquea y sus sesiones se cierran.

## Una sola aplicación para todos

Todos los usuarios entran por la misma aplicación web, que se adapta según quién eres:

- **Funcionarios** (directivos, coordinadores, docentes, secretaría)
- **Estudiantes y familias**, con un portal de consulta
- **Administración de la plataforma**

La aplicación se puede instalar en el celular como una app (PWA) y toma el logo y los colores de cada
colegio. Los funcionarios pueden consultar información clave aun sin conexión.

## Tecnologías

| Capa | Tecnología |
|------|-----------|
| Servidor | Laravel (PHP) |
| Interfaz | React y Tailwind CSS |
| Datos | MySQL, una base por colegio |
| Trabajos en segundo plano | Colas con Redis (boletines en lote, correos, importaciones) |
| Panel de administración | Filament |
| Generador de horarios | Servicio en Python con optimización por restricciones |
| Pagos | Pasarela Wompi (PSE) |

## Principios de diseño

- **Automatizar lo repetitivo**, para devolverle tiempo a docentes y funcionarios.
- **Lo pesado va en segundo plano**: los procesos largos no hacen esperar a nadie.
- **Mínima información necesaria**: cada pantalla muestra solo lo que su rol necesita.
- **Probado antes de publicar**: cada cambio pasa por pruebas automáticas antes de llegar a producción.
