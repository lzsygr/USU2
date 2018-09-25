<template>
<div id="grouping" class="window">
	<section class="objs">
	  <div class="table">
	  <header class="table-header">
	  <span class="t0">序号</span>
	  <span class="t1">属于</span>
      <span class="t2">类型</span>
      <span class="t1">配置字</span>
      <span class="t1">数据类型</span>
	  <span class="t3">Group Addr</span>
      <span class="t0">操作</span>
		</header>
		<section class="table-body">
			<ul>
				<li v-for="obj in objs">
					<span class="t0">{{obj.id}}</span>
					<span class="t1">{{obj.belongTo | channel}}</span>
          <span class="t2">{{obj.comment}}</span>
					<span class="t1">{{obj.mixedKey | stringMixedKey}}</span>
          <span class="t1">{{obj.dataType | stringDataType}}</span>
					<span class="t3">{{obj | groupAddrs}}</span>
          <span class="t0"><i @click="addObjToGroup(obj.id)" class="icon icon-plus-circled"></i></span>
				</li>
			</ul>
		</section>
	  </div>
	</section>
	<section class="groups">
   <aside>
     <p><i @click="addGroup(0)"class="icon icon-plus-circled"></i> 群组</p>
     <ul>
        <template v-for="group in groups">
         <li :class="{active: activedId === group.id}" @click="activeGroup(group.id)">
          <div class="r"><i @click="addGroup(group.id)" v-if="group.level<3" class="icon icon-plus-circled"></i> <i @click="delGroup(group.id)" class="icon icon-cancel-circled"></i></div>
          <template v-for="i in group.level">
            <span class="indent"></span>
          </template>
          <span class="icon icon-doc-text"></span> {{group.value}} {{group.name}}
         </li>
        </template>
     </ul>
   </aside>
   <div class="groups2objs">
    <div class="table">
    <header class="table-header">
      <span class="t0">Index</span>
      <span class="t1">Channel</span>
      <span class="t2">Type</span>
      <span class="t0">操作</span>
    </header>
    <section class="table-body">
      <ul>
        <li v-for="obj in activedGroupIncludes">
          <span class="t0">{{obj.id}}</span>
          <span class="t1">{{obj.belongTo | channel}}</span>
          <span class="t2">{{obj.comment}}</span>
          <span class="t0"><i @click="removeObjFromGroup(obj.id)" class="icon icon-cancel-circled"></i></span>
        </li>
      </ul>
    </section>
    </div>
   </div>
  </section>
  <group-module :parentGroup="activedGroup" @submitGroupModel="receivedSubmitFromClild" @cancelGroupModel="receivedCancelFromClild" v-if="showModel"></group-module>
  <footer class="toolbar toolbar-footer">
    <div class="toolbar-actions">
      <button @click="prev" class="btn btn-default">
        上一步
      </button>

      <button @click="save" class="btn btn-primary pull-right">
        烧录
      </button>
    </div>
  </footer>
</div>
</template>

<script>
  import Device from '@/services/device'
  import Ini from '@/services/ini'
  import GroupModule from './GroupingPage/GroupModule'
  export default {
    components: { GroupModule },
    data () {
          let initdata = {
                showModel: false,
                parentId: 0,
                activedId: 0,
                //处于激活状态的群组
                activedGroup: null,
                objs: Device.CObject.All,
                //当前所有有意义的群组
                groups: [],
                //绑定给活动群组的所有通讯对象
                activedGroupIncludes: []
          }
          return initdata
        },
        mounted () {
          Device.CGroup.Clean()
          let that = this
          let inifile = './USU.ini'
          let fs = require('fs');
          let fileconfig = Ini.loadFileSync(inifile)
          let groupnum = parseInt(fileconfig.groupnum)
          if(groupnum>0){
                for (let i=1; i<=groupnum;i++) {
                         let dbsrc = JSON.parse(fileconfig[`group${i}`])
                         Device.CGroup.Id = parseInt(dbsrc.id)
                         new Device.CGroup(parseInt(dbsrc.parentId),dbsrc.name, parseInt(dbsrc.value))
                   }
            this.groups = Device.CGroup.Dump()
            let adjustarr=[]
            let setindex=1
            this.groups.forEach(function (g) {
                  if (g.level !== 3) {
                    return
                  }
                  adjustarr[setindex]=g.id
                  setindex++
             })


            let existobj=[]
            for (var sobj of this.objs) {
                existobj.push(sobj.id)
            }

            let objtog= fileconfig.link || ''
            if(objtog){
                let linkarr = objtog.split(',')
                let mixreset = []
                for(let j=0,count=linkarr.length;j<count;j=j+2){
                    let tmpid = parseInt(linkarr[j+1])
                    let tmpgid = parseInt(linkarr[j])
                    if(existobj.includes(tmpid)){
                        let tmp=`${adjustarr[tmpgid]}-${tmpid}`
                        mixreset.push(tmp)
                    }
                }
                if(mixreset.length>0){
                    mixreset.sort()
                    let first=mixreset[0].split('-')
                    let curgr=parseInt(first[0])
                    that.activeGroup(curgr)
                    for (var back of mixreset) {
                       let tmpval = back.split('-')
                       let loopgrp=parseInt(tmpval[0])
                       if(loopgrp!=curgr){
                            that.activeGroup(loopgrp)
                       }
                       that.addObjToGroup(parseInt(tmpval[1]))
                    }
                }
            }
          }else{
                new Device.CGroup(0, '主群租', 0)
                new Device.CGroup(1, '中间组', 0)
                new Device.CGroup(2, '子群组', 0)
                //初始化this.groups为除顶级群组外的所有群组
                this.groups = Device.CGroup.Dump()
                //初始化顶级群组为当前活动群组，即id=0的群组为当前活动群组
                //初始化this.activedId为0，this.activedGroup为id=0的组对象，this.activedGroupIncludes为对象的项
                this.activeGroup(0)
         }
      },
    methods: {
      prev () {
        this.$router.replace('/')
      },
      save () {
        let groups = []
        let index = 1
        let that = this
        let inifile = './USU.ini'
        let fs = require('fs');
        let fileconfig = Ini.loadFileSync(inifile)
        //给所有的子群组添加唯一编号并保存到groups数组中
        let gnum = 0
        let groupsave = []
        this.groups.forEach(function (g) {
           gnum++
           let groupkey=`group${gnum}`
           let ginfo={}
           ginfo.id = g.id
           ginfo.name = g.name
           ginfo.value = g.value
           ginfo.parentId = g.parentId
           ginfo.level = g.level
           groupsave.push(ginfo)
           fileconfig[groupkey] = JSON.stringify(ginfo)
          if (g.level !== 3) {
            return
          }
          g.index = index
          index++
          groups.push(g)
        })
        fileconfig.groupnum = gnum
        let steptext=JSON.stringify(groupsave)
        let storage = window.localStorage
        storage.setItem('savegroups',steptext)

        let ds = []
        groups.forEach(function (g) {
          //组合成新数组返回给ds
          //g.codeAddr()对加了唯一编号的子群组的地址的处理值，返回包含两个元素的数组
           ds = ds.concat(g.codeAddr())
        })
        this.receivedFromClild(new Device.FrameConfig('group', ds))

        //group2objs
        let group2objs = []
        Device.manager.allCObjects().forEach(function (obj) {
          obj.groups().forEach(function (g) {
            group2objs = group2objs.concat([g.index, obj.id])
          })
        })
        console.log('group2objs', group2objs)
        fileconfig.link = group2objs.join(',')
        fs.writeFileSync(inifile, fileconfig)
        this.receivedFromClild(new Device.FrameConfig('group2objs', group2objs))
        // next
        this.$router.replace('/result')
      },
      addGroup: function (parentId) {
        this.parentId = parentId
        this.showModel = true
      },
      delGroup: function (id) {
        Device.CGroup.RemoveGroupById(id)
        // force reload
        this.reload()
      },
      activeGroup: function (id) {
        this.activedId = id
        this.activedGroup = Device.CGroup.FindGroupById(Device.CGroup.Root, id)
        this.activedGroupIncludes = this.activedGroup ? this.activedGroup.includeObjs() : []
      },
      addObjToGroup: function (objId) {
        if (this.activedGroup.level === 3) {
          this.activedGroup.addObj(objId)
          this.activeGroup(this.activedId)
        }
      },
      removeObjFromGroup: function (objId) {
        this.activedGroup.removeObj(objId)
        this.activeGroup(this.activedId)
      },
      reload: function () {
        this.activedId = 0
        this.groups = Device.CGroup.Dump()
        this.activeGroup(this.activedId)
      },
      receivedSubmitFromClild: function (msg) {
        this.receivedCancelFromClild()
        new Device.CGroup(this.parentId, msg.name, msg.addr)
        this.groups = Device.CGroup.Dump()
        // force reload
        this.activedId = 0
        this.reload()
        console.log(Device.CGroup.Root)
      },
      receivedFromClild (msg) {
        Device.manager.update(msg.name, msg)
        console.log('receivedFromClild', msg)
        msg.dump()
      },
      receivedCancelFromClild: function () {
        this.showModel = false
      }
    },
    filters: {
      channel: function (index) {
        return ['channelA', 'channelB', 'channelC', 'channelD', '组合按键'][index]
      },
      stringDeviceType: function (index) {
        return ['-', '开关', '开关/调光', '窗帘'][index]
      },
      stringMixedKey: function (index) {
        let d = {}
        d[Device.CObject.CWT] = 'CWT'
        d[Device.CObject.CT] = 'CT'
        return d[index]
      },
      stringDataType: function (index) {
        let d = {}
        d[Device.CObject.UINT1] = 'UINT1'
        d[Device.CObject.UINT2] = 'UINT2'
        d[Device.CObject.UINT4] = 'UINT4'
        d[Device.CObject.UINT8] = 'UINT8'
        return d[index]
      },
      groupAddrs: function (obj) {
        console.log('filters:groupAddrs', obj.groupAddrs())
        //有5个obj就会调用10次
        return obj.groupAddrs()
      }
    }
  }
</script>

<style>

  * {
    margin: 0;
    padding: 0;
  }
  #grouping { width: 100%; height: 100%; }
  #grouping .toolbar-actions { padding-bottom: 6px; }
  .objs {height: 60%; width: 100%; box-sizing: border-box; background: #fff; border-bottom: 1px solid #ddd;}
  .groups {height: 40%; width: 100%;}
  .table { overflow-y: auto; height: 100%; }
  .table span { display: inline-block; padding: 4px 10px; border-right: 1px solid #ddd; }
  .table-header { font-size: 14px; line-height: 2.2; border-bottom: 1px solid #ddd;}
  .table-body li {border-bottom: 1px solid #ddd;}
  .t0 { width: 80px; }
  .t1 { width: 100px; }
  .t2 { width: 150px; }
  .t3 { width: 300px; }
  .groups {display: flex;}
  .groups aside { width: 40%; height: 100%; box-sizing: border-box; border-right: 1px solid #ddd; padding: 10px; background: #f5f5f4; overflow-y: auto; }
  .groups .groups2objs { flex: 1; box-sizing: border-box; overflow-y: auto; }
  .groups2objs li {}
  aside p i { float: right; }
  aside li {padding: 5px 1em; line-height: 2;}
  aside .r { float: right; }
  aside .active { background: #dcdfe1; }
  aside .indent { display: inline-block; width: 1em; height: 100%; }
</style>