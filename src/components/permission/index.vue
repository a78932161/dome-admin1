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
      <form-container ref="inlineForm" :model="inlineForm">
        <field-input v-model="inlineForm.name" label="角色名" width="10"
                     :rules="r(true).all(R.require)" prop="name"></field-input>
        <!--<el-form-item label="权限" label-width="120px">-->
        <!--<checkbox1 all="全选" :list="list1" :chosen="chosen" @dataList="dataList"></checkbox1>-->
        <!--</el-form-item>-->
        <!--<el-form-item label=约束规则 label-width="120px">-->
        <!--<el-input placeholder="请输入规则" v-for="(item, index) in ruleList" :key="index" v-model="ruleData[index]">-->
        <!--<template slot="prepend">{{item.name}}</template>-->
        <!--</el-input>-->
        <!--</el-form-item>-->
        <el-form-item label="测试" label-width="120px">
          <checkbox2 all="全选" :list="list1" :chosen="chosen" :attach="attach" @dataList="dataList"
                     @rule1="rule1"></checkbox2>
        </el-form-item>

      </form-container>
    </field-dialog>


  </div>
</template>

<script>


  import {filterAttr} from 'common/js'
  import {rules} from 'common/js/validate';
  import api from 'graph/role.graphql';
  import {historyPageMixin} from 'common/js/mixin';
  import checkbox1 from '@/base/checkbox'
  import checkbox2 from '@/base/checkbox2'

  export default {
    components: {
      checkbox1,
      checkbox2
    },
    data() {
      return {
        rules,
        dialogType: '',
        inlineForm: {},
        list1: [],
        chosen: [],
        ruleList: [],
        ruleData: [],
        ruleData1: [],
        attach: [],
      }
    },
    mixins: [historyPageMixin],
    apollo: {
      list() {//loadingKey
        //created的时候会执行一次，context代表的是vm对象，调试时可以查阅代码：vue-apollo.esm.js:  options = options.call(context)
        return this.getEntityQuery(api.RoleList);
      },
      ruleList1() {
        return this.getEntityQuery(api.PrivilegeList);
      }
    },
    computed: {
      isDialogAdd() {
        return this.dialogType === 'add' ? true : false;
      },
      title() {
        return this.dialogType === 'add' ? '添加角色' : '修改角色';
      }
    },


    methods: {
      rule1(data) {
        this.ruleData = data;
      },
      dataList(data) {
        console.log(data);
        this.ruleList = data;
      },
      formatTime(time) {
        if (time == null) return;
        return new Date(parseInt(time)).toLocaleString().replace(/:\d{1,2}$/, ' ');
      },
      adminStatus(disabled) {
        return disabled ? '删除' : "正常";
      },
      dialogShow(type, row) {
        this.chosen = [];
        this.ruleData = [];
        this.ruleList = [];
        this.list1 = [];
        this.ruleList1.forEach((value, index) => {
          this.ruleList1[index] = filterAttr(value);
        });
        this.list1 = this.ruleList1;
        this.dialogType = type;
        this.$refs.dialog.show();
        this.$nextTick(() => {    //dialog框出现以后，进行清空验证
          this.$refs.inlineForm.clearValidate();
        });
        if (type === 'add') {
          this.inlineForm = {};
          return;
        } else {
          this.inlineForm = JSON.parse(JSON.stringify(row));
          if (this.inlineForm.privilegeItems) {
            // this.inlineForm.privilegeItems.forEach((value, index) => {
            //   if (value.constraintRule != null) {
            //     this.ruleData.push(value.constraintRule);
            //   } else {
            //     this.ruleData.push('');
            //   }
            //   this.list1.forEach((value1, index) => {
            //     if (value.privilege.name == value1.name) {
            //
            //     } else {
            //       console.log(1);
            //     }
            //   })
            // });

            let promise = new Promise((resolve, reject) => {
              this.list1.forEach((value, index) => {
                this.inlineForm.privilegeItems.forEach((value1, index1) => {
                  if (value.name == value1.privilege.name) {
                    this.$nextTick(() => {
                      this.chosen.push(value);
                      let a = {
                        name: value.name,
                        roles: value1.constraintRule
                      };
                      this.ruleData.push(a);
                      resolve();
                    });
                  }
                })
              });
            });
            promise.then(() => {
              this.list1.forEach((value, index) => {
                this.ruleData.forEach((value1, index1) => {
                  if (value.name == value1.name) {
                    this.ruleData1.push(value1);
                  }
                })
                // if (value.name == this.ruleData[index].name) {
                //   this.ruleData1.push(this.ruleData[index].roles)
                // } else {
                //   this.ruleData1.push('')
                // }
              })
            })
          }
          console.log(this.ruleData1);
          this.ruleList = this.chosen;
          // this.attach = this.ruleData;
        }
      },
      dialogConfirm() {
        this.inlineForm['privilegeItems'] = [];
        this.ruleList.forEach((value, index) => {
          let a = {
            constraintRule: this.ruleData[index],
            privilege: {
              id: value.id,
            }
          };
          this.inlineForm['privilegeItems'].push(a);
        });
        this.$refs.inlineForm.gqlValidate(this.dialogType === 'add' ? api.addRole : api.updateRole, {
          role: this.inlineForm
        }, () => {
          this.callback(`${this.title}成功`);
          this.$refs.dialog.hide();
        });
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

