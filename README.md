Ansible Postgresql
=========

Requirements
------------
no requirements

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:
```yaml

```

## How is installed in default

Restic bin    /opt/restic/                           <restic_download_path>
                          resitc_version/
                                         0.7
                                         0.8
                                         0.9
                                         1.1
                                         1.2/restic

resitc bin    /usr/local/bin/resitc --> /opt/restic/resitc_version/1.2/resitc

restic script /etc/restic/                           <restic_configuration_path>
                         script
                         config


# Example script

## Postgres dump
```
echo "start postgres backup"
DATE="$(date +%Y%m%d%H%M)"
DATABASE_BACKUP_PATH={{restic_backup_path}}
metrics_path=/tmp/prometheus_metrics
mkdir -p $metrics_path
DATABASE_LIST=$(su - postgres -c "psql -ltA -F "," | cut -d "," -f1 | grep -vE '(template|postgres)'")
PROMETHEUS_RESULT_FILE=$metrics_path/prometheus_metrics.prom
# reset metrics
> $PROMETHEUS_RESULT_FILE
echo "postgres_backup_last_run{host=\"$HOSTNAME\"} $DATE" >> $PROMETHEUS_RESULT_FILE
for database in ${DATABASE_LIST}
do
  echo "postgres_backup{host=\"$HOSTNAME\", database=\"$database\"} 1" >> $PROMETHEUS_RESULT_FILE
  echo "Starting backup of ${database}"
  start=`date +%s`
  su - postgres -c "pg_dump --create $database -Fd --jobs={{backup_jobs_number}} -f $DATABASE_BACKUP_PATH/${database}_${DATE}"
  if [[ "$?" == "0" ]]
  then
      echo "postgres_backup_database_status{host=\"$HOSTNAME\", database=\"$database\"} 1" >> $PROMETHEUS_RESULT_FILE
  else
      echo postgres_backup_status{host=\"$HOSTNAME\", database=\"$database\"} 0 >> $PROMETHEUS_RESULT_FILE
  fi
  end=`date +%s`
  runtime=$((end-start))
  echo "postgres_database_backup_duration{host=\"$HOSTNAME\", database=\"$database\"} $runtime" >> $PROMETHEUS_RESULT_FILE
  echo "postgres_database_backup_last_run{host=\"$HOSTNAME\", database=\"$database\"} `date +%s`" >> $PROMETHEUS_RESULT_FILE
done
su - postgres -c "pg_dumpall --globals-only > ${DATABASE_BACKUP_PATH}/schema_$DATE.sql"
if [[ "$?" == "0" ]]
then
  echo "postgres_schema_backup_last_run{host=\"$HOSTNAME\", database=\"schema\"} `date +%s`" >> $PROMETHEUS_RESULT_FILE
fi
find ${DATABASE_BACKUP_PATH} -mindepth 1 -maxdepth 1 -type d -mtime +{{keep_hotbackup}} | xargs rm -rf
```


## For restore db
- **-c**  elimina il vecchio database
- **-C**  Crea il database e restora li dentro i file
- **-Fd** Formato Directory

```
su - postgres -c "pg_restore -c -C -d postgres -Fd /var/lib/pgsql/13/backups/<DIR_BACKUP>/"
```

License
-------

MIT

## Devel
for devel use
https://github.com/ansible-community/molecule-libvirt

requirements for testings
KVM host
```
libvirt-python
```
# ansible-restic
