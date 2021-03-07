<template>
  <el-table :data="tableData" style="width: 100%" max-height="500">
    <el-table-column prop="name" label="函数名称" width="120"></el-table-column>
    <el-table-column prop="status" label="状态" width="120"> </el-table-column>
    <el-table-column prop="replicas" label="副本数" width="80">
    </el-table-column>
    <el-table-column prop="invocationCount" label="调用次数" width="100">
    </el-table-column>
    <el-table-column prop="image" label="使用镜像" width="200">
    </el-table-column>
    <el-table-column prop="url" label="访问URL" width="200"> </el-table-column>
    <el-table-column label="操作" width="250">
      <template slot-scope="scope">
        <el-row>
          <el-button
            @click.native.prevent="deleteFunction(scope.$index, tableData)"
            size="small"
            type="danger"
          >
            删除
          </el-button>
        </el-row>
        <el-row>
          <el-input-number
            v-model="num"
            @change="handleChange"
            :min="0"
            :max="10"
            label="副本数量"
            size="mini"
          ></el-input-number>
          <el-button
            @click.native.prevent="scaleFunction(scope.$index, tableData)"
            size="small"
          >
            伸缩
          </el-button>
        </el-row>
      </template>
    </el-table-column>
  </el-table>
</template>

<script>
export default {
  data() {
    return {
      tableData: [
        {
          name: "func-1",
          status: "running",
          replicas: 1,
          invocationCount: 100,
          image: "test.io/checkpoint/java",
          url: "localhost:20202",
        },
        {
          name: "func-2",
          status: "running",
          replicas: 3,
          invocationCount: 100,
          image: "test.io/checkpoint/javaddd",
          url: "localhost:20203",
        },
      ],
    };
  },
};
</script>