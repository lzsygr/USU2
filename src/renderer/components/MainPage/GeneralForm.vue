<template>
  <form>
    <base-form :formData="formData"></base-form>
    <footer class="form-footer">
      <label>修改后请点击保存</label>
      <button type="button" class="form-control btn btn-form btn-primary" @click="save">保存</button>
    </footer>
  </form>
</template>

<script>
  import Device from '@/services/device'
  import Ini from '@/services/ini'
  import BaseForm from './../BaseForm'
  export default {
    components: { BaseForm },
    //props选项声明子组件要显示name
    props: ['name'],
    data () {
      let inifile = './USU.ini'
      let fileconfig = Ini.loadFileSync(inifile)
      let dbsrc = fileconfig[this.name]
      var initval={
                formData: [{
                  type: 'input',
                  name: 'daley',
                  title: '总线电压恢复后通讯延时2-255',
                  val: '2'
                }, {
                  type: 'checkbox',
                  name: 'numberLimitEnable',
                  title: '不限制报文数量',
                  val: true
                }, {
                  type: 'input',
                  name: 'numberLimitMax',
                  title: '单周期最大报文传输量1-255',
                  val: '20'
                }, {
                  type: 'select',
                  name: 'numberLimitPeriod',
                  title: '周期时间',
                  data: {
                    0: '50ms',
                    1: '100ms',
                    2: '200ms',
                    3: '500ms',
                    4: '1s',
                    5: '2s',
                    6: '5s',
                    7: '10s',
                    8: '30s',
                    9: '1m'
                  },
                  val: 7
                }, {
                  type: 'select',
                  name: 'transmitDays',
                  title: '阀值更新时间周期',
                  data: {
                    0: '7days',
                    1: '14days',
                    2: '30days'
                  },
                  val: 2
                }, {
                  type: 'select',
                  name: 'transmitPeriod',
                  title: '单次阀值更新时间',
                  data: {
                    0: '1min',
                    1: '2min',
                    2: '5min'
                  },
                  val: 1
                }],
                firstval:''
              }

               initval.firstval = dbsrc
               let resetdata =JSON.parse(dbsrc)
              initval.formData[0].val=resetdata["daley"]
              if(resetdata["numberLimitEnable"]=="true"){
                 resetdata["numberLimitEnable"] = true
              }
              else if(resetdata["numberLimitEnable"]=="false"){
                  resetdata["numberLimitEnable"] = false
             }
              initval.formData[1].val = resetdata["numberLimitEnable"]
              initval.formData[2].val = resetdata["numberLimitMax"]
              initval.formData[3].val = parseInt(resetdata["numberLimitPeriod"])
              initval.formData[4].val = parseInt(resetdata["transmitDays"])
              initval.formData[5].val = parseInt(resetdata["transmitPeriod"])
              return initval
      },
    methods: {
      save () {
        let that=this
        let d = [parseInt(this.formData[0].val), this.formData[1].val + 0, parseInt(this.formData[2].val), parseInt(this.formData[3].val), this.formData[4].val + 0, parseInt(this.formData[5].val)]
        let inifile = './USU.ini'
        let desp=[this.formData[0].name,this.formData[1].name,this.formData[2].name,this.formData[3].name,this.formData[4].name,this.formData[5].name]
        let fs = require('fs');
        let fileconfig = Ini.loadFileSync(inifile)
        let storage = window.localStorage
        let step1 = {}
        //因为USU.ini永远存在，所以节点永远存在，就算不存在，先删除也不会有问题
        //先删除，再创建，确保每次对象都是新的
        desp.forEach(function(value, index) {
             step1[value]=that.formData[index].val
         })
        let totext=JSON.stringify(step1)
        fileconfig[that.name] = totext
        fs.writeFileSync(inifile, fileconfig)
        storage.setItem(that.name,totext)
        // submitFrameConfig事件放在了父组件里面绑定，这里会调用父组件绑定的方法
        // this.name指明那个页面的配置，d是提交的配置数据
        this.$emit('submitFrameConfig', new Device.FrameConfig(this.name, d))
      }
    }
  }
</script>

<style>
form { padding: 30px 15px; }
.form-footer { margin-top: 50px; }
</style>