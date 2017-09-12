---
title: 【JS】导出图片和导出html
date: 2017-09-11 17:49:03
tags: 
  - js
categories: 
  - js
---

> 最近的项目中最过的一个需求，之前没遇到过，正好记录下来方便以后参考，实现方式并不是最好，不过暂时能完成需求，性能方便还未做过优化，先看着吧

<!-- more -->

由于是公司项目- -截图啥的我就不发了，这里说下核心功能实现

```html
<template>
  <button @click="exportToImage">导出为图片</button>
  <div id="html2canvas" ref="html2canvas">
    <!-- 代码 -->
  </div>
</template>

<script>
// 导入html2canvas
require('html2canvas/html2canvas.min');

export default {
  data() {
    return {}
  },
  methods: {
    clickToImage() {
      // 拿到当前的this执行环境
      var _this = this;
      // 将html转为图片
      html2canvas(this.refs.html2canvas)
        .then(canvas => {
          let imgUri = canvas.toDataURL('image/png').replace('image/png', 'image/octet-stream');
          // 下载图片
          let time = new Date().getTime();
          let $el = document.createElement('a');
          $el.href = imgUri;
          $el.target = '_blank';
          $el.download = `下载名_${time}.jpg`;
          $el.click();
          window.location.href = imgUri;
        })
    }
  }
}
</script>
```

---

下面再说另一种导出为html的方式

```html
<template>
  <button @click="exportToHTML">导出为HTML</button>
  <div id="html2canvas" ref="html2canvas">
    <!-- 代码 -->
  </div>
</template>

<script>
// 使用前需要 npm install file-saver --save-dev
import fileSaver from 'file-saver';

export default {
  data() {
    return {}
  },
  methods: {
    exportToHTML() {
      // 获取当前页面内容
      let record = document.getElementById('html2canvas').innerHTML;
      let time = new Date().getTime();
      const title = '我是title';
      const style = '这里是style我把需要的样式拼了过去，虽然手动麻烦了些，但导出内容一般样式都是一份，所以这样能节省导出文件大小';
      // 这里需要注意的是模版里的 / 需要转义~~ 第一次这里踩了坑弄半天没编译通过
      const head = `
        <head>
          <meta charset="UTF-8">
          <meta name="viewport" content="width=device-width, initial-scale=1.0">
          <meta http-equiv="X-UA-Compatible" content="ie=edge">
          <title>${title}<\/title>
          <style>
            ${style}
          <\/style>
        <\/head>
      `;
      const script = '同上，把需要的业务逻辑拼在了这里';
      const html = `
        <!DOCTYPE html>
        <html>
        ${head}
        <body>
          ${record}
          ${script}            
        <\/body>
        <\/html>
      `;

      // 最后保存文件
      let blob = new Blob([html], {type: 'text/html; charset=utf-8'});
      // 这个文件会自动下载到浏览器设定的下载目录下
      fileSaver.saveAs(blob, `${title}_${time}.html`);
    }
  }
}
</script>
```

这里再额外说一下，因为我导出的html是带有echarts图表的，图片不方便查看一些细节，所以需求里加了html，这里导出script时需要注意下：

```js
// echarts相关字符串拼接
let dataString = JSON.stringify(this.echartsData);
const script = `
  <script>
    // 这里把echarts.min.js的内容写入即可，或者也可根据用到的组件按需加载，这样体积小些    
  <\/script>
  <script>
    var charts = echarts.init(document.getElementById('charts'));
    var data = ${dataString}
    let options = {
      tooltip: {
        trigger: 'item',
        formatter: "{a} <br/>{b}: {c} ({d}%)"
      },
      legend: {
        orient: 'horizontal',
        x: 'center',
        y: 520,
        data: data.levels
      },
      series: [{                  
          name: '名称',
          type: 'pie',
          selectedMode: 'single',
          radius: [0, '30%'],
          label: {
            normal: {
              position: 'inner'
            }
          },
          labelLine: {
            normal: {
              show: false
            }
          },
          data: data.levels                 
        },
        {
          name: '名称',
          type: 'pie',
          radius: ['40%', '55%'],
          data: data.seriesData
        }
      ]
    }
    charts.setOption(options);
    if (null != charts) {
      charts.resize();
    }
    window.onresize = function() {
      throttle(charts.resize)
    }
    function throttle(method, context) {
      clearTimeout(method.tId);
      method.tId = setTimeout(function () {
        method.call(context);
      }, 200);
    }
  <\/script>
`
```

这样导出图片和html的功能就实现了