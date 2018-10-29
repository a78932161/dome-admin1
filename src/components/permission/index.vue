<template>
  <div class="app-container">


    <form-container :model="param" class="_search">
      <field-input class="searchInput" label="搜索关键字" :wrapforlike="true" v-model="param.namelike"></field-input>
      <el-button type="primary" style="margin-left: 15px">查询</el-button>
      <el-button type="primary" @click="dialogShow('add')">增加角色</el-button>
    </form-container>


    <el-table :data="list" class="_list"
              v-loading.body="$apollo.queries.list.loading" element-loading-text="Loading"
              fit highlight-current-row
    >
      <bos-table-column lable="编号" field="number" width="180"></bos-table-column>
      <bos-table-column lable="角色名" field="name"></bos-table-column>
      <bos-table-column lable="创建时间" field="createtime" width="200"
                        :filter="(row)=>formatTime(row.createtime)"
      ></bos-table-column>
      <bos-table-column lable="状态" field="disabled"
                        :filter="(row)=>adminStatus(row.disabled)"
      ></bos-table-column>
      <el-table-column label="操作" align="center" width="200">
        <template slot-scope="scope">
          <el-button type="text" size="mini" @click="dialogShow('edit',scope.row)">修改</el-button>
          <el-button type="text" size="mini" @click="deleteOne(scope.row)" class="_remote-tip">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <bos-paginator :pageInfo="paginator" @bosCurrentPageChanged="changePage"/>
    <field-dialog :title="title" ref="dialog" @confirm="dialogConfirm">
      <form-container ref="role" :model="role">
        <field-input v-model="role.name" label="角色名" width="10"
                     :rules="r(true).all(R.require)" prop="name"></field-input>
        <div v-for="(item,index) in role.privilegeItems">

          <el-checkbox v-model="item.chosen"  >{{item.privilege.name}}</el-checkbox>
          <field-input v-model="item.constraintRule" width="6"></field-input>
        </div>
      </form-container>
    </field-dialog>


  </div>
</template>

<script>


  import {filterAttr} from 'common/js'
  import {rules} from 'common/js/validate';
  import api from 'graph/role.graphql';
  import {historyPageMixin} from 'common/js/mixin';
  import {formRulesMixin,extendRules} from '../../field/common/mixinComponent';



  export default {
    components: {
    },
    mixins: [formRulesMixin,historyPageMixin],
    data() {
      return {
        param: {
          namelike: '%%'
        },
        permissionList:[],
        role:{},



      }
    },

    apollo: {
      list() {//loadingKey
        //created的时候会执行一次，context代表的是vm对象，调试时可以查阅代码：vue-apollo.esm.js:  options = options.call(context)
        return this.getEntityQuery(api.RoleList);
      }
   /* ,  ruleList1() {
        return this.getEntityQuery(api.PrivilegeList);
      }
*/
   },
    computed: {
      isDialogAdd() {
        return this.dialogType === 'add' ? true : false;
      },
      title() {
        return this.dialogType === 'add' ? '添加角色' : '修改角色';
      }
    },


    created(){
      this.gqlQuery(api.PrivilegeList,{paginator:{page:1,size:50}},function(data){
        this.permissionList=data;
      },true);
    },
    methods: {
      query(query, variables) {
        return this.$apollo.query({
          query: query,
          variables: variables,
        });
      },

      gqlQuery(graphql, variables, sCallback,isdefault) {
        this.query(graphql, variables).then((data) => {
          if (data.errors) {   //未通过服务端的表单验证
            this.$message.error(`${data.errors}`);
          }else {
            if(isdefault){
              var deepclonedata = JSON.parse(JSON.stringify(data.data));
              var jqlname = Object.keys(deepclonedata)[0];
              var result = deepclonedata[jqlname];
              data= !result ? null : (result.hasOwnProperty('content') ? result.content : result);
            }
            sCallback.call(this, data);
          }
        }).catch((error) => {
          console.error(error);  //服务器错误或者网络状态问题
          this.$message.error(`${error}`);
        })
      },


      formatTime(time) {
        if (time == null) return;
        return new Date(parseInt(time)).toLocaleString().replace(/:\d{1,2}$/, ' ');
      },

      adminStatus(disabled) {
        return disabled ? '删除' : "正常";
      },
      dialogShow(type, row) {
        this.role=row;

        for(var k=0;k<this.role.privilegeItems.length;k++){
          this.role.privilegeItems[k].chosen=true;
        }

        for(var i=0;i<this.permissionList.length;i++){
          var found=false;
          for(var j=0;j<this.role.privilegeItems.length;j++){
            if(this.permissionList[i].id==this.role.privilegeItems[j].privilege.id){
             // this.role.privilegeItems[j].privilege=this.permissionList[i];
              found=true;
              break;
            }
          }
          if(!found){
            this.role.privilegeItems.push({
              constraintRule:null,
              chosen:false,
              privilege:this.permissionList[i]});
          }
        }

        this.dialogType = type;
        this.$refs.dialog.show();
      },



      dialogConfirm() {
        this.role.privilegeItems=this.role.privilegeItems.filter((item)=>item.chosen);
        JSON.parse(JSON.stringify(this.role))


        for(var k=0;k<this.role.privilegeItems.length;k++){
          var item=this.role.privilegeItems[k];
          delete item.chosen;
        }
        this.removeTypename(this.role);

        this.gqlMutate(api.updateRole,{role:this.role},() => {
          this.callback(`${this.title}成功`);
          this.$refs.dialog.hide();
        })
      },

      removeTypename(obj){
        if(!(obj instanceof Object)){
          return;
        }
        if(obj["__typename"]){
          delete obj["__typename"];
        }
        var arr=Object.keys(obj);
        for(var i=0;i<arr.length;i++){
          this.removeTypename(obj[arr[i]]);
        }
      },


      deleteOne(row) {
        let item = JSON.parse(JSON.stringify(row));
        item.disabled = true;
        this.gqlMutate(api.editAdminist, {
          administ: filterAttr(item)
        }, () => {
          this.callback('删除成功')
        })
      }
    }
  }
</script>

