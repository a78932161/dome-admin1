<template>
  <div class="qaq">
    <el-checkbox :indeterminate="isIndeterminate" v-model="checkAll" @change="handleCheckAllChange">{{all}}
    </el-checkbox>
    <!--<div style="margin: 15px 0;"></div>-->
    <el-checkbox-group v-model="checkedCities" @change="handleCheckedCitiesChange">
      <div style="width: 100%">
        <el-checkbox v-for="(city,index) in cities" :label="city" :key="city.id">
          {{city.name}}
          <el-input placeholder="请输入规则" class="inputList" :key="city.id" v-model="ruleList[index]"
                    @change="listChange"></el-input>
        </el-checkbox>
      </div>
    </el-checkbox-group>
  </div>
</template>
<script>
  export default {
    props: {
      all: {
        type: String
      },
      list: {
        type: Array
      },
      chosen: {
        type: Array
      },
      attach: {
        type: Array
      }
    },
    data() {
      return {
        checkAll: false,
        checkedCities: this.chosen,
        cities: this.list,
        isIndeterminate: true,
        ruleList: this.attach,
      };
    },
    methods: {
      handleCheckAllChange(val) {
        this.checkedCities = val ? this.cities : [];
        this.isIndeterminate = false;
      },
      handleCheckedCitiesChange(value) {
        let checkedCount = value.length;
        this.checkAll = checkedCount === this.cities.length;
        this.isIndeterminate = checkedCount > 0 && checkedCount < this.cities.length;
        this.$emit('dataList', this.checkedCities);
      },
      listChange() {
        console.log(this.ruleList);
        this.$emit('rule1', this.ruleList);
      }
    }
  };
</script>

<style slot>
  .qaq .el-checkbox {
    margin-left: 0;
  }

  .inputList {
    margin: 0 20px 0 20px;
  }

</style>
