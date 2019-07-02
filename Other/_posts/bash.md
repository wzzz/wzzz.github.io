# 循环读入变量
将line变量的值读入old_db table new_db中
read old_db table new_db <<< $(echo $line)


# rpm命令
rpm2cpio xxx.rpm | cpio -div


# lsattr和chattr命令
lsattr /etc/group
lsattr /etc/gshadow
chattr -i /etc/group
chattr -i /etc/gshadow
chattr +i /etc/group
chattr +i /etc/gshadow

# 美化输出json
echo `curl -s "http://db-api.58dns.org/cluster/getclusterbyip/10.126.85.157?token=cdb"`| python -mjson.tool
