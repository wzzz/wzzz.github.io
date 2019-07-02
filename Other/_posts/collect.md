
file=collect.txt
com_select=0
com_update=0
com_insert=0
com_delete=0
com_slow=0
while true; do
    echo `date +"%Y-%m-%d %H:%M:%S.%N"` >> ${file}
    query_info=`mysql -S /work/mysql4558/tmp/mysql.sock -uroot -pxxx -N -s -e "show global status where variable_name in ('Com_select', 'Com_update', 'Com_delete', 'Com_insert', 'Slow_queries')"`
    t_com_select=`echo "${query_info}" | grep Com_select | awk '{print $2}'`
    select=`echo ${t_com_select} - ${com_select} | bc`
    echo "select:"${select} >> ${file}
    com_select=${t_com_select}

    t_com_update=`echo "${query_info}" | grep Com_update | awk '{print $2}'`
    update=`echo ${t_com_update} - ${com_update} | bc`
    echo "update:"${update} >> ${file}
    com_update=${t_com_update}

    t_com_insert=`echo "${query_info}" | grep Com_insert | awk '{print $2}'`
    insert=`echo ${t_com_insert} - ${com_insert} | bc`
    echo "insert:"${insert} >> ${file}
    com_insert=${t_com_insert}

    t_com_delete=`echo "${query_info}" | grep Com_delete | awk '{print $2}'`
    delete=`echo ${t_com_delete} - ${com_delete} | bc`
    echo "delete:"${delete} >> ${file}
    com_delete=${t_com_delete}

    t_com_slow=`echo "${query_info}" | grep Slow_queries | awk '{print $2}'`
    slow=`echo ${t_com_slow} - ${com_slow} | bc`
    echo "slow:"${slow} >> ${file}
    com_slow=${t_com_slow}
    sleep 0.2
done