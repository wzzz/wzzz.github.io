
mysqldiff的研究：

object_type的获取：
	INFORMATION_SCHEMA.TABLES的TABLE_TYPE字段值
	mysql.proc的TYPE字段值
	INFORMATION_SCHEMA.TRIGGERS->'TRIGGER'
	mysql.event->'EVENT'

# List of database objects for enumeration
_DATABASE, _TABLE, _VIEW, _TRIG, _PROC, _FUNC, _EVENT, _GRANT = "DATABASE", \
    "TABLE", "VIEW", "TRIGGER", "PROCEDURE", "FUNCTION", "EVENT", "GRANT"
