# TryHackmeWriteUp__Boogeyman-2

# 🕵️‍♂️ Boogeyman 2 – Write-Up

## 📌 Introducción
Tras el análisis realizado en el laboratorio **Boogeyman 1**, donde Quick Logistics LLC enfrentó un ataque inicial del grupo conocido como *Boogeyman*, la compañía reforzó sus defensas.  

Sin embargo, el adversario ha regresado con **tácticas, técnicas y procedimientos (TTPs) más avanzados**, lo que nos lleva a este nuevo escenario: **Boogeyman 2**.  
En este write-up documentamos el proceso de investigación, las herramientas utilizadas y los hallazgos principales de este segundo ataque.

---

## 🎯 Objetivo

Analizar los artefactos proporcionados (correo de phishing y volcado de memoria de la estación de trabajo comprometida) para identificar:  
- El vector inicial de compromiso.  
- La carga útil descargada y ejecutada.  
- Los mecanismos de persistencia implementados.  
- Las técnicas de exfiltración y comunicación con la infraestructura del atacante.  

---

## 🛠️ Herramientas utilizadas

- **olevba** → Para extraer y analizar macros de documentos maliciosos.  
- **Volatility 3** → Para explorar procesos, conexiones de red, librerías cargadas, archivos y memoria asociada.  
- **strings** → Para búsqueda de cadenas en dumps de memoria y artefactos.  
- **Wireshark / tshark** → Para el análisis de tráfico de red (DNS, TCP streams).  
- **CyberChef** → Para decodificación y conversión de datos (ej. Base64, decimal, hex).  

---

## 📩 Análisis del correo y del documento

Se entregó un correo de phishing con un archivo adjunto en formato Word.  
Mediante **olevba** se identificó un macro que, al abrir el documento, realizaba:  
- Descarga de un archivo malicioso desde infraestructura controlada por el atacante.  
- Ejecución del mismo utilizando `wscript.exe`.  

Este comportamiento confirmó el **vector inicial de compromiso: macros en documentos ofuscados**.  

---

## 🧠 Análisis de memoria

Con el volcado de memoria se aplicaron diferentes técnicas:  

- **Árbol de procesos (`pstree`)** para identificar la relación entre `winword.exe`, `wscript.exe` y procesos hijos maliciosos.  
- **`dlllist` y `memmap`** para revisar módulos y regiones de memoria de procesos sospechosos.  
- **`filescan`** para localizar copias del documento malicioso en cachés temporales.  
- **`netscan`** para evidenciar conexiones a dominios pertenecientes a la infraestructura del atacante.  

Esto permitió trazar el flujo desde la apertura del documento hasta la ejecución del malware descargado.  

---

## 🔒 Persistencia

Durante el análisis de memoria y dumps de proceso, se encontró la creación de una **tarea programada con `schtasks.exe`** que garantizaba la ejecución automática del payload a diario.  
El comando revelaba además el uso de **PowerShell con código almacenado en el registro del sistema**, técnica usada para ocultar la carga.  

Esta persistencia refuerza el uso de TTPs como:  
- **T1053.005 – Scheduled Task**  
- **T1547.001 – Registry for persistence**  

---

## 📡 Comunicación y exfiltración

El malware interactuaba con dominios bajo el control del atacante, además de emplear técnicas de exfiltración mediante consultas DNS con datos fragmentados en subdominios.  

Con herramientas como **tshark** y **CyberChef** se pudieron reconstruir partes de la información extraída.  

---

## 📑 Conclusión

El escenario **Boogeyman 2** demuestra la evolución del adversario respecto a la primera campaña:  
- De simples macros descargando binarios, pasaron a integrar **persistencia avanzada mediante tareas programadas y abuso del registro**.  
- Se observó un enfoque más sofisticado en **exfiltración y C2**, usando DNS y tráfico HTTP.  
- El análisis de memoria y de red resultó clave para identificar tanto el vector de acceso inicial como los mecanismos de persistencia y la infraestructura maliciosa.  

Este laboratorio refuerza la importancia de combinar el análisis de **artefactos locales (documentos, memoria, procesos)** con la investigación de **tráfico de red** para comprender completamente el ciclo de ataque.

---

## 🔗 Referencias
- [Volatility 3 Documentation](https://volatility3.readthedocs.io/)  
- [Oletools (olevba)](https://github.com/decalage2/oletools)  
- [MITRE ATT&CK Framework](https://attack.mitre.org/)  

