---
title: "Vue3使用Echarts"
published: 2024-08-01
updated: 2024-09-20
description: ""
tags: ["Vue3","Echarts"]
category: "前端相关"
draft: false
---

**在vue3中** 

       <template>
          <div id=\"main\" style=\"height: 500px;width: 500px;\"></div>
        </template>
        
        <script setup>
        import * as echarts from \'echarts\';
        import { onMounted,onUnmounted } from \'vue\';
        onMounted(() => {
          initChart();
        });
        
        onUnmounted(() => {
          myChart.dispose();// 调用dispose方法释放echarts实例
        });
        // 基于准备好的dom，初始化echarts实例
        const initChart = () => {
          let myChart = echarts.init(document.getElementById(\'main\'));
        // 绘制图表
        myChart.setOption({
          title: {
            text: \'ECharts 入门示例\'
          },
          tooltip: {},
          xAxis: {
            data: [\'衬衫\', \'羊毛衫\', \'雪纺衫\', \'裤子\', \'高跟鞋\', \'袜子\']
          },
          yAxis: {},
          series: [
            {
              name: \'销量\',
              type: \'bar\',
              data: [5, 20, 36, 10, 10, 20]
            }
          ]
        });
        }
        </script>

注意：Echarts必须在DOM节点加载后才能渲染，也就是必须在onMounted中调用
官网：https://echarts.apache.org/zh/index.html