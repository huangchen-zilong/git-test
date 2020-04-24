
<template>
  <div class="execution">
    <basic-container>
      <div class="formSearch">
        <form class="el-form el-form--inline" id="formSearch">
          <div class="el-form-item">
            <el-input v-model="params.dangerChemicalName" placeholder="危化品名称"></el-input>
          </div>
          <div class="el-form-item">
            <el-select v-model="params.deptId" placeholder="管理部门" filterable :clearable="true">
              <el-option
                v-for="item in deptIdList"
                :key="item.deptId"
                :label="item.name"
                :value="item.deptId"
              ></el-option>
            </el-select>
          </div>
          <div class="el-form-item" v-if="searchMore">
            <el-input v-model="params.address" placeholder="所在装置或区域"></el-input>
          </div>
          <div class="el-form-item" v-if="searchMore">
            <el-select v-model="params.storageMode" filterable :clearable="true" placeholder="存储形式">
              <el-option
                v-for="item in storageModeOption"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              ></el-option>
            </el-select>
          </div>
          <!--<div class="el-form-item" v-if="searchMore">
            <el-select v-model="params.staffId" placeholder="管理人员">
              &lt;!&ndash;<el-option>
              </el-option>&ndash;&gt;
            </el-select>
          </div>-->
          <div class="el-form-item">
            <el-button type="primary" @click="onSubmit" icon="el-icon-search">搜索</el-button>
          </div>
          <div class="el-form-item">
            <a
              href="javascript:void(0);"
              @click="searchMore=!searchMore"
              class="searchMore"
            >{{searchMore==true?"隐藏更多查询条件":"更多查询条件"}}</a>
          </div>
        </form>
        <div class="formBtns">
          <el-button
            type="primary"
            @click="handleAdd"
            size="small"
            icon="el-icon-plus"
            plain
            v-if="permissions.risk_dangermatter_add"
          >新 增</el-button>
          <!-- <el-upload
            class="upload-demo"
            ref="upload"
            action="/admin/file/upload"
            :headers="headers"
            :data="uploadParameter"
            :show-file-list=false
            v-if="permissions.risk_dangermatter_import"
            accept=".xls,.xlsx"
            :on-success="uploadSuccess"
            style="display: inline-block;margin-left: 10px;">
            <el-button type="primary"
                       size="small"
                       icon="el-icon-download" plain>导入</el-button>
          </el-upload>-->
          <ImportFile
            v-if="permissions.risk_dangermatter_import"
            :bucketParameter="uploadParameter"
            requestUrl="/risk/dangermatter/importDangerMatter"
            @uploadSuccess="uploadSuccess"
          ></ImportFile>
          <!-- <el-button
            type="primary"
            @click="exportList"
            size="small"
            icon="el-icon-upload2"
            plain
            v-if="permissions.risk_dangermatter_export"
            v-show="!imgMenu"
            style="margin-left: 10px;"
          >导出</el-button>-->
          <ExportFile
            v-if="permissions.risk_dangermatter_export"
            v-show="!imgMenu"
            :selectedList="selectedList"
            :params="params"
            requestUrl="/risk/dangermatter/exportDangerMatter"
          ></ExportFile>
          <!-- <el-button
            type="primary"
            @click="deleteList"
            size="small"
            v-show="!imgMenu"
            icon="el-icon-delete"
            plain
            v-if="permissions.risk_dangermatter_del"
          >批量删除</el-button>-->
          <DeleteList
            v-show="!imgMenu"
            v-if="permissions.risk_dangermatter_del"
            :selectedList="selectedList"
            requestUrl="/risk/dangermatter/deleteDangerMatter"
            @deleteSuccess="refreshChange"
          ></DeleteList>
        </div>

        <div class="menuTab">
          <el-button
            :type="imgMenu==true?'primary':''"
            icon="el-icon-picture"
            circle
            size="small"
            @click="imgMenu=true"
          ></el-button>
          <el-button
            :type="!imgMenu==true?'primary':''"
            icon="el-icon-tickets"
            circle
            size="small"
            @click="imgMenu=false"
          ></el-button>
        </div>
      </div>
      <div class="imgTemplate" v-show="imgMenu">
        <ul>
          <li class="imgTemplateList" v-for="item in imgTableData" :key="item.id">
            <div class="listInCon">
              <div class="listImg" @click="dialogImageUrl=item.picture;dialogImgVisible=true;">
                <img :src="item.picture" alt />
              </div>
              <div
                class="listMessage"
                @click="$router.push({path:`/risk/dangermatterDetail/`,query:{id:item.id,}});"
              >
                <el-tooltip :content="item.dangerChemicalName" placement="top">
                  <h4>{{item.dangerChemicalName}}</h4>
                </el-tooltip>
                <p class="titleLabel">
                  <span v-if="item.whetherDangerChemicals===1">危险化学品</span>
                  <span v-if="item.whetherHighlyToxic===1">剧毒</span>
                  <span v-if="item.whetherMajorSupervisionChemicals===1">重点监管</span>
                  <span v-if="item.whetherPrecursorToxicChemicals===1">易制毒</span>
                  <span v-if="item.whetherPrecursorExplosiveChemicals===1">易制爆</span>
                </p>
                <div class="listItem">
                  <span class="itemLabel">所在装置/区域：</span>
                  <el-tooltip :content="item.address" placement="top-start">
                    <p class="itemContent">{{item.address}}</p>
                  </el-tooltip>
                </div>
                <div class="listItem">
                  <div class="floatLeft">
                    <span class="itemLabel">存在量：</span>
                    <el-tooltip :content="item.quantityOfMatter+item.unit" placement="top-start">
                      <p class="itemContent">{{item.quantityOfMatter+item.unit}}</p>
                    </el-tooltip>
                  </div>
                  <div class="floatLeft" v-if="item.thresholdQuantity!=null">
                    <span class="itemLabel">临界量：</span>
                    <el-tooltip :content="item.thresholdQuantity+item.unit" placement="top-start">
                      <p
                        class="itemContent"
                        style="color:#FF0000;"
                      >{{item.thresholdQuantity+item.unit}}</p>
                    </el-tooltip>
                  </div>
                  <div style="clear: both"></div>
                </div>
                <div class="listItem">
                  <span class="itemLabel">管理部门：</span>
                  <el-tooltip :content="item.deptName" placement="top-start">
                    <p class="itemContent">{{item.deptName}}</p>
                  </el-tooltip>
                </div>
              </div>
            </div>
          </li>
          <div style="clear: both"></div>
        </ul>
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="imgPage.currentPage"
          :page-sizes="[4, 8]"
          :page-size="4"
          layout="total,sizes, prev, pager, next, jumper"
          :total="imgPage.total"
          style="text-align: right"
        ></el-pagination>
      </div>
      <avue-crud
        ref="crud"
        :page="page"
        :data="tableData"
        :table-loading="tableLoading"
        :option="tableOption"
        v-show="!imgMenu"
        @selection-change="selectionChange"
        @refresh-change="refreshChange"
        @current-change="currentChange"
        @size-change="sizeChange"
        @row-click="rowClick"
      >
        <template slot-scope="scope" slot="menu">
          <el-button
            type="primary"
            v-if="permissions.risk_dangermatter_edit"
            size="small"
            @click="handleEdit(scope.row,scope.index)"
          >编辑</el-button>
          <el-button
            type="danger"
            v-if="permissions.risk_dangermatter_del"
            size="small"
            @click="handleDel(scope.row,scope.index)"
          >删除</el-button>
        </template>
      </avue-crud>
      <!-- <el-dialog title="信息" :visible.sync="dialogVisible" width="30%">
        <span>{{dialogMessage}}</span>
        <span slot="footer" class="dialog-footer">
          <el-button @click="uploadTemplate">下载错误提示文件</el-button>
          <el-button type="primary" @click="dialogVisible = false">关闭</el-button>
        </span>
      </el-dialog> -->
      <el-dialog :visible.sync="dialogImgVisible">
        <img width="100%" :src="dialogImageUrl" alt />
      </el-dialog>
    </basic-container>
  </div>
</template>

<script>
import {
  fetchList,
  getObj,
  addObj,
  putObj,
  delObj,
  exportDangerMatter,
  importDangerMatter
} from "@/api/risk/dangermatter";
import { listMixin } from "@/mixins/list";
import { tableOption } from "@/const/crud/risk/dangermatter";
import { listDeptByUser } from "@/api/admin/dept";
import store from "@/store";
import { tableImg, handleFile, download } from "@/util/util";
import { mapGetters } from "vuex";
import ImportFile from "@/components/upload/import-file";
import ExportFile from "@/components/export/export-file";
import DeleteList from "@/components/delete/delete-list";
export default {
  mixins: [listMixin],
  name: "dangermatter",
  components: {
    ImportFile,
    ExportFile,
    DeleteList
  },
  data() {
    return {
      /* headers: {
        Authorization: "Bearer " + store.getters.access_token
      }, */
      uploadParameter: {
        bucketType: 23
      },
      listUrl:"/risk/dangermatter/listPagingDangerMatter",
      storageModeOption: [
        { value: 1, label: "存储" },
        { value: 2, label: "仓库" },
        { value: 3, label: "堆场" },
        { value: 4, label: "钢瓶" },
        { value: 5, label: "桶" },
        { value: 6, label: "袋装" },
        { value: 7, label: "管道" },
        { value: 8, label: "瓶装" },
        { value: 9, label: "其他" }
      ],
      imgMenu: true, //是否是图片模式
      deptIdList: [],
      //tableData: [],
      /* page: {
        total: 0, // 总页数
        currentPage: 1, // 当前页数
        pageSize: 10 // 每页显示多少条
      }, */
      //tableLoading: false,
      tableOption: tableOption,
      //searchMore: false,
      params: {
        whetherMajor: 0,
        dangerChemicalName: "",
        deptId: "",
        address: "",
        storageMode: "",
        staffId: ""
      },
      //selectedList: [],
      //dialogVisible: false,
      //dialogImgVisible: false,
      /* dialogMessage: "",
      templateAddress: "",
      dialogImageUrl: "", */
      //imgTableData: [], //图片模式数据
      /* imgPage: {
        //图片模式页数
        total: 0, // 总页数
        currentPage: 1, // 当前页数
        pageSize: 4 // 每页显示多少条
      } */
    };
  },
  created() {},
  mounted: function() {
    listDeptByUser().then(response => {
      //console.log(response);
      if (response.status == 200 && response.data.code == 0) {
        this.deptIdList = response.data.data;
      } else {
        this.$message.error(response.msg);
      }
    });
    this.getList(this.page, this.params);
    this.getImgList(this.imgPage, this.params);
  },
  computed: {
    ...mapGetters(["permissions"])
  },
  watch: {
    $route() {
      this.refreshChange();
    }
  },
  methods: {
    /*获取表格列表数据*/
    /* getList(page, params) {
      this.tableLoading = true;
      fetchList(
        Object.assign(
          {
            page: {
              current: page.currentPage,
              size: page.pageSize
            }
          },
          params
        )
      ).then(response => {
        //console.log(response)
        if (response.status == 200 && response.data.code == 0) {
          var list = response.data.data.list;
          for (var i in list) {
            if (list[i].picture != "" && list[i].picture != null) {
              var pictureFirst = list[i].picture.split(",")[0];
              tableImg(pictureFirst, i).then(obj => {
                list[obj.num].picture = obj.src;
              });
            }
          }
          this.tableData = list;
          this.page.total = response.data.data.page.total;
          this.tableLoading = false;
        } else {
          this.$message.error(response.msg);
        }
      });
    }, */

    /*获取图片模式数据*/
    /* getImgList(page, params) {
      this.tableLoading = true;
      fetchList(
        Object.assign(
          {
            page: {
              current: page.currentPage,
              size: page.pageSize
            }
          },
          params
        )
      ).then(response => {
        if (response.status == 200 && response.data.code == 0) {
          var list = response.data.data.list;
          for (var i in list) {
            var pictureFirst = list[i].picture.split(",")[0];
            tableImg(pictureFirst, i).then(obj => {
              list[obj.num].picture = obj.src;
            });
          }
          this.imgTableData = list;
          this.imgPage.total = response.data.data.page.total;
          this.tableLoading = false;
          //console.log(this.imgTableData);
        } else {
          this.$message.error(response.msg);
        }
      });
    }, */

    onSubmit: function() {
      this.imgPage.currentPage = 1;
      this.page.currentPage = 1;
      this.refreshChange();
    },
    /**
     * @title 打开新增窗口
     * @detail 调用crud的handleadd方法即可
     *
     **/
    handleAdd: function() {
      /*this.$refs.crud.rowAdd()*/
      this.$router.push({ path: `/risk/dangermatter/dangermatterAdd` });
    },
    handleEdit(row, index) {
      //console.log(row);
      this.$router.push({
        path: `/risk/dangermatter/dangermatterAdd`,
        query: {
          name: "危化品编辑",
          id: row.id
        }
      });
      //this.$refs.crud.rowEdit(row, index)
    },
    handleDel(row, index) {
      var _this = this;
      //console.log(row);
      this.$confirm(
        "是否确认删除危化品名称为" + row.dangerChemicalName,
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      )
        .then(function() {
          return delObj({ idList: [row.id] });
        })
        .then(data => {
          if (data.data.msg == "success") {
            //_this.tableData.splice(index, 1)
            _this.$message({
              showClose: true,
              message: "删除成功",
              type: "success"
            });
            this.refreshChange();
          } else {
            this.$message.error("删除失败！");
          }
        });
    },
    /*批量删除*/
    /* deleteList: function() {
      var _this = this;
      if (this.selectedList.length == 0) {
        this.$message.error("请先勾选表格数据！");
      } else {
        this.$confirm("是否确认删除勾选的数据", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(function() {
            //console.log({idList:_this.selectedList});
            return delObj({ idList: _this.selectedList });
          })
          .then(data => {
            if (data.data.msg == "success") {
              _this.$message({
                showClose: true,
                message: "删除成功",
                type: "success"
              });
              this.refreshChange();
            } else {
              this.$message.error("删除失败！");
            }
          });
      }
    }, */

    /*行点击查看详情*/
    rowClick: function(row, event, column) {
      if (column.property == "dangerChemicalName") {
        this.$router.push({
          path: `/risk/dangermatterDetail/`,
          query: {
            id: row.id
          }
        });
      } else if (column.property == "picture") {
        this.dialogImageUrl = row.picture;
        this.dialogImgVisible = true;
      }
    },
    /**
     * 刷新回调
     */
    /* refreshChange() {
      this.getList(this.page, this.params);
      this.getImgList(this.imgPage, this.params);
    }, */
    /*获取勾选的数据*/
    /* selectionChange: function(list) {
      let selectArr = [];
      for (var i in list) {
        selectArr.push(list[i].id);
      }
      this.selectedList = selectArr;
    }, */
    /*导出*/
    /* exportList: function() {
      this.$confirm("您确定要下载吗", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.params.idList = this.selectedList;
          return exportDangerMatter(this.params);
        })
        .then(response => {
          if (response.data.code == 0) {
            var fileName = response.data.data;
            handleFile(fileName).then(src => {
              download(
                fileName.split("&")[fileName.split("&").length - 1],
                src
              );
            });
          } else {
            this.$message.error("导出失败！");
          }
        });
    }, */

    /*导入*/
    /* uploadSuccess: function() {
      //console.log(response);
      this.refreshChange();
      if(response.code==0){
          var fileName=response.data.bucketName+"-"+response.data.fileName;
           importDangerMatter({fileUrl:fileName}).then(result =>{
              //console.log(result);
            if(result.data.code==0){
              this.$message.success("导入成功！");
              this.refreshChange();
            }else{
              if(result.data.data==null){
                this.$message.error(result.data.msg);
              }else{
                this.dialogMessage=result.data.msg;
                this.templateAddress=result.data.data;
                this.dialogVisible=true;
              }
            }
          })
        }else{
          this.$message.error(response.msg);
        }
    }, */

    /*下载导入错误文件*/
    /* uploadTemplate: function() {
      handleFile(this.templateAddress).then(src => {
        download(
          this.templateAddress.split("&")[
            this.templateAddress.split("&").length - 1
          ],
          src
        );
      });
    }, */

    /*表格页码切换*/
    /* currentChange: function(index) {
      this.page.currentPage = index;
      this.getList(this.page, this.params);
    }, */

    /*表格切换每页条数*/
    /* sizeChange: function(number) {
      this.page.pageSize = number;
      this.getList(this.page, this.params);
    }, */

    /*图片模式切换每页条数*/
    handleSizeChange: function(number) {
      this.imgPage.pageSize = number;
      this.getImgList(this.imgPage, this.params);
    },
    /*图片模式跳页面*/
    handleCurrentChange: function(index) {
      this.imgPage.currentPage = index;
      this.getImgList(this.imgPage, this.params);
    }
  }
};
</script>

<style lang="scss" scoped>
</style>
