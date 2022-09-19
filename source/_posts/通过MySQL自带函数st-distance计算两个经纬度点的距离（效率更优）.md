---
title: 通过MySQL自带函数st_distance计算两个经纬度点的距离（效率更优）
date: 2022-05-19 07:02:33
categories: Java
tags: [MySQL,Java]
---
## 情况一

> 数据库：有point类型的location字段
> 实体类：有经纬度字段(double)

```sql
select
		regionId,
		provinceName,
		cityName,
		districtName,
		lon,
		lat,
		(st_distance
		(location,point(31.0, 103.0) ) / 0.0111) AS distance
		FROM
		t_region
		WHERE 1=1
		AND deleted = 0
		order by distance asc
```

## 情况二

> 数据库：有经度纬度字段，但是没有point字段
> 实体类：有经纬度字段(double)

```sql
select
		regionId,
		provinceName,
		cityName,
		districtName,
		lon,
		lat,
		(st_distance
		(point(lon, lat),point(31.0, 103.0) ) / 0.0111) AS distance
		FROM
		t_region
		WHERE 1=1
		AND deleted = 0
		order by distance asc
```
## 附：建表语句

```sql
CREATE TABLE `t_region`  (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `regionId` int NOT NULL DEFAULT 0 COMMENT '编号规则6位有符号整数，例如110000。',
  `provinceName` varchar(32) CHARACTER SET utf8 COLLATE utf8_bin NOT NULL DEFAULT '' COMMENT '省编号',
  `cityName` varchar(32) CHARACTER SET utf8 COLLATE utf8_bin NOT NULL DEFAULT '' COMMENT '市编号',
  `districtName` varchar(32) CHARACTER SET utf8 COLLATE utf8_bin NOT NULL DEFAULT '' COMMENT '区县编号',
  `lon` double NOT NULL DEFAULT 0 COMMENT '经度',
  `lat` double NOT NULL DEFAULT 0 COMMENT '纬度',
  `location` point NULL COMMENT '经纬度point属性方便使用mysql地理位置运算',
  `parentRegionId` int NOT NULL DEFAULT 0 COMMENT '父节点',
  `createTime` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `createUser` varchar(20) CHARACTER SET utf8 COLLATE utf8_bin NOT NULL DEFAULT 'sys' COMMENT '创建人',
  `updateTime` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  `updateUser` varchar(20) CHARACTER SET utf8 COLLATE utf8_bin NOT NULL DEFAULT 'sys' COMMENT '修改人',
  `deleted` int NOT NULL DEFAULT 0 COMMENT '删除状态 0 正常 1 已删除',
  `version` int NOT NULL DEFAULT 0 COMMENT '修改序号',
  PRIMARY KEY (`id`) USING BTREE,
)
```

```sql
INSERT INTO `t_region` VALUES (1, 510000, '四川省', '', '', 104.081703, 30.65722, ST_GeomFromText('POINT(30.65722 104.081703)'), 0, '2017-03-15 15:06:55', 'sys', '2022-05-05 13:50:07', 'sys', 0, 0);
```
