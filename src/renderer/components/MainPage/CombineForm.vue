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
  import BaseForm from './../BaseForm'
  import Ini from '@/services/ini'
  export default {
    components: { BaseForm },
    props: ['name'],
    data () {
      let inifile = './USU.ini'
      let that = this
      let fileconfig = Ini.loadFileSync(inifile)
      let dbsrc = fileconfig[that.name];
      let initdata = {
        formData: [{
          type: 'checkbox',
          name: 'ABKey',
          title: 'A+B组合',
          val: false
        }, {
          type: 'checkbox',
          name: 'BCKey',
          title: 'B+C组合',
          val: false
        }, {
          type: 'checkbox',
          name: 'CDKey',
          title: 'C+D组合',
          val: false
        }, {
          type: 'input',
          name: 'daley',
          title: '按键等待延时1-255(0.1s为单位)',
          val: '2'
        }, {
          type: 'select',
          name: 'buttonSendKey',
          title: '按键发送值',
          data: {0: '开启', 1: '关闭', 2: '反转', 3: '发送2bit固定值', 4: '发送1byte固定值'},
          val: 2
        }, {
          type: 'input',
          name: 'ABSendKey',
          title: 'A+B发送值',
          val: 0
        }, {
          type: 'input',
          name: 'BCSendKey',
          title: 'B+C发送值',
          val: 0
        }, {
          type: 'input',
          name: 'CDSendKey',
          title: 'C+D发送值',
          val: 0
        }],
        firstval:''
      }
       initdata.firstval=dbsrc
       dbsrc =JSON.parse(dbsrc)
        if(dbsrc){
             if(dbsrc["ABKey"]=="true"){
                dbsrc["ABKey"]=true
             }
             else if(dbsrc["ABKey"]=="false"){
                  dbsrc["ABKey"]=false
             }

              if(dbsrc["BCKey"]=="true"){
                 dbsrc["BCKey"]=true
              }
              else if(dbsrc["BCKey"]=="false"){
                   dbsrc["BCKey"]=false
              }

          if(dbsrc["CDKey"]=="true"){
              dbsrc["CDKey"]=true
           }
           else if(dbsrc["CDKey"]=="false"){
                dbsrc["CDKey"]=false
           }
             initdata.formData[0].val=dbsrc["ABKey"]
             initdata.formData[1].val=dbsrc["BCKey"]
             initdata.formData[2].val=dbsrc["CDKey"]
             initdata.formData[3].val=dbsrc["daley"]
             initdata.formData[4].val=parseInt(dbsrc["buttonSendKey"])
             initdata.formData[5].val=parseInt(dbsrc["ABSendKey"])
             initdata.formData[6].val=parseInt(dbsrc["BCSendKey"])
             initdata.formData[7].val=parseInt(dbsrc["CDSendKey"])
        }
      return initdata
    },
    methods: {
      save () {
        let config={}
        let refname=this.name
        let inifile = './USU.ini'
        let fs = require('fs');
        let fileconfig = Ini.loadFileSync(inifile)
        for(let i=0;i<8;i++){
            config[this.formData[i].name]=this.formData[i].val
        }
        let ABKey = this.formData[0].val
        let BCKey = this.formData[1].val
        let CDKey = this.formData[2].val
        let a = ABKey << 0 | BCKey << 1 | CDKey << 2
        let ds = [a, parseInt(this.formData[3].val), parseInt(this.formData[4].val), parseInt(this.formData[5].val), parseInt(this.formData[6].val), parseInt(this.formData[7].val), 0, 0, 0]
        let buttonSendKey = this.formData[4].val
        let dt = Device.CObject.UINT1
        if (buttonSendKey === '3') {
          dt = Device.CObject.UINT2
        } else if (buttonSendKey === '4') {
          dt = Device.CObject.UINT8
        }
        let os = []
        if (ABKey) {
          os.push(new Device.CObject(Device.CObject.CWT, dt, Device.CObject.BelongToCombine, 'ABKey',`41`))
        }
        if (BCKey) {
          os.push(new Device.CObject(Device.CObject.CWT, dt, Device.CObject.BelongToCombine, 'BCKey',`42`))
        }
        if (CDKey) {
          os.push(new Device.CObject(Device.CObject.CWT, dt, Device.CObject.BelongToCombine, 'CDKey',`43`))
        }
         let storage = window.localStorage
         let steptext=JSON.stringify(config)
         fileconfig[this.name] = steptext
         storage.setItem(this.name, steptext)
         fs.writeFileSync(inifile, fileconfig)
        //console.log(ds)
        this.$emit('submitFrameConfig', new Device.FrameConfig(this.name, ds, os))
      }
    }
  }
</script>

<style>
form { padding: 30px 15px; }
.form-footer { margin-top: 50px; }
</style>