---
title: "运筹学实验报告"
published: 2023-11-22
updated: 2023-11-22
description: ""
category: "学习记录"
draft: false
---

1、	实验目的和任务                  
1.1. 进一步掌握Lingo编程操作；
1.2通过实验进一步掌握运筹学运输问题的建模以及求解过程，提高学生分析问题和解决问题能力。
2、	实验仪器、设备及材料 
计算机、Lingo
3、	实验内容
运输问题
3个产地4个销地的运输问题
	 
 
 
 
产量
 
6	2	6	7	30
 
4	9	5	3	25
 
8	8	1	5	21
销量	15	17	22	12	

建模
决策变量：决策变量就是产地 到销地 的运量 
目标函数：
 ，
约束条件：第i个产地的运出量应小于或等于该地的生产量，即
 
第j个销地的运入量应等于该地的需求量，即

 
求解过程
编写模型程序：
model:
! 3 Warehouse,4 Customer Transportation Problem;
sets:
  Warehouse/1..3/:a;
  Customer/1..4/:b;
  Routes(warehouse,customer):c,x;
endsets
!here are the parameters;
data:
a=30,25,21;
b=15,17,22,12;
c=6,2,6,7,
  4,9,5,3,
  8,8,1,5;
enddata
! The objective;
[obj] min=@sum(routes:c*x);
!The supply constraints;
@for(warehouse(i):[sup]@sum(customer(j):x(i,j))<=a(i));
!The demand constraints;
@for(customer(j):[dem] @sum(warehouse(i):x(i,j))=b(j));
end

使用LINGO软件计算6个发点8个收点的最小费用运输问题。产销单位运价如下表。
销地
产地	B1	B2	B3	B4	B5	B6	B7	B8	产量
A1	6	2	6	7	4	2	5	9	60
A2	4	9	5	3	8	5	8	2	55
A3	5	2	1	9	7	4	3	3	51
A4	7	6	7	3	9	2	7	1	43
A5	2	3	9	5	7	2	6	5	41
A6	5	5	2	2	8	1	4	3	52
销量	35	37	22	32	41	32	43	38	 

