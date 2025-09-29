# Ansible Collection - `demo_company.mssql`

This collection automates Microsoft SQL Server database backup, restore, and validation workflows on Windows hosts using the `dbatools` PowerShell module and the `ansible.windows` collection.

## Requirements
- Ansible `>= 2.14`
- Collection dependency: `ansible.windows >= 1.13.0`
- Managed host: Windows with WinRM configured
- SQL Server instance available on the managed host
- PowerShell Module `dbatools` (will be installed by roles if missing)

## Roles
- `backup`: Performs a compressed COPY_ONLY full backup and fetches the `.bak` to the controller.
- `restore`: Copies the `.bak` to the target and restores it, optionally moving data/log files.
- `validate`: Validates the restored database (ONLINE status, `RESTORE VERIFYONLY`, optional DBCC, custom queries).

## Role Variables (common)
- `sql_instance` (string): SQL Server instance name. Default: `MSSQLSERVER`.
- `db_name` (string): Database name to operate on.
- `backup_dir` (path): Backup directory on Windows host. Default: `C:\\MSSQL_Backups`.

### `backup` variables
- `controller_backup_path` (path): Path on controller where `.bak` is stored. Default: `{{ playbook_dir }}/files/{{ db_name }}.bak`.

### `restore` variables
- `restore_data_path` (path): Destination data dir for `.mdf`.
- `restore_log_path` (path): Destination log dir for `.ldf`.
- `controller_backup_path` (path): Source `.bak` on controller.

### `validate` variables
- `validate_online` (bool): Ensure DB is ONLINE. Default: `true`.
- `validate_verifyonly` (bool): Run VERIFYONLY. Default: `true`.
- `validate_dbcc` (bool): Run `DBCC CHECKDB`. Default: `false`.
- `validation_queries` (list[string]): Optional custom T-SQL queries to run post-restore.

## Example Playbook
```yaml
- hosts: windows
  collections:
    - demo_company.mssql
  roles:
    - role: backup
      vars:
        sql_instance: MSSQLSERVER
        db_name: ExampleDb
        backup_dir: C:\\MSSQL_Backups

    - role: restore
      vars:
        sql_instance: MSSQLSERVER
        db_name: ExampleDb
        backup_dir: C:\\MSSQL_Backups
        restore_data_path: C:\\Program Files\\Microsoft SQL Server\\MSSQL16.MSSQLSERVER\\MSSQL\\DATA
        restore_log_path: C:\\Program Files\\Microsoft SQL Server\\MSSQL16.MSSQLSERVER\\MSSQL\\DATA

    - role: validate
      vars:
        sql_instance: MSSQLSERVER
        db_name: ExampleDb
        backup_dir: C:\\MSSQL_Backups
        validate_online: true
        validate_verifyonly: true
        validate_dbcc: false
```

## Installation
```bash
ansible-galaxy collection build
ansible-galaxy collection install ./demo_company-mssql-*.tar.gz
```

## License
MIT-0

## Authors
Demo Company
