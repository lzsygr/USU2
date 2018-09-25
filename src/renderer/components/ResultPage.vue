<template>
<div id="result" class="window">
  <form>
    <div class="form-merge">
    <div class="form-merge-item">
            <div class="form-group">
              <label>备注</label>
              <input v-model="place" type="text" class="form-control">
            </div>
          </div>

      <div class="form-merge-item">
        <div class="form-group">
          <label>区(0-15)</label>
          <input v-model="darea" type="text" class="form-control">
        </div>
      </div>
      <div class="form-merge-item">
        <div class="form-group">
          <label>线路(0-15)</label>
          <input v-model="dline" type="text" class="form-control">
        </div>
      </div>
      <div class="form-merge-item">
        <div class="form-group">
          <label>总线设备(0-255)</label>
          <input v-model="dbus"type="text" class="form-control">
        </div>
      </div>
      <div class="form-merge-item">
        <div class="form-group">
          <label>预览</label>
          <p>{{dd}}</p>
        </div>
      </div>
    </div>
    <div class="form-merge">
      <div class="form-merge-item" style="display:none">
        <div class="form-group">
          <label>串口</label>
          <input v-model="port" type="text" class="form-control">
        </div>
      </div>
      <div class="form-merge-item">
        <div class="form-group">
          <label>确认写入</label>
          <button @click="write" type="button" class="form-control btn btn-primary">写入</button>
        </div>
      </div>
      <div class="form-merge-item">
        <div class="form-group">
          <label>&nbsp;</label>
          <button @click="restartDevice" type="button" class="form-control btn btn-warning">下载</button>
        </div>
      </div>
    </div>
    <div v-for="item in items" class="form-group">
      <label>{{item.name}}</label>
      <input :value="item | dump" type="text" class="form-control">
    </div>

     <footer class="form-footer">
          <button type="button" class="form-control btn btn-primary" @click="reset">返回首页</button>
        </footer>
  </form>
  <console-module :lines="consoleLines" @closeConsole="receivedCloseFromClild" v-if="showModel"></console-module>
</div>
</template>

<script>
  import Device from '@/services/device'
  import ConsoleModule from './ResultPage/ConsoleModule'
  import Ini from '@/services/ini'
  let fs = require('fs')
  let path = require('path')
  let storage = window.localStorage
  export default {
    components: { ConsoleModule },
    data () {
      let initdata = {
             darea: 1,
             place:'客厅',
             dline: 1,
             dbus: 13,
             items: [],
             port: 'COM3',
             showModel: false,
             consoleLines: [],
             ordown: false
           }
      let adjustflag = storage.getItem("file")
      if(adjustflag){
        adjustflag = adjustflag.split('/').pop()
        let giveval = adjustflag.split('.')
        initdata.place = giveval[0]
        initdata.darea = giveval[1]
        initdata.dline = giveval[2]
        initdata.dbus = giveval[3]
      }
      return initdata
  },
    computed: {
      dd: function () {
        let addr0 = parseInt(this.darea) << 4 | parseInt(this.dline)
        let addr1 = parseInt(this.dbus)
        Device.manager.setAddr(addr0, addr1)
        this.items = Device.manager.dump()
        return ['0x' + addr0.toString(16), '0x' + addr1.toString(16)].join(' ')
      }
    },
    mounted: function () {
      this.items = Device.manager.dump()
      console.log(this.items)
    },
    methods: {
      reset: function () {
        this.$router.replace('/')
      },
    prev () {
        this.$router.push('/grouping')
     },
      receivedCloseFromClild: function () {
        this.showModel = false
   },
      restartDevice: function () {
       if(this.ordown){
          const {shell} = require('electron')
          var path1 = `"${path.join('./')}download.exe"`
          shell.openItem(path1)
          this.ordown = false
       }else{
          alert('必须写入后，才能下载')
       }
      },
      write: function () {
        let self=this;
        let filename = `${self.place}.${self.darea}.${self.dline}.${self.dbus}.ini`
        let pathdir='./data'
        let dbfilename = pathdir + '/' + filename
        let inifile = './USU.ini'
        let fs = require('fs')
        let fileconfig = Ini.loadFileSync(inifile)
        fileconfig.file = filename
        fileconfig.removeSection("download")
        let finalRes = fileconfig.getOrCreateSection("download")
        self.items.forEach((item) => {
            finalRes[item.name] = item.dump()
        })


          let dbsrc7 = JSON.parse(storage.getItem('alldevice'))
          let objnum=0;
          dbsrc7.forEach(function(val, index) {
               objnum = index+1
               let curindex = `obj${objnum}`
               fileconfig[curindex] = JSON.stringify(val)
           });
            fileconfig.objnum= objnum

         if(!Ini.CheckExistsSync(pathdir)){
                 fs.mkdirSync(pathdir)
         }

         fs.writeFile(inifile, fileconfig, (err) => {
              if (err) throw err;
              Ini.fileCopy(inifile,dbfilename)
              alert('文件写入成功')
              self.ordown = true
              storage.removeItem("file")
            });
      }
    },
    filters: {
      dump: function (d) {
        return d.dump()
      }
    }
  }
</script>

<style type="text/css">
  #result form { padding: 30px 15px; box-sizing: border-box; width: 100%; height: 100%; overflow-y: auto; }
  .form-merge { display: flex; }
  .form-merge .form-merge-item { flex: 1; padding: 10px; }
</style>