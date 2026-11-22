# 🧠 Entrenamiento de Pensamiento Crítico para Ingeniería de Software

Este documento funciona como **plan de entrenamiento, bitácora de ejercicios y radar de habilidades**. El objetivo es desarrollar un pensamiento crítico riguroso aplicable al diseño de sistemas, código, producto y resolución de incidentes.

---

## ⚙️ Dinámica de cada Ejercicio

Cada sesión sigue un ciclo de 4 pasos:
1. **El Escenario (Reto):** Se plantea un caso realista con información incompleta, supuestos ocultos o trampas de diseño.
2. **Tu Análisis:** Respondes cómo abordarías el problema, qué preguntas harías o qué decisión tomarías.
3. **Stress-Test Socrático:** Analizamos tu respuesta buscando puntos ciegos, sesgos cognitivos o efectos de segundo orden no contemplados.
4. **Actualización de Bitácora:** Registramos en este archivo tus aciertos, los puntos ciegos descubiertos y qué habilidad debemos reforzar en el siguiente reto.

---

## 🗺️ Plan de Módulos y Ejercicios

| # | Módulo / Habilidad Crítica | Ejercicio | Estado | Nivel Logrado |
| :-: | :--- | :--- | :-: | :-: |
| **1** | **Cuestionamiento de Requerimientos y Supuestos** | *El Ticket "Urgente" del CEO / Producto* (Separar problema de solución y detectar ambigüedades) | ✅ Completado | 🟢 Sólido (con refuerzo en flujos asíncronos y precisión de riesgos) |
| **2** | **Pensamiento de Segundo Orden y Modos de Fallo** | *La Transferencia y el Doble Cobro* (Concurrencia, fallos parciales e idempotencia) | 🔄 Siguiente | - |
| **3** | **Análisis de Trade-offs y Sesgo de Confirmación** | *El Abogado del Diablo Arquitectónico* (Defender y destruir una decisión de diseño) | ⏳ Pendiente | - |
| **4** | **Diagnóstico Científico y Causa Raíz** | *Misterio en Producción a las 3 AM* (Formulación de hipótesis refutables vs. adivinar) | ⏳ Pendiente | - |
| **5** | **Escala, Cuellos de Botella y Simplificación** | *El Juego de los Extremos (0.01x vs 100x)* (Detectar sobreingeniería y límites físicos) | ⏳ Pendiente | - |
| **6** | **Decisiones Irreversibles y Contratos (APIs/Datos)** | *Puertas de Una Vía vs. Dos Vías* (Acoplamiento, migraciones y evolución del sistema) | ⏳ Pendiente | - |

---

## 📊 Radar de Habilidades (Para Seguimiento Continuo)

A medida que avancemos, iremos actualizando qué áreas están sólidas y cuáles requieren más presión:

### ✅ Fortalezas Demostradas
* **Detección de volumen no acotado:** Identificas rápidamente cuando un requerimiento ("traer todo") implica volúmenes de datos peligrosos (ej. cientos de MBs).
* **Escepticismo saludable ante estimaciones triviales:** No aceptas el clásico *"es solo un SELECT de 2 horas"* sin evaluar qué componentes faltan construir.
* **Protección de la Base de Datos Productiva (OLTP vs. OLAP):** Excelente criterio para separar cargas transaccionales (clientes operando en vivo) de extracciones analíticas masivas (desviar lecturas pesadas a BigQuery / Data Warehouse).

### 🎯 Puntos Ciegos / Áreas a Reforzar
* **Reflejo de construir antes de desarmar el Problema XY:** En la primera iteración aceptaste construir el exportador CSV sin cuestionar que el dolor real era la lentitud del buscador web (y que un CSV de 500MB tampoco abriría en Excel). *Regla: Cuestionar el "para qué" antes de diseñar el "cómo".*
* **Efectos de segundo orden en el servidor (Infraestructura):** Al evaluar cargas grandes, no solo pensar en el peso de descarga para el usuario, sino en los límites físicos del backend: **RAM (OOM crash), Timeouts de Gateway HTTP (30-60s)** y **peticiones duplicadas por impaciencia del usuario**.
* **Especificidad en Seguridad:** Pasar de intuiciones generales (*"debe estar protegido de alguna forma"*) a amenazas con nombre y apellido (Rate Limiting, RBAC/Exfiltración de datos, CSV Injection).

### 🧩 Caja de Herramientas (Modelos Mentales Incorporados)
1. **El Problema XY:** El usuario pide tu ayuda con su solución inventada (*Y: exportar a Excel*), en lugar de explicarte su problema raíz (*X: la búsqueda por fecha tarda 20 segundos*).
2. **Separación OLTP vs. OLAP:** Nunca castigar la base de datos transaccional productiva con consultas analíticas o extracciones masivas; usar Data Warehouse (BigQuery) o Réplicas de Lectura.
3. **Solución Cero-Código vs. Flujo Asíncrono:** Si una extracción masiva es puntual, resolverla fuera de la app (BigQuery directo). Si debe estar en la app, nunca usar un `GET` síncrono: usar **Job Asíncrono + Streaming a Object Storage (S3/GCS) + Link de descarga (Signed URL)**.

---

## 📓 Historial de Sesiones

### Ejercicio 1: Cuestionamiento de Requerimientos y Supuestos Ocultos (*"El Botón de Exportar Todo"*)
* **Estado:** ✅ Completado (2026-11-22)
* **Objetivo:** Detectar qué información falta, qué supuestos son peligrosos y cuál es el problema real antes de proponer arquitectura o código.
* **Resumen de tu respuesta:**
  * *Iteración 1:* Detectaste el riesgo del tamaño del archivo (500MB, proponiendo particionarlo en bloques de 100MB), cuestionaste si existía el componente generador para refutar la estimación de 2 horas y mencionaste proteger el servicio.
  * *Iteración 2 (Post Stress-Test):* Atacaste la causa raíz preguntando cómo buscan los usuarios para revisar y crear índices en la BD, y propusiste proteger la BD productiva extrayendo los datos pesados desde **BigQuery** en lugar de golpear el motor transaccional.
* **Análisis y Feedback:**
  * Muy buena evolución entre la primera y segunda respuesta. El salto clave fue pasar de *"¿cómo divido el archivo de 500MB?"* a *"¿por qué no arreglamos el índice del buscador y usamos BigQuery para no degradar producción?"*.
* **Lección accionable:** Antes de estimar o dividir una historia técnica, hacer siempre la pregunta: *"¿Qué problema intenta resolver el usuario final con esto, y qué pasa en la memoria/red del servidor si el volumen es 100 veces mayor al de desarrollo?"*
