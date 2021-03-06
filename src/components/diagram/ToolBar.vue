<template>
  <div class="header-container">
    <i title="创建 diagram" class="app-icon app-icon-new"></i>
    <i title="打开 diagram" class="app-icon app-icon-open" @click="$refs.refFile.click()"></i>|
    <i title="保存 diagram" class="app-icon app-icon-save" @click="saveXml"></i>
    <i title="导出 diagram" class="app-icon app-icon-save-as" @click="saveBPMN"></i>|
    <i title="复原" class="app-icon app-icon-undo" @click="redo"></i>
    <i title="撤销" class="app-icon app-icon-redo" @click="undo"></i>|
    <i title="导出为图片" class="app-icon app-icon-picture" @click="saveSVG"></i>|
    <i :class="{open: isOpenColor}" title="设置元素颜色" class="app-icon app-icon-set-color-tool" @click="openColor">
      <i class="el-icon-caret-bottom"></i>
      <ul class="colorSelect" v-show="isOpenColor">
        <li v-for="c in COLORS" :key="c.title">
          <div class="colorBlock" :title="c.title" :style="{borderColor: c.stroke, backgroundColor: c.fill}" @click="setColor(c.fill, c.stroke)"></div>
        </li>
      </ul>
    </i>|
    <i title="元素左对齐" class="app-icon app-icon-align-left-tool" @click="alignElements('left')"></i>
    <i title="元素垂直居中对齐" class="app-icon app-icon-align-center-tool" @click="alignElements('center')"></i>
    <i title="元素右对齐" class="app-icon app-icon-align-right-tool" @click="alignElements('right')"></i>
    <i title="元素顶部对齐" class="app-icon app-icon-align-top-tool" @click="alignElements('top')"></i>
    <i title="元素水平居中对齐" class="app-icon app-icon-align-middle-tool" @click="alignElements('middle')"></i>
    <i title="元素底部对齐" class="app-icon app-icon-distribute-vertical-tool" @click="alignElements('bottom')"></i>|
    <i title="水平分布元素" class="app-icon app-icon-distribute-horizontal-tool" @click="distributeElements('horizontal')"></i>
    <i title="垂直分布元素" class="app-icon app-icon-distribute-vertical-tool" @click="distributeElements('vertical')"></i>

    <!-- <i title="放大" class="app-icon el-icon-zoom-in" @click="handlerZoom(0.1)"></i>
    <i title="缩小" class="app-icon el-icon-zoom-out" @click="handlerZoom(-0.1)"></i> -->

    <input type="file" id="files" ref="refFile" style="display: none" @change="loadBPMN" />

  </div>
</template>

<script>

const COLORS = [{
  title: 'White',
  fill: 'white',
  stroke: 'black'
}, {
  title: 'Blue',
  fill: 'rgb(187, 222, 251)',
  stroke: 'rgb(30, 136, 229)'
}, {
  title: 'Orange',
  fill: 'rgb(255, 224, 178)',
  stroke: 'rgb(251, 140, 0)'
}, {
  title: 'Green',
  fill: 'rgb(200, 230, 201)',
  stroke: 'rgb(67, 160, 71)'
}, {
  title: 'Red',
  fill: 'rgb(255, 205, 210)',
  stroke: 'rgb(229, 57, 53)'
}, {
  title: 'Purple',
  fill: 'rgb(225, 190, 231)',
  stroke: 'rgb(142, 36, 170)'
}]
export default {
  props: {
    bpmnModeler: Object,
    xml: String,
    elementSelector: Array,
    scale: Number
  },
  data () {
    return {
      color: '',
      isOpenColor: false,
      activeName: 'first',
      COLORS
    }
  },
  methods: {
    redo () {
      this.bpmnModeler.get('commandStack').redo()
    },
    undo () {
      this.bpmnModeler.get('commandStack').undo()
    },

    openColor () {
      this.isOpenColor = !this.isOpenColor
    },

    setColor (fill, stroke) {
      const modeling = this.bpmnModeler.get('modeling')
      this.elementSelector.map(element => {
        modeling.setColor(element, {
          fill,
          stroke
        })
      })
    },

    // left/top/right/bottom/cneter/middle
    alignElements (position = 'center') {
      const alignElements = this.bpmnModeler.get('alignElements')
      alignElements.trigger(this.elementSelector, position)
    },

    // horizontal/vertical
    distributeElements (axis) {
      const alignElements = this.bpmnModeler.get('distributeElements')
      alignElements.trigger(this.elementSelector, axis)
    },

    handlerZoom (radio) {
      const newScale = !radio ? 1.0 : this.scale + radio
      this.bpmnModeler.get('canvas').zoom(newScale)
      this.$emit('scale', newScale)
    },

    async saveXml () {
      const res = await this.bpmnModeler.saveXML({ format: true })
      this.$emit('updateXml', res.xml)
      this.$message.success('保存成功')
    },

    async saveBPMN () {
      try {
        const result = await this.bpmnModeler.saveXML({ format: true })
        const { xml } = result

        const xmlBlob = new Blob([xml], {
          type: 'application/bpmn20-xml;charset=UTF-8,'
        })

        const downloadLink = document.createElement('a')
        downloadLink.download = `bpmn-${+new Date()}.bpmn`
        downloadLink.innerHTML = 'Get BPMN SVG'
        downloadLink.href = window.URL.createObjectURL(xmlBlob)
        downloadLink.onclick = function (event) {
          document.body.removeChild(event.target)
        }
        downloadLink.style.visibility = 'hidden'
        document.body.appendChild(downloadLink)
        downloadLink.click()
      } catch (err) {
        console.log(err)
      }
    },

    async loadBPMN () {
      const that = this
      const file = this.$refs.refFile.files[0]

      const reader = new FileReader()
      reader.readAsText(file)
      reader.onload = function () {
        that.$emit('updateXml', this.result)
      }
    },

    async saveSVG () {
      try {
        const result = await this.bpmnModeler.saveSVG()
        const { svg } = result

        const svgBlob = new Blob([svg], {
          type: 'image/svg+xml'
        })

        const downloadLink = document.createElement('a')
        downloadLink.download = `bpmn-${+new Date()}.SVG`
        downloadLink.innerHTML = 'Get BPMN SVG'
        downloadLink.href = window.URL.createObjectURL(svgBlob)
        downloadLink.onclick = function (event) {
          document.body.removeChild(event.target)
        }
        downloadLink.style.visibility = 'hidden'
        document.body.appendChild(downloadLink)
        downloadLink.click()
      } catch (err) {
        console.log(err)
      }
    }
  }
}
</script>

<style lang="less" scoped>
.header-container {
  user-select: none;
  .app-icon {
    font-size: 14px;
    padding: 5px;
    margin-right: 4px;
    width: 14px;
    height: 16px;
    cursor: pointer;
    border: 1px solid rgba(1, 1, 1, 0);

    &:hover {
      background-color: #f8f8f8;
      border: 1px solid #ccc;
    }
  }

  .app-icon-set-color-tool {
    position: relative;
    .colorBlock {
      margin: 4px;
      width: 56px;
      height: 16px;
      border: 1px solid;
    }

    &.open {
      background-color: #f8f8f8;
      border: 1px solid #ccc;
    }
  }
  .colorSelect {
    z-index: 10;
    position: absolute;
    left: 0;
    list-style: none;
    transform: translateX(-50%);
  }
}
</style>
