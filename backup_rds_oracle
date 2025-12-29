#!/bin/bash

# Variables de entorno de Oracle
export ORACLE_HOME=/usr/lib/oracle/23/client64  # Ajusta según tu instalación
export PATH=$ORACLE_HOME/bin:$PATH
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$LD_LIBRARY_PATH

# Variables de conexión
RDS_HOST="nombre del host rds
RDS_PORT="1521"
RDS_SID="SID de la BD"
USUARIO="user"
PASSWORD="password"
ESQUEMA="mismo nombre que el usuario"

# Generar nombre de archivo con fecha
FECHA=$(date +%Y%m%d)
NOMBRE_ARCHIVO="export_${FECHA}.dmp"
LOG_FILE="export_${FECHA}.log"

# Directorio donde se guardará el backup (local)
BACKUP_DIR="/home/ec2-user/backups"
mkdir -p $BACKUP_DIR

# Ejecutar expdp
expdp ${USUARIO}/${PASSWORD}@//${RDS_HOST}:${RDS_PORT}/${RDS_SID} \
  schemas=${ESQUEMA} \
  directory=DATA_PUMP_DIR \
  dumpfile=${NOMBRE_ARCHIVO} \
  logfile=${LOG_FILE}

# Opcional: Mover archivos a directorio local si es necesario
# O subirlos a S3
# aws s3 cp ${NOMBRE_ARCHIVO} s3://tu-bucket/backups/

echo "Backup completado: ${NOMBRE_ARCHIVO}"

# Transferir archivo de RDS a S3 usando procedimiento PL/SQL
sqlplus user/pass@hostrds/DBSID << EOF
SET SERVEROUTPUT ON
SELECT rdsadmin.rdsadmin_s3_tasks.upload_to_s3(
    p_bucket_name => 'nombre del bucket',
    p_directory_name => 'DATA_PUMP_DIR'
) AS TASK_ID FROM DUAL;
EXIT;
EOF

log_message "✓ Archivo transferido de RDS a S3"
