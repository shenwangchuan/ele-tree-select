<!--下拉树el-select+el-tree -->
<template>
  <el-select ref="select"
             popper-class="TREE_SELECT_POPPER"
             :value="showLabel"
             :size="size"
             :placeholder="placeholder"
             :clearable="clearable"
             :disabled="disabled"
             :filterable="filterable" :filter-method="selectFilter"
             @visible-change="visibleChange"
             @focus="selectFocus"
             @clear="emitVal">
    <el-option class="option_li" :style="{height: optionHeight+'px'}" :value="showLabel" :label="showLabel">
      <el-scrollbar class="option_li_scroll">
        <el-tree ref="tree"
                 :accordion="accordion"
                 :data="options"
                 :props="props"
                 :node-key="props.value"
                 :show-checkbox="multiple"
                 :default-expand-all="defaultExpandAll"
                 :default-expanded-keys="defaultExpanded"
                 :default-checked-keys="(multiple && Array.isArray(value)) ? value : []"
                 :expand-on-click-node="false"
                 :check-strictly="checkStrictly"
                 :filter-node-method="treeFilter"
                 @node-click="nodeClick"
                 @check="nodeCheck"
        >
        </el-tree>
      </el-scrollbar>
    </el-option>
  </el-select>
</template>

<script type="es6">
  export default {
    name: "v-el-tree-select",
    props: {
      props: { //配置项
        type: Object,
        default() {
          return {
            value: 'value',
            label: 'label',
            children: 'children',
          }
        }
      },
      options: { //选项列表数据
        type: Array,
        default() {
          return []
        }
      },
      multiple: { // 是否可多选
        type: Boolean,
        default: false
      },
      value: { //绑定值
        type: [String, Number, Array],
        default() {
          return this.multiple ? [] : ''
        }
      },
      accordion: { //是否每次只展开一个同级树节点
        type: Boolean,
        default: false
      },
      size: String,
      filterable: { // 是否可搜索
        type: Boolean,
        default: false
      },
      clearable: { // 是否可清空
        type: Boolean,
        default: true
      },
      disabled: { // 是否禁用
        type: Boolean,
        default: false
      },
      placeholder: String,
      checkStrictly: { //父子是否不互相关联
        type: Boolean,
        default: false
      },
      defaultExpandAll: {
        type: Boolean,
        default: false
      },
      defaultExpandedKeys: {
        type: Array,
        default() {return []}
      }
    },
    data() {
      return {
        optionHeight: 247, //el-option高度
      }
    },
    computed: {
      showLabel() {
        let label = '';
        const value = this.value;
        if (this.multiple) { //多选
          if (Array.isArray(value) && value.length>0) {
            const labelArr = [];
            value.forEach(value=>{
              labelArr.push(this.queryTree(this.options, value))
            });
            label = labelArr.join(',')
          }
        } else { //单选
          if (value) {
            label = this.queryTree(this.options, value);
          }
        }
        return label;
      },
      // 默认展开
      defaultExpanded() {
        let keys = []
        if (this.value) {
          keys = (this.multiple && Array.isArray(this.value)) ? this.value : [this.value]
        } else {
          keys = this.defaultExpandedKeys
        }
        return keys
      }
    },
    methods: {
      // 搜索树状数据中的 ID,获取label
      queryTree(tree, id) {
        let stark = [];
        stark = stark.concat(tree);
        let label = '';
        while (stark.length) {
          const temp = stark.shift();
          if (temp[this.props.children]) {
            stark = stark.concat(temp[this.props.children]);
          }
          if (temp[this.props.value] === id) {
            label = temp[this.props.label];
          }
        }
        return label;
      },
      // 提交值
      emitVal(val) {
        if (!val) {
          val = this.multiple ? [] : ''
        }
        this.$emit('input', val)
      },
      // select框获得焦点
      selectFocus() {
        this.$refs.tree.filter('');
      },
      // 下拉框出现/隐藏
      visibleChange(show) {
        if (show) {
          this.$nextTick(()=>{
            const tree_H = this.$refs.tree.$el.clientHeight
            if (tree_H<this.optionHeight) {
              this.optionHeight = tree_H
            }
          })
        }
      },
      // select option过滤
      selectFilter(label) {
        this.$refs.tree.filter(label);
        return true
      },
      // 树过滤方法
      treeFilter(query, data) {
        if (!query) {
          return true
        } else {
          const labelArray = query.split(',');
          return labelArray.some(value => {
            return value && data[this.props.label].includes(value)
          })
        }
      },

      // 点击节点
      nodeClick(node) {
        if (!this.multiple) {
          this.emitVal(node[this.props.value])
          this.$refs.select.blur()  //使select失去焦点 隐藏下拉框
        }
      },
      // 点击复选框
      nodeCheck(node, data) {
        this.emitVal(data.checkedKeys)
      }
    },
  };
</script>

<style lang="scss" type="text/scss">
  .TREE_SELECT_POPPER>.el-scrollbar {
    >.el-scrollbar__bar.is-vertical {
      .el-scrollbar__thumb,.el-scrollbar__thumb:hover {
        background-color: transparent;
      }
    }
  }
</style>

<style scoped lang="scss" type="text/scss">
  .option_li {
    padding: 0;
    &.selected {
      font-weight: normal;
    }
    .option_li_scroll {
      height: 100%;
      /deep/ .el-scrollbar__wrap{
        overflow-x: hidden;
      }
    }
  }
  .el-tree {
    box-sizing: border-box;
    padding: 0 12px;
  }
</style>
