<template>
  <div class="baseSearch">
    <el-row class="bs-top">
      <el-col class="left">
        <span>搜索文档：</span>
        <el-select v-model="idxValue" placeholder="请选择" @change="changDocument(idxValue)">
          <el-option
            v-for="(item,index) in idxList"
            :key="index"
            :label="item.id+'('+item.docs+')'+'文档'"
            :value="item.id">
          </el-option>
        </el-select>
        <el-button type="primary"  plain icon="el-icon-search" @click="showDetails = true">自定义查询</el-button>
      </el-col>
    </el-row>
    <el-row class="bs-middle">
      <el-col class="content">
        <span>查询条件：</span>
        <el-form ref="form" v-model="query" v-for="(item,index) in query" :key="index">
          <el-select v-model="query[index].first" class="first">
            <el-option label="must" value="must"></el-option>
            <el-option label="must_not" value="must_not"></el-option>
            <el-option label="should" value="should"></el-option>
          </el-select>
          <!-- 第二个选择框 -->
          <el-select v-model="query[index].second" class="second" @change="changeMain(query[index].second,index)">
            <el-option v-for="(item,index) in idxList[documentIdx].typeDetails" :key="index" :label="idxList[documentIdx].typeDetails[index].name" :value="idxList[documentIdx].typeDetails[index].name"></el-option>
          </el-select>
          <!-- 第三个选择框 -->
          <el-select v-if="mainType['type'+index] == 'string' && mainType['type'+index] !== '_all'" v-model="query[index].third" class="third" @change="changeThird(index)">
            <el-option v-for="(item,index) in usualList.typeOne" :key="index" :label="usualList.typeOne[index]" :value="usualList.typeOne[index]"></el-option>
          </el-select>
          <el-select v-if="mainType['type'+index] !== 'string' && mainType['type'+index] !== 'match_all' && mainType['type'+index] !== '_all'" v-model="query[index].third" class="third" @change="changeThird(index)">
            <el-option v-for="(item,index) in usualList.typeTwo" :key="index" :label="usualList.typeTwo[index]" :value="usualList.typeTwo[index]"></el-option>
          </el-select>
          <el-select v-if="mainType['type'+index] == '_all'" v-model="query[index].third" class="third">
              <el-option v-for="(item,index) in usualList.typeThree" :key="index" :label="usualList.typeThree[index]" :value="usualList.typeThree[index]"></el-option>
          </el-select>
          
          <!-- <el-select v-if="mainType['type'+index] == '_all'" v-model="query[index].thirdTwo" class="third">
            <el-option v-for="(item,index) in usualList.typeThree" :key="index" :label="usualList.typeThree[index]" :value="usualList.typeThree[index]"></el-option>
          </el-select> -->
          <!-- 第三个选择框之后的内容 -->
           <!-- 如果只有一个query的情况 -->
            <el-input v-if="query[index].third == 'term'" v-model="query[index].obj" @input="fouthInput()" placeholder="请输入"></el-input>
            <el-input v-if="query[index].third == 'wildcard'" v-model="query[index].obj" @input="fouthInput()" placeholder="请输入"></el-input>
            <el-input v-if="query[index].third == 'prefix'" v-model="query[index].obj" @input="fouthInput()" placeholder="请输入"></el-input>
            <el-input v-if="query[index].third == 'query_string'" v-model="query[index].obj" @input="fouthInput()" placeholder="请输入"></el-input>
            <el-input v-if="query[index].third == 'text'" v-model="query[index].obj" @input="fouthInput()" placeholder="请输入"></el-input>
            <div class="fourth" v-if="query[index].third == 'query'" >
              <el-input v-model="query[index].obj" @input="fouthInput()"  placeholder="请输入"></el-input>
            </div>
            <div class="fourth" v-if="query[index].third == 'fuzzy'">
              <el-input v-model="query[index].obj['selectDefaultPt']" @input="fouthInput()" placeholder="请输入"></el-input>
              <el-select v-model="query[index].obj['selectA']" @change="fouthInput()">
                  <el-option v-for="(item,index) in fuzzyOption" :key="index" :label="fuzzyOption[index]" :value="fuzzyOption[index]"></el-option>
              </el-select>
              <el-input  v-model="query[index].obj['selectAPt']" @input="fouthInput()" placeholder="请输入"></el-input>
            </div>
            <div class="fourth" v-if="query[index].third == 'range'">
              <el-select  v-model="query[index].obj['selectA']" @change="fouthInput()">
                  <el-option v-for="(item,index) in rangeFrom" :key="index" :label="rangeFrom[index]" :value="rangeFrom[index]"></el-option>
              </el-select>
              <el-input v-model="query[index].obj['selectAPt']" @input="fouthInput()" placeholder="请输入"></el-input>
              <el-select  v-model="query[index].obj['selectB']" @change="fouthInput()">
                  <el-option v-for="(item,index) in rangeTo" :key="index" :label="rangeTo[index]" :value="rangeTo[index]"></el-option>
              </el-select>
              <el-input v-model="query[index].obj['selectBPt']" @input="fouthInput()" placeholder="请输入"></el-input>
            </div>
          <el-button v-show="index === 0" type="primary" icon="el-icon-plus" @click="addQuery"></el-button>
          <el-button v-show="index === 0" type="primary" icon="el-icon-search" @click="findQuery">搜索</el-button>
          <el-button style="marginLeft:0px" v-show="index !== 0" icon="el-icon-close" @click="deleteQuery(index)"></el-button>
        </el-form>
      </el-col>
    </el-row>
    <div class="bs-bottom">
      <el-table
        :data="browseData"
        style="width: 100%"
        height="100%">
        <el-table-column
          prop="_id"
          label="_id"
          width="180">
        </el-table-column>
        <el-table-column
          prop="_index"
          label="_index"
          width="180">
        </el-table-column>
        <el-table-column
          prop="_score"
          label="_score">
        </el-table-column>
        <el-table-column
          prop="_type"
          label="_type">
        </el-table-column>
        <el-table-column v-for="(item,index) in secondTitle" :key="index" width="150">
          <template slot="header" slot-scope="scope">
            <span>{{item}}</span>
          </template>
          <template slot-scope="scope">
            {{scope.row._source[item]}}
          </template>
        </el-table-column>
      </el-table>
    </div>
      <div class="details" :class="showDetails == true ? 'show-details' : ''">
        <img :src="img3" alt="" @click="showDetails = false">
        <CustomQuery></CustomQuery>
      </div>
  </div>
</template>

<script>
import API from '@/api.js'
import CustomQuery from '@/views/dataMonitor/esearch/baseSearch/customQuery.vue'
export default {
  components:{
    CustomQuery
  },
  props:{
    defaultIp:{
      type:String,
      required:true,
    }
  },
  data () {
    return {
      img3:'../../static/image/button-close.png',
      showDetails:false,
      browseData:[], //查询出的数据
      secondTitle:[],  //查询出的二级数据
      idxList:[{ //索引列表
        typeDetails:[
          {
            name:'111',
            type:'string'
          },
          {
            name:'222',
            type:'int'
          },
          {
            name:'333',
            type:'match_all'
          },
          {
            name:'444',
            type:'_all'
          }
        ],  //默认值
      }],
      query:[  //查询数据
        {  
        first:'must',
        second:'match_all',
        third:'',
        thirdTwo:'',
        fourthObj: '',  //存第四列不固定内容 
        obj:{}
      }
      ],
      idxValue: '',  //所选文档
      documentIdx:0, // 所选文档编号
      mainType:{  //主要选择框
        // type0:'match_all'
      },
      usualList:{
        typeOne:['term','wildcard','prefix','fuzzy','range','query_string','text','missing'],  //string类型
        typeTwo:['term','fuzzy','range','query_string','missing'],  //int,long类型
        typeThree:['query'],
        typeMatchAll:['match_all']
      },
      fuzzyOption:['max_expansions','min_similarity'],  //fuzzy的内容
      rangeFrom: ['from','gt','gte'],  //range from选择框内容
      rangeTo: ['to','gt','gte'],  //range to选择框内容
      showDetails:false,
    }
  },
  methods:{
    getIdxList(){  //获取索引列表
      var data = {
        ip:this.defaultIp,
      };
      API.getDBMonitorEsearchIndice(data).then((res)=>{
        this.idxList = res.data;
        this.idxValue = this.idxList[0].id;  //搜索文档默认值
        for(var i = 0;i<this.idxList.length;i++){  //转换格式
          this.idxList[i].typeDetails = [{name:'match_all',type:'match_all'},{name:'_all',type:'_all'}];  //默认类型
          for(var j = 0;j<this.idxList[i].type.length;j++){  //第二个选择框.前面的东西
            for(var item in this.idxList[i].type[j]){     //第二个选择框.前面的东西
              for(var item2 in this.idxList[i].type[j][item]){  //第二个选择框.后面的东西,后缀
                this.idxList[i].typeDetails.push({name:item+'.'+item2,type:this.idxList[i].type[j][item][item2].type})
              }
            }
            
          }
        }
      }).catch((error)=>{
      })
    },
    changDocument(value){  //改变搜索文档
      for(var i = 0;i<this.idxList.length;i++){
        if(this.idxList[i].id === value){
          this.documentIdx = i;
      //     query:[{  //查询数据
      //   first:'must',
      //   second:'match_all',
      //   third:'term',
      //   thirdTwo:'query'
      // }],
        }
      }
    },
    changeThird(index){
      this.$forceUpdate() 
      switch(this.query[index].third){
        case 'fuzzy':
        case 'range':
          this.query[index].obj = {};
          break;
        default:
          this.query[index].obj = "";
          break;
      }
    },
    fouthInput(){
      this.$forceUpdate() 
    },
    changeMain(value,index){  //改变主要的选择框(第二个选择框)
      
      this.$forceUpdate() 
      for(var i = 0;i<this.idxList.length;i++){  //获取type值   this.idxList 三个文档对应的内容
        for(var j =0;j<this.idxList[i].typeDetails.length;j++){   //三个文档分别对应的第二个选择框内的内容
          if(value === this.idxList[i].typeDetails[j].name){
            this.mainType['type'+index] = this.idxList[i].typeDetails[j].type        //这一行的第二个选择框的内容存到this.mainType['type' + 第几行]
          }
        }
      }
      switch(this.mainType['type'+index]){
        case 'string':
          this.query[index].third = this.usualList.typeOne[0];
          break;
        case 'match_all':
          this.query[index].third = this.usualList.typeMatchAll[0];
          break;
        case '_all':
          this.query[index].third = this.usualList.typeThree[0];
          break;
        default:
          this.query[index].third = this.usualList.typeTwo[0];
          break;
      }
      this.changeThird(index)
    },
    addQuery(){  //增加一项
      this.query.push({
        first:'',
        second:'',
      })
    },
    deleteQuery(index){  //删除一项
      this.query.splice(index,1)
    },
    findQuery(){  //搜索
    console.log(this.query,"this.query")
    let bool = {
      must:[],
      must_not:[],
      should:[],
    }
    for(let i = 0;i<this.query.length;i++){
      let obj = {};
      obj[this.query[i].third] = {};
      if(this.query[i].third == 'match_all'){
        bool[this.query[i].first].push(obj);
      }else{
        let obj2 = {};
        switch(this.query[i].third){
          case 'fuzzy':
            obj2.value = this.query[i].obj.selectDefaultPt;
            obj2[this.query[i].obj.selectA] = this.query[i].obj.selectAPt;
            break;
          case 'range':
            obj2[this.query[i].obj.selectA] = this.query[i].obj.selectAPt;
            obj2[this.query[i].obj.selectB] = this.query[i].obj.selectBPt;
            break;
          default:
            obj2 = this.query[i].obj;
            break;
        }
        obj[this.query[i].third][this.query[i].second] = obj2;
        bool[this.query[i].first].push(obj);
      }
    }
    console.log(bool,"bool")

    var data = {
        query:{
          query:{
            bool
          },
          from:0,
          size:"10", 
          sort:[],
          facets:{}
        },
        ip:this.defaultIp,
        indice:this.idxValue,
      }
      data.query = JSON.stringify(data.query)
      API.getDBMonitorEsearchSearch(data).then((res)=>{
        this.browseData = res.data.hits.hits;
        var stArr = [];  //所有二级表头
        for(var i = 0;i<this.browseData.length; i++){
          for(var item in this.browseData[i]._source){
            stArr.push(item)
          }
        }
        this.secondTitle = [...new Set(stArr)]
      })
    }
  },
  created(){
    this.getIdxList();
  }
}
</script>

<style lang="scss" scoped>
.baseSearch{
  height: 100%;
  // border: 1px red solid;
  .bs-top{
    height: 50px;
    // border: 1px blue solid;
    .left{
      line-height: 50px;
      padding-left: 10px;
      span{
        float: left;
      }
      .el-select{
        width: 350px;
      }
      .el-button{
        float: right;
        margin-top: 4px;
        margin-right: 10px;
      }
    }
  }
  .bs-middle{
    height:200px;
    width: 100%;
    overflow: auto;
    .content{
      height: 100%;
      line-height: 50px;
      padding-left: 10px;
      span{
        float: left;
      }
      .el-form{
        margin-left: 90px;
        .first{
          width: 150px;
        }
        .second{
          width: 150px;
        }
        .third{
          width: 150px;
        }
        .el-input {
          width: 150px;
        }
        .fourth {
          display: inline-block;
          .el-select {
            width: 150px;
          }
        }
      }
    }
  }
  .bs-bottom{
    height: calc(100% - 250px);
    // border: 1px blue solid;
  }
}
.details{
    position: fixed;
    top: 90px;
    right: -110%;
    height: calc(100% - 150px);
    width: 80%;
    z-index: 99;
    transition: 0.5s;
    background-color: #fff;
    img{
      position: absolute;
      top: calc(50% - 40px);
      left: -20px;
      cursor: pointer;
    }
  }
.show-details{
  right: 0px;
  transition: 0.5s;
}
</style>
