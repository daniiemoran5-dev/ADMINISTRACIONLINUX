**# 🖥️ Deber 2 - Script para Monitoreo del Sistema**



**\*\*Integrantes:\*\* Diego Canchignia / Daniela Moran**



**---**



**## Descripción**



**Script desarrollado en Bash para realizar verificaciones básicas del estado del sistema Linux. Permite analizar el uso de almacenamiento de un directorio, comprobar la memoria disponible y verificar el estado del servicio SSH.**



**Al finalizar la ejecución, genera un archivo denominado `reporte\_sistema.txt` con los resultados del monitoreo.**



**---**



**## Script de monitoreo**



**```bash**

**#!/usr/bin/env bash**



**# ------------------------------------------------------------------------**

**# DEBER 2 - SCRIPT PARA MONITOREO DEL SISTEMA**

**# INTEGRANTES: DIEGO CANCHIGNIA / DANIELA MORAN**

**# ------------------------------------------------------------------------**



**# --- INGRESO DEL PARAMETRO ----------------------------------------------**

**# Recibe el parametro del directorio como $1.**

**# Si no se indica un directorio, se utiliza "/" por defecto.**



**directorio="${1:-/}"**



**# Servicio SSH a comprobar**

**servicio="sshd"**



**# Archivo de reporte**

**reporte="reporte\_sistema.txt"**





**# --- VALIDACIONES DEL DIRECTORIO RECIBIDO -------------------------------**

**# Valida que el directorio ingresado exista.**

**# Caso contrario, informa el error y finaliza el script.**



**if \[ ! -d "$directorio" ]; then**

    **echo "ERROR: el directorio '$directorio' no existe"**

    **exit 1**

**fi**





**# --- VARIABLES DE ESTADO GENERAL ----------------------------------------**

**# Inicia con el estado "SIN ERRORES".**

**# El estado cambia cuando alguna verificacion falla.**



**estado\_general="SIN ERRORES"**





**# --- DATOS PARA EL REPORTE ----------------------------------------------**



**# Fecha y hora actual del sistema**

**fecha=$(date '+%F %T')**



**# Nombre del equipo**

**equipo=$(cat /etc/hostname)**



**# Tiempo de actividad del sistema**

**tiempo\_actividad=$(uptime -p)**





**# --- VERIFICACION DE ALMACENAMIENTO -------------------------------------**

**# df --output=pcent obtiene el porcentaje de uso.**

**# tail -1 descarta el encabezado.**

**# tr elimina espacios y el simbolo %.**



**uso\_disco=$(df --output=pcent "$directorio" | tail -1 | tr -d ' %')**



**# Se genera un estado de error cuando el uso es igual o superior al 90%.**



**if \[ "$uso\_disco" -ge 90 ]; then**

    **estado\_disco="ERROR"**

    **estado\_general="SE DETECTO ERROR"**

**else**

    **estado\_disco="OK"**

**fi**





**# --- VERIFICACION DE MEMORIA --------------------------------------------**

**# free -m muestra la memoria en MiB.**

**# Se obtiene la memoria disponible para nuevos procesos.**



**memoria\_disponible=$(free -m | awk '/Mem:/{print $7}')**



**# Se considera error si existen menos de 2000 MiB disponibles.**



**if \[ "$memoria\_disponible" -lt 2000 ]; then**

    **estado\_memoria="ERROR"**

    **estado\_general="SE DETECTO ERROR"**

**else**

    **estado\_memoria="OK"**

**fi**





**# --- VERIFICACION DEL SERVICIO SSH --------------------------------------**

**# systemctl is-active --quiet devuelve 0 si el servicio esta activo.**



**if systemctl is-active --quiet "$servicio"; then**

    **estado\_servicio="SERVICIO ACTIVO - OK"**

**else**

    **estado\_servicio="SERVICIO INACTIVO - ERROR"**

    **estado\_general="SE DETECTO ERROR"**

**fi**





**# --- GENERACION DEL REPORTE ---------------------------------------------**

**# El contenido generado se guarda en reporte\_sistema.txt.**



**{**

    **echo "============================================"**

    **echo "       REPORTE DE MONITOREO DEL SISTEMA"**

    **echo "============================================"**

    **echo "Fecha:                 $fecha"**

    **echo "Equipo:                $equipo"**

    **echo "Directorio analizado:  $directorio"**

    **echo "Tiempo UPTIME:         $tiempo\_actividad"**

    **echo "Uso almacenamiento:    ${uso\_disco}% - $estado\_disco"**

    **echo "Memoria disponible:    ${memoria\_disponible} MiB - $estado\_memoria"**

    **echo "Servicio $servicio:    $estado\_servicio"**

    **echo "--------------------------------------------"**

    **echo "RESULTADO GENERAL:     $estado\_general"**

    **echo "============================================"**



**} > "$reporte"**





**# --- VISUALIZACION DEL REPORTE EN TERMINAL ------------------------------**



**cat "$reporte"**

**```**



**---**



**## Reporte generado**



**La ejecución del script genera el archivo:**



**```text**

**reporte\_sistema.txt**

**```**



**El reporte contiene:**



**- Fecha y hora de ejecución**

**- Nombre del equipo**

**- Directorio analizado**

**- Tiempo de actividad del sistema**

**- Porcentaje de almacenamiento utilizado**

**- Memoria disponible**

**- Estado del servicio SSH**

**- Estado general del sistema**



**### Ejemplo de salida**



**```text**

**============================================**

       **REPORTE DE MONITOREO DEL SISTEMA**

**============================================**

**Fecha:                 2026-09-25 18:10:25**

**Equipo:                servidor-linux**

**Directorio analizado:  /**

**Tiempo UPTIME:         up 2 hours, 15 minutes**

**Uso almacenamiento:    42% - OK**

**Memoria disponible:    3250 MiB - OK**

**Servicio sshd:         SERVICIO ACTIVO - OK**

**--------------------------------------------**

**RESULTADO GENERAL:     SIN ERRORES**

**============================================**

**```**



