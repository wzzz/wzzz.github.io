# show slave status
```
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 10.249.7.201           ---来自于mysql.slave_master_info.Host
                  Master_User: repl            ---来自于mysql.slave_master_info.User_name
                  Master_Port: 3363       ---来自于mysql.slave_master_info.Port
                Connect_Retry: 60           ---来自于mysql.slave_master_info.Connect_retry
              Master_Log_File: mysql.000016             ---来自于mysql.slave_master_info.Master_log_name
          Read_Master_Log_Pos: 54771   	             ---来自于mysql.slave_master_info.Master_log_pos
               Relay_Log_File: mysql-relay-bin.000016            ---来自于mysql.slave_relay_log_info.Relay_log_name
                Relay_Log_Pos: 54930             ---来自于mysql.slave_relay_log_info.Relay_log_pos
        Relay_Master_Log_File: mysql.000016                ---来自于mysql.slave_relay_log_info.Master_log_name
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 54771                 ---来自于mysql.slave_relay_log_info.Master_log_pos
              Relay_Log_Space: 55262
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No           ---来自于mysql.slave_master_info.Enabled_ssl
           Master_SSL_CA_File:            ---来自于mysql.slave_master_info.Ssl_ca
           Master_SSL_CA_Path:             ---来自于mysql.slave_master_info.Ssl_capath
              Master_SSL_Cert:             ---来自于mysql.slave_master_info.Ssl_cert
            Master_SSL_Cipher:              ---来自于mysql.slave_master_info.Ssl_cipher
               Master_SSL_Key:               ---来自于mysql.slave_master_info.Ssl_key
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No              ---来自于mysql.slave_master_info.Ssl_verify_server_cert
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids:                ---来自于mysql.slave_master_info.Ignored_server_ids
             Master_Server_Id: 3363
                  Master_UUID: aca39ef9-6230-11e7-9879-d4ae5263ff25                ---来自于mysql.slave_master_info.Uuid
             Master_Info_File: /data/mysql/mysql_dir3353/mysql_data/master.info  ---当全局参数master_info_repository为FILE时，会slave_master_info到该文件
                    SQL_Delay: 0                    ---来自于mysql.slave_relay_log_info.Sql_delay
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for the slave I/O thread to update it
           Master_Retry_Count: 86400                 ---来自于mysql.slave_master_info.Retry_count
                  Master_Bind:               ---来自于mysql.slave_master_info.Bind
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0                ---来自于mysql.slave_master_info.Enabled_auto_position
                 Channel_Name:                ---来自于mysql.slave_master_info.Channel_name或者mysql.slave_relay_log_info.Channel_name，5.7版本新增
           Master_TLS_Version:                  ---来自于mysql.slave_master_info.Tls_Version，5.7版本新增
```
