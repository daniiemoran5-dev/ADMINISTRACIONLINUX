# ✏️ Start writing

#!/usr/bin/env bash
#------------------------------------------------------------------------
#DEBER 2 SCRIPT PARA MONITOREO DEL SISTEMA
#INTEGRANTES: DIEGO CANCHIGNIA / DANIELA MORAN
#------------------------------------------------------------------------
#
#-----INGRESO DEL PARAMETRO---------------------------------------------
#Recibe el parametro del directorio como $1, sino se indica directorio
#se coloca por defecto "/"
directorio="${1:-/}"

#Servicio SSH a comprobarse
servicio="sshd"

#Generacion del archivo de reporte
reporte="reporte_sistema.txt"

#----VALIDACIONES DEL DIRECTORIO RECIBIDO---------------------------------
#Sentencia IF  para que valide que el directorio ingresado exista
#Caso contrario se informa el error y el script finaliza controladamente
if [ ! -d "$directorio" ]; then
	echo "ERROR: el directorio '$directorio' no existe "
	exit 1
fi

#----VARIABLES DE ESTADO GENERAL-----------------------------------------------
#Inicia  desde el estado "SIN ERRORES" y cambia cuando una verificacion falla
estado_general="SIN ERRORES"

#---DATOS PARA EL REPORTE-------------------------------------------------------
#-Fecha y hora actual del sistema
fecha=$(date '+%F %T')

#-Nombre del equipo
equipo=$(cat /etc/hostname)

#-Timpo de actividad del sistema 
tiempo_actividad=$(uptime -p)

#-Verificacion de almacenamiento
#Verificacion del porcentaje de uso con df --output=pcent
#tail -1 para descartar el encabezado -tr elimina espacios

uso_disco=$(df --output=pcent "$directorio" | tail -1 | tr -d ' %')

#Condicion de error al 90%
if [ "$uso_disco" -ge 90 ]; then
	estado_disco="ERROR"
	estado_general="SE DETECTO ERROR"
else
	estado_disco="OK"
fi


#-Verificacion memoria
#Con free -m muestra la memoria en MiB y "Mem" muestra la memoria disponible para nuevos procesos
memoria_disponible=$(free -m | awk '/Mem:/{print $7}')

#Condicional de 2000MiB sea OK
if [ "$memoria_disponible" -lt 2000 ]; then
	estado_memoria="ERROR"
	estado_general="SE DETECTO ERROR"
else
	estado_memoria="OK"
fi

#-Verificacion Servicio
#Systemctl is-active --quiet deevuelve codigo de salida (0 es activo; diferente inactivo)
#Condicionante servicio OK
if systemctl is-active --quiet "$servicio"; then
	estado_servicio="SERVICIO ACTIVO - OK"
else
	estado_servicio="SERVICIO INACTIVO - ERROR"
	estado_general="SE DETECTO ERROR"
fi

#---GENERACION REPORTE----------------------------------------------------
#Todo lo que esta dentro de {} se agrupa y se envia a una salida completa .txt

{
	echo "REPORTE DE MONITOREO SISTEMA"
	echo "fecha: $fecha"
	echo "equipo: $equipo"
	echo "Directorio analizado: $directorio"
	echo "Tiempo  UPTIME: $tiempo_actividad"
	echo "Uso Almacenamiento: ($directorio): ${uso_disco}% - $estado_disco"
	echo "Uso de memoria: ${memoria_disponible} MiB - $estado_memoria"
	echo "Servicio: $servicio: $estado_servicio"
	echo "RESULTADO GENERAL: $estado_general"

} > "$reporte"

#---VISOR DEL REPORTE EN TERMINAL-----------------------------------------
cat "$reporte"














