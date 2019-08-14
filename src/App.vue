<template>
  <div id="app">
    <el-form :model="form" style="width: 500px">
      <el-form-item label="单选：">
        <TreeSelect v-model="form.radioV" :options="allArea" :props="areaProps"></TreeSelect>
      </el-form-item>
      <el-form-item label="多选：">
        <TreeSelect v-model="form.multiV" :options="allArea" :props="areaProps" multiple></TreeSelect>
      </el-form-item>
      <el-form-item>
        <el-button @click="onSubmit">提交</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
  import TreeSelect from './components/TreeSelect'
export default {
  name: 'app',
  components: {TreeSelect},
  data() {
    return {
      form: {
        radioV:'',
        multiV: [2]
      },
      allArea: [],
      areaProps: {
        label: 'area_name',
        value: 'area_code',
        children: 'sub'
      }
    }
  },
  created() {
    this.getJsonData()
  },
  methods: {
    getJsonData() {
      this.$axios.get('json/area.json').then(r=>{
        if (r.status === 200 &&  r.data.content) {
          this.allArea = r.data.content
        }
      })
    },
    onSubmit() {
      console.log(this.form)
    }
  }
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  padding: 20px;
}
</style>
