<template>
  <el-table :data="tableData" style="width: 100%" max-height="500">
    <el-table-column
      prop="name"
      label="函数名称"
      min-width="12%"
    ></el-table-column>
    <el-table-column prop="status" label="状态" min-width="12%">
    </el-table-column>
    <el-table-column prop="replicas" label="副本数" min-width="12%">
    </el-table-column>
    <el-table-column prop="image" label="使用镜像" min-width="18%">
    </el-table-column>
    <el-table-column prop="url" label="访问URL" min-width="30%">
    </el-table-column>
    <el-table-column label="操作" min-width="16%">
      <template slot-scope="scope">
        <el-row :gutter="1">
          <el-col :span="7">
            <el-button
              @click.native.prevent="deleteFunction(scope.$index, tableData)"
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
                <el-button @click="dialogVisible = false">取 消</el-button>
                <el-button type="primary" @click.native.prevent="invokeFunction"
                  >请 求</el-button
                >
              </div>
            </el-dialog>
          </el-col>
          <el-col :span="7">
            <el-button @click="scaleDialogVisible = true" size="mini">
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
                <el-button type="primary" @click="dialogVisible = false"
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
const URLOpenfaas = "http://192.168.150.128:9528/openfaas";
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
      },
    };
  },
  methods: {
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
          response.data.forEach((item, index, array) => {
            newTableData.push({
              name: item.name,
              status: item.replicas > 0 ? "Ready" : "Not ready",
              replicas: item.replicas,
              image: item.image,
              url: URLOpenfaas + "/function/" + item.name,
            });
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