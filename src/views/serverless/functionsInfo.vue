<template>
  <el-table
    :data="tableData"
    style="
      width: 100%;
      box-shadow: 0 2px 4px rgba(0, 0, 0, 0.12), 0 0 6px rgba(0, 0, 0, 0.04);
    "
    max-height="700"
  >
    <el-table-column
      prop="name"
      label="函数名称"
      min-width="12%"
    ></el-table-column>
    <el-table-column prop="status" label="状态" min-width="10%">
    </el-table-column>
    <el-table-column prop="replicas" label="副本数" min-width="8%">
    </el-table-column>
    <el-table-column prop="image" label="使用镜像" min-width="18%">
    </el-table-column>
    <el-table-column prop="url" label="访问URL" min-width="30%">
    </el-table-column>
    <el-table-column label="操作" min-width="22%">
      <template slot-scope="scope">
        <el-row :gutter="1">
          <el-col :span="7">
            <el-button
              @click.native.prevent="deleteFunction(scope.$index)"
              size="mini"
              type="danger"
            >
              删除
            </el-button>
          </el-col>
          <el-col :span="7">
            <el-button
              @click="
                invokeDialogVisible = true;
                currentIndex = scope.$index;
                invokeForm.content = '';
                invokeForm.return = '';
                invokeForm.duration = '未知';
              "
              size="mini"
            >
              请求
            </el-button>
            <el-dialog title="请求函数接口" :visible.sync="invokeDialogVisible">
              <el-form :model="invokeForm">
                <el-form-item label="请求类型：">
                  <el-radio v-model="invokeForm.type" label="text"
                    >纯文本</el-radio
                  >
                  <el-radio v-model="invokeForm.type" label="json"
                    >JSON</el-radio
                  >
                </el-form-item>
                <el-form-item label="请求数据：" style="margin-bottom: 0px">
                </el-form-item>
                <el-input
                  placeholder="请输入请求内容"
                  v-model="invokeForm.content"
                  auto-complete="off"
                  clearable
                  style="margin-top: 0px"
                ></el-input>
                <el-form-item label="响应：" style="margin-bottom: 0px">
                </el-form-item>
                <el-input
                  v-model="invokeForm.return"
                  :readonly="true"
                  style="margin-top: 0px"
                ></el-input>
                <el-form-item label="响应时间：" style="margin-top: 20px">
                  <div>
                    {{ invokeForm.duration }}
                  </div>
                </el-form-item>
              </el-form>
              <div slot="footer" class="dialog-footer">
                <el-button
                  type="primary"
                  @click.native.prevent="invokeFunction"
                  :loading="invokeForm.loading"
                  >请 求</el-button
                >
              </div>
            </el-dialog>
          </el-col>
          <el-col :span="7">
            <el-button
              @click="
                scaleDialogVisible = true;
                currentIndex = scope.$index;
                scaleNum = 0;
              "
              size="mini"
            >
              伸缩
            </el-button>
            <el-dialog title="伸缩函数" :visible.sync="scaleDialogVisible">
              将函数副本数设置为：
              <el-input-number
                v-model="scaleNum"
                :min="0"
                :max="10"
              ></el-input-number>
              <span slot="footer" class="dialog-footer">
                <el-button @click="dialogVisible = false">取 消</el-button>
                <el-button type="primary" @click.native.prevent="scaleFunction"
                  >确 定</el-button
                >
              </span>
            </el-dialog>
          </el-col>
        </el-row>
      </template>
    </el-table-column>
  </el-table>
</template>

<script>
import Axios from "axios";
const URLOpenfaas = "http://133.133.135.25:9528/serverless";
const instance = Axios.create({
  baseURL: URLOpenfaas,
  timeout: 1000,
});
export default {
  data() {
    return {
      currentIndex: 0,
      invokeDialogVisible: false,
      scaleDialogVisible: false,
      scaleNum: 0,
      timer: undefined,
      tableData: [],
      invokeForm: {
        type: "text",
        content: "",
        return: "",
        duration: "未知",
        loading: false,
      },
    };
  },
  methods: {
    deleteFunction: function (index) {
      let funcName = this.tableData[index].name;
      this.$confirm("是否确定删除该函数？", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then(() => {
        instance
          .delete("/system/functions?namespace=openfaas-fn", {
            data: {
              functionName: funcName,
            },
            headers: { "Content-Type": "application/json" },
            responseType: "json",
          })
          .then(() => {
            this.$message("删除成功");
            this.tableData[index].status = "Deleting";
          })
          .catch((error1) => {
            this.$message(
              "发生错误：" + error1.statusText + "\n" + error1.data
            );
          });
      });
    },
    scaleFunction: function () {
      let funcName = this.tableData[this.currentIndex].name;
      instance
        .post(
          "/system/scale-function/" + funcName,
          {
            serviceName: funcName + ".openfaas-fn",
            replicas: this.scaleNum,
          },
          {
            headers: { "Content-Type": "application/json" },
          }
        )
        .then(() => {
          this.$message("设置成功");
          this.scaleDialogVisible = false;
        })
        .catch((error1) => {
          this.$message("发生错误：" + error1.statusText + "\n" + error1.data);
        });
    },
    invokeFunction: function () {
      let requestContentType =
        this.invokeForm.type === "json" ? "application/json" : "text/plain";
      let funcInfo = this.tableData[this.currentIndex];
      let invocationStart = new Date().getTime();
      this.invokeForm.loading = true;
      instance
        .post(
          "/function/" + funcInfo.name + ".openfaas-fn",
          this.invokeForm.content,
          {
            headers: { "Content-Type": requestContentType },
            responseType: this.invokeForm.type,
          }
        )
        .then((response) => {
          let data = response.data;
          let status = response.status;
          if (typeof data === "object") {
            this.invokeForm.return = JSON.stringify(data, null, 2);
          } else {
            this.invokeForm.return = data;
          }
          let invocationEnd = new Date().getTime();
          this.invokeForm.duration =
            (invocationEnd - invocationStart) / 1000 + " s";
          this.$message("请求成功");
        })
        .catch((error1) => {
          this.invokeForm.return = error1.statusText + "\n" + error1.data;
          let invocationEnd = new Date().getTime();
          this.invokeForm.duration =
            (invocationEnd - invocationStart) / 1000 + " s";
          this.$message("错误");
        });
      this.invokeForm.loading = false;
    },
    updateFunctionInfo: function () {
      const instance = Axios.create({
        baseURL: URLOpenfaas,
        timeout: 1000,
      });
      instance
        .get("/system/functions", {
          params: {
            namespace: "openfaas-fn",
          },
        })
        .then((response) => {
          console.log(this.currentIndex);
          let newTableData = [];
          let currentTableMap = new Map();
          this.tableData.forEach((item, index, array) => {
            currentTableMap.set(item.name, item);
          });
          response.data.forEach((item, index, array) => {
            let newFuncInfo = {
              name: item.name,
              replicas: item.replicas,
              image: item.image,
              status: item.replicas > 0 ? "Ready" : "Not ready",
              url: URLOpenfaas + "/function/" + item.name,
            };
            if (currentTableMap.get(item.name)) {
              let info = currentTableMap.get(item.name);
              if (info.status !== "Ready" && info.status !== "Not ready") {
                newFuncInfo.status = info.status;
              }
            }
            newTableData.push(newFuncInfo);
          });
          this.tableData = newTableData.sort((a, b) => {
            if (a.name < b.name) return -1;
            if (a.name > b.name) return 1;
            return 0;
          });
        });
    },
  },
  mounted() {
    this.updateFunctionInfo();
    if (this.timer) {
      clearInterval(this.timer);
    } else {
      this.timer = setInterval(() => {
        this.updateFunctionInfo();
      }, 2000);
    }
  },
  destroyed() {
    clearInterval(this.timer);
  },
};
</script>