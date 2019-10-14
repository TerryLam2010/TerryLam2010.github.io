Mysql 导入导出语句



导入

mysql -ucbprd -p cbprd --default-character-set=utf8mb4 -S /tmp/mysql.cbprd.sock </home/cbprd/cb_itemcode20191010.sql

导出 excel

mysql -h10.82.34.74 -ucbprd -p   -e "select cps.item_code,cps.brand_id as '品牌ID',cb.name as '品牌名',cps.intro as '描述'
from cb_itemcode20191010 ci
inner join cb_product_sku cps on cps.item_code = ci.item_code
left join cb_brand cb on cps.brand_id = cb.brand_id" cbprd> /home/cbprd/temp201910101354.xls;



导出sql

全库

mysqldump -ucbprd -p cbprd  --set-gtid-purged=OFF --default-character-set=utf8mb4 -S /tmp/mysql.cbprd.sock > cbprd.full.20191008.sql



按表

mysqldump -ucbprd -p --set-gtid-purged=OFF cbprd cb_product_spu -S /tmp/mysql.cbprd.sock > /tmp/cb_product_spu.sql