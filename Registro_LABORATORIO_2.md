# Registro  Laboratorio 2
**Materia:** Aplicaciones Móviles  
**Laboratorio:** 2 Ciclo de vida, navegación e Intents  
**Alumno:** Diego Montaño  
**Fecha:** 2026-09-22  

---

## Checklist Oficial de Salida

- [x] **HolaYo tiene dos pantallas conectadas por un Intent explícito que lleva un dato**
- [x] **Ambas Activities instrumentadas con Logcat (`tag:VIDA`); sé leer el entrelazado de navegación**
- [x] **Tabla de la tortura completa**, con predicciones escritas antes de cada observación
- [x] **Presencié la recreación por giro:** secuencia en el log, contador perdido (vuelve a 0), frase de notas preservada (gracias a su `android:id`), y sé explicar el porqué
- [x] **Maté el proceso con adb en fondo (`adb shell am kill ...`),** verifiqué el PID nuevo y entendí qué reconstruye el sistema y qué se pierde
- [x] **Detuve la ejecución con un breakpoint** e inspeccioné variables vivas en el depurador

---

##  Etapa 3: Tabla del Experimentador (La Tortura Metódica)

| #   | Acción (con la Segunda pantalla abierta)                    | Mi predicción                                                                       | Lo observado en Logcat (`tag:VIDA`)                                                      |
| --- | ----------------------------------------------------------- | ----------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| 1   | **Apretar botón Home** (la app va a segundo plano)          | `onPause → onStop` se detiene pero el proceso y la Activity siguen vivos en memoria | `Segunda → onPause`<br>`Segunda → onStop`                                                |
| 2   | **Volver a la app desde Recientes**                         | `onStart → onResume` revive sin pasar por `onCreate`                                | `Segunda → onStart`<br>`Segunda → onResume`                                              |
| 3   | **Apagar la pantalla** con botón de encendido               | `onPause → onStop` pantalla apagada equivale a pasar a fondo                        | `Segunda → onPause`<br>`Segunda → onStop`                                                |
| 4   | **Encender la pantalla y desbloquear**                      | `onStart → onResume` vuelve a ser visible e interactiva                             | `Segunda → onStart`<br>`Segunda → onResume`                                              |
| 5   | **Abrir cortina de notificaciones** completa y cerrarla     | `onPause → onResume`                                                                | `Segunda → onPause → Segunda → onResume`                                                 |
| 6   | **Recibir algo encima** (abrir otra app o diálogo) y volver | `onPause → onStop` al ser tapada, luego `onStart → onResume` al regresar            | `Segunda → onPause`<br>`Segunda → onStop`<br>`Segunda → onStart`<br>`Segunda → onResume` |

---
##  Etapa 4: El Giro

- **¿Qué pasó con el contador?**  
  Volvió a **0**. La variable `contador` era una propiedad de instancia de la `Activity` destruida; la nueva instancia nació limpia ejecutando su inicializador original (`contador = 0`).
- **¿Qué pasó con el campo de texto?**  
  La frase escrita se mantuvo **intacta**. El sistema Android guarda y restaura automáticamente el estado de las vistas de la UI siempre y cuando tengan un `android:id` asignado.

---
##  Etapa 5: Muerte por Memoria (Provocada por ADB)

- **PID inicial (con la app en la 2da pantalla):** PID: 13144
- **Comando ejecutado:** `adb shell am kill com.example.holayo` 
- **PID nuevo al regresa:** PID nuevo: 13677
- **¿Qué sobrevivió y qué se perdió?**
  - Saludo con nombre:  Sobrevivió 
  - Campo de notas : Sobrevivió 
  - Variable contador : Se perdió 
- **Conclusión de continuidad:**  Para el usuario es indistinto, pero por debajo el proceso fue liquidado al 100%. Android sostiene esta ilusión reconstituyendo la pila de Activities, reentregando los Intents originales y restaurando los estados de vistas con ID. 

---
