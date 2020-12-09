<template>
  <div :class="['json-view-container',theme,`deep-${currentDeep}`,showShadow?'show-shadow':'']">
    <div
      v-if="!showError"
      :class="['json-view', length ? 'closeable' : '']"
      :style="{fontSize:fontSize+'px',lineHeight:lineHeight+'px',paddingLeft:indent+'px'}">
      <!--indent-fontSize)/2+3 中的3，和base-line样式中的left一致-->
      <span @click="toggleClose" class="json-angle" v-if="length" :style="{left:((indent-fontSize)/2+3)+'px'}">
          <slot name="closedIcon" v-if="innerclosed">
            <svg :fill="iconColors[0]" viewBox="0 0 1792 1792" style="vertical-align: middle; height: 1em; width: 1em;">
              <path
                d="M1344 800v64q0 14-9 23t-23 9h-352v352q0 14-9 23t-23 9h-64q-14 0-23-9t-9-23v-352h-352q-14 0-23-9t-9-23v-64q0-14 9-23t23-9h352v-352q0-14 9-23t23-9h64q14 0 23 9t9 23v352h352q14 0 23 9t9 23zm128 448v-832q0-66-47-113t-113-47h-832q-66 0-113 47t-47 113v832q0 66 47 113t113 47h832q66 0 113-47t47-113zm128-832v832q0 119-84.5 203.5t-203.5 84.5h-832q-119 0-203.5-84.5t-84.5-203.5v-832q0-119 84.5-203.5t203.5-84.5h832q119 0 203.5 84.5t84.5 203.5z">
              </path>
            </svg>
          </slot>
          <slot name="openedIcon" v-else>
            <svg :fill="iconColors[1]" viewBox="0 0 1792 1792" style="vertical-align: middle;height: 1em; width: 1em;">
              <path
                d="M1344 800v64q0 14-9 23t-23 9h-832q-14 0-23-9t-9-23v-64q0-14 9-23t23-9h832q14 0 23 9t9 23zm128 448v-832q0-66-47-113t-113-47h-832q-66 0-113 47t-47 113v832q0 66 47 113t113 47h832q66 0 113-47t47-113zm128-832v832q0 119-84.5 203.5t-203.5 84.5h-832q-119 0-203.5-84.5t-84.5-203.5v-832q0-119 84.5-203.5t203.5-84.5h832q119 0 203.5 84.5t84.5 203.5z">
              </path>
            </svg>
          </slot>
        </span>
      <div class="content-wrap">
        <div :class="['first-line',length>0?'pointer':'']" @click="toggleClose">
          <div v-if="jsonKey" class="json-key" :style="{textAlign: labelAlign}">
            <span v-show="showQuote">"</span>
            <span class="json-key-label" :style="{width: labelWidth}" :title="jsonKey">{{jsonKey}}</span>
            <span v-show="showQuote">"</span>
            <span>:</span>
          </div>
          <div v-if="length">{{prefix}}{{innerclosed ? ('...' + subfix) : ''}}
            <span class="json-note">{{innerclosed ? '// '+length+(isArray?' items':' properties') : ''}}</span>
          </div>
          <span v-if="!length">{{`${isArray ? '[]' : '{}'}${isLast?'':','}`}}</span>
        </div>
        <div v-if="!innerclosed && length" class="json-body">
          <template v-for="(item, index) in items">
            <json-view
              v-if="item.isJSON"
              :closed="isClose()"
              :key="index"
              :data="item.value"
              :jsonKey="item.key"
              :currentDeep="templateDeep + 1"
              :deep="deep"
              :theme="theme"
              :fontSize="fontSize"
              :lineHeight="lineHeight"
              :iconColor="iconColors"
              :isLast="index === items.length - 1"
              :hasSiblings="item.hasSiblings"
              :showValue="showValue"
              :showQuote="showQuote"
              :labelWidth="labelWidth"
              :labelAlign="labelAlign"
            >
              <template slot="suffix" slot-scope="scope">
                <slot name="suffix" :data="scope.data"></slot>
              </template>
              <template slot="closedIcon">
                <slot name="closedIcon"></slot>
              </template>
              <template slot="openedIcon">
                <slot name="openedIcon"></slot>
              </template>
            </json-view>
            <!--叶子属性-->
            <div class="json-item" v-else :key="index" :style="{paddingLeft: indent+'px'}">
              <!--{{(isArray ? '' : '"' + item.key + '":')}}-->
              <div class="json-key" :style="{textAlign: labelAlign}">
                <span v-show="showQuote">"</span>
                <span class="json-key-label" :style="{width: labelWidth}" :title="item.key">{{item.key}}</span>
                <span v-show="showQuote">"</span>
                <span>:</span>
              </div>
              <div :class="['json-value',getDataType(item.value)]" v-if="showValue">
                {{`${getDataType(item.value)==='string'?'"':''}${item.value}${getDataType(item.value)==='string'?'"':''}${index ===items.length - 1 ? '' : ','}`}}
              </div>
              <div class="json-extra" v-if="$scopedSlots.suffix"><slot name="suffix" :data="item"></slot></div>
            </div>
          </template>
          <span v-if="showLine&&!innerclosed" class="base-line"></span>
        </div>
        <p v-if="!innerclosed " class="last-line">
          <span>{{subfix}}</span>
        </p>
      </div>
    </div>
    <div class="json-error" v-else>
      <img src="./images/error.png" alt="">
      <p>JSON格式错误！</p>
    </div>
  </div>
</template>
<script>
  export default {
    name: 'JsonView',
    components: {},
    props: {
      data: { // 传入的json数据
        type: [Object, Array],
        required: true
      },
      showError: { // 是否格式错误
        type: Boolean,
        default: false
      },
      jsonKey: { // json的key值，用于第二层及二层以上的组件的key值
        type: String,
        default: ''
      },
      closed: { // 是否折叠
        type: Boolean,
        default: false
      },
      showLine: { //是否显示连线
        type: Boolean,
        default: true
      },
      showValue: { //是否显示value
        type: Boolean,
        default: true
      },
      labelWidth: { //key的宽度值
        type: String,
        default: 'auto'
      },
      labelAlign: { //key的对齐方式，只有当labelWidth不为auto的时候生效
        type: String,
        default: 'left'
      },
      showShadow: { //是否显示内阴影
        type: Boolean,
        default: false
      },
      showQuote: { //是否显示key的引号
        type: Boolean,
        default: true
      },
      isLast: { //是否是最后一行
        type: Boolean,
        default: true
      },
      indent: { //是否显示value
        type: Number,
        default: 25
      },
      fontSize: { //字体大小
        type: Number,
        default: 14
      },
      lineHeight: { //行高
        type: Number,
        default: 24
      },
      deep: { // 展开深度
        type: Number,
        default: 3
      },
      currentDeep: { // 当前为递归的第几层
        type: Number,
        default: 1
      },
      // iconStyle: { // 折叠icon样式
      //   type: String,
      //   default: 'square'
      // },
      iconColor: { //icon颜色
        type: Array,
        default() {
          return []
        }
      },
      theme: { // 主题
        type: String,
        default: ''
      },
      hasSiblings: { // 是否有兄弟节点
        type: Boolean,
        default: true
      }
    },
    data() {
      return {
        innerclosed: this.closed,
        templateDeep: this.currentDeep,
        visible: false
      }
    },
    computed: {
      isArray() {
        return this.getDataType(this.data) === 'array'
      },
      length() {
        return this.isArray ? this.data.length : Object.keys(this.data).length
      },
      subfix() {
        const data = this.data;
        if (this.isEmptyArrayOrObject(data)) { // 如果是空数组或空对象
          return ''
        } else {
          return (this.isArray ? ']' : '}') + (this.isLast ? '' : ',')
        }
      },
      prefix() {
        return this.isArray ? '[' : '{'
      },
      items() {
        const json = this.data;

        if (this.isArray) {
          return json.map(item => {
            const isJSON = this.isObjectOrArray(item)
            return {
              value: item,
              isJSON,
              key: ''
            }
          })
        }
        return Object.keys(json).map(key => {
          const item = json[key]
          const isJSON = this.isObjectOrArray(item)
          return {
            value: item,
            isJSON,
            key
          }
        })
      },
      iconColors() {
        const {theme, iconColor} = this
        if (iconColor.length === 2) {
          return iconColor
        } else {
          if (theme === 'one-dark') {
            return ['#747983', '#747983']
          } else if (theme === 'vs-code') {
            return ['#c6c6c6', '#c6c6c6']
          } else {
            return ['#747983', '#747983']
          }
        }
      }
    },
    mounted() {
      // 老代码：收起22ms，
      this.$nextTick(() => {
        // this.visible = true
        // console.log(new Date().getTime())
      });
    },
    methods: {
      getDataType(data) {
        return Object.prototype.toString.call(data).slice(8, -1).toLowerCase()
      },
      isObjectOrArray(source) {
        return ['array', 'object'].includes(this.getDataType(source))
      },
      toggleClose() {
        const a = new Date().getTime()
        if (this.length === 0) {
          return
        }
        this.innerclosed = !this.innerclosed;
        this.$nextTick(() => {
          // this.visible = true
          console.log((new Date().getTime()) - a)
        });
      },
      isClose() {
        return this.templateDeep + 1 > this.deep
      },
      isEmptyArrayOrObject(data) { // 空数组或者空对象
        return [{},
          []
        ].map(item => JSON.stringify(item)).includes(JSON.stringify(data))
      },
    },
    watch: {
      closed() {
        this.innerclosed = this.closed
      }
    }
  }
</script>
<style scoped lang="less">
  @import "./style/index";
  @import "./style/one-dark";
  @import "./style/vs-code";
</style>
