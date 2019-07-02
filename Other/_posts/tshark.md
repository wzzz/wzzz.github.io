# tshark的使用

## 原理

## 使用说明
read filter -R 指定
capture filter -> pcap library -f指定（可以多个）

### 查看read filter的语法
man wireshark-filter

## 实践

tshark -i any -f 'port 3306' -R 'mysql.query matches "^update" and mysql.query contains t_script_alive' -T fields -e 'ip.src' -e 'mysql.query'