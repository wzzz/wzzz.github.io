# tcpdump和tshark的使用

## 原理

## 使用说明
read filter -R 指定
capture filter -> pcap library -f指定（可以多个）

### 查看read filter的语法
man wireshark-filter

## 实践

tshark -i any -f 'dst host 10.126.126.157 and dst port 3306' -R 'mysql.query matches "^update" and mysql.query contains t_script_alive' -T fields -e 'ip.src' -e 'mysql.query'
tshark -i any -f 'dst host 10.126.126.157 and dst port 3306' -a duration:20 -T fields -R 'mysql.query' -e 'ip.src' -e tcp.srcport -e 'mysql.query' | awk '{print $1, $2}' | sort -u | uniq


tshark -i any -f 'dst host 10.128.226.175 and dst port 4695' -a duration:10 -R 'mysql.query matches "^select" and mysql.query contains hu_tables_upload_status' -T fields -e 'ip.src' -e tcp.srcport -e 'mysql.query'

根据tcpdump抓的pcap包分析：
# 文本格式
tshark -d tcp.port==4598,mysql -R 'mysql.query' -r 4598.cap -T fields -e 'frame.time' -e 'ip.src' -e tcp.srcport -e 'mysql.query' > 4598_result.txt
# xml格式
tshark -d tcp.port==4598,mysql -R 'mysql.query' -r 4598.cap -T pdml > 4598_result.xml

tcpdump的抓包：
timeout 1 /usr/sbin/tcpdump -lnn -i any 'dst host 10.128.226.175 and dst port 4695' > t.txt
timeout 1 /usr/sbin/tcpdump -lnn -A -i any 'dst host 10.128.226.175 and dst port 4695' > t.txt

抓30s后，生成可读文本文件查看
nohup timeout 30s /usr/sbin/tcpdump -i any 'dst host 10.128.227.58 and dst port 4624' -w /work/dba/4624.cap -l 2>&1 &
watch -n 1 ls -lh /work/dba/4598.cap
/usr/sbin/tcpdump -A -lnn -r 4624.cap | less

/usr/bin/timeout 600s /usr/sbin/tcpdump -l -i any 'dst host 10.128.226.175 and dst port 4695' -w /work/dba/4695-`date +"%Y%m%d%H%M%S"`.cap