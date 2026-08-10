# Orbe Entrena

Sistema de gestión para academias deportivas. Multi-cliente: cada academia ve
solo sus datos.

Producto de la línea Orbe (orbepy.com). **Independiente del sistema de Academia
DG** — no comparte repo, ni base, ni puente. Aquel se usa como referencia, nunca
como código.

## Estado

En construcción. No vendible hasta completar la lista de abajo.

## Qué significa "listo" (no se le agregan puntos)

- [ ] 1. Estructura de datos con cada academia separada, y las tres pruebas de
      seguridad pasando
- [ ] 2. Crear academia e invitar por correo
- [ ] 3. Alumnos: alta, edición, búsqueda
- [ ] 4. Calendario y horarios
- [ ] 5. Asistencia
- [ ] 6. Importar y exportar CSV
- [ ] 7. Prueba de 30 días desde el primer alumno; al vencer queda solo lectura

## Las tres pruebas que tienen que FALLAR

Antes de cargar un solo dato real, en el simulador de reglas de Firebase:

1. Leer una academia sin estar logueado → tiene que fallar
2. Leer la academia de otro cliente → tiene que fallar
3. Agregarse a sí mismo a `/miembros` desde el navegador → tiene que fallar

Si alguna pasa, no se carga nada hasta arreglarlo.

## Estructura de datos

```
/academias/{idAcademia}/
    config/        nombre, moneda, sedes, plan, vence, modulos
    alumnos/
    profesores/
    calendario/
    asistencia/
    cobros/        ← nodo aparte, solo admin
    log/

/miembros/{uid}/{idAcademia}   →  { rol: "admin" | "profe" }
/superadmin/{uid}              →  true
```

**Regla de diseño:** lo que necesita permisos distintos va en un **nodo**
distinto, no en filas distintas. Realtime Database sabe poner permisos por nodo,
no por fila.

**`/miembros` no lo escribe ninguna app.** Solo el superadmin. Si un cliente
pudiera escribir ahí, se agregaría a la academia que quiera.

## Decisiones tomadas

| Tema | Decisión |
|---|---|
| Base de datos | Realtime Database (lo que ya funciona, sin tecnología nueva) |
| Identidad | `auth.uid`, no el correo — no se rompe con los puntos |
| Login | Google |
| Permisos | Invitación por correo, tres roles: dueño, admin, profe |
| Altas de academias | Manuales al principio, desde panel propio. Sin backend |
| Precio | Prueba de 30 días. Sin plan gratis |
| Reloj de la prueba | Arranca con el primer alumno cargado, no al registrarse |
| Al vencer | Solo lectura. Los datos no se borran |
| Alcance | Cualquier deporte, cualquier país. Nada clavado a pádel ni a Paraguay |
| Google Sheets | No se usa. Solo importar y exportar CSV |

## Archivos

- `index.html` — app principal (login, academia, alumnos)
- `reglas-firebase.json` — reglas de seguridad, se pegan en la consola

## Antes de publicar cualquier cambio

1. Abrir en incógnito y mirar la consola
2. Entrar con una cuenta de prueba
3. Hacer la operación principal
4. Verificar desde otro dispositivo

No publicar viernes a la tarde ni antes de un feriado.
