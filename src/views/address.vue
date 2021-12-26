<template>
    <div> 
        <div class="container">
            <!-- 表单 -->
            <el-table ref="filterTable" row-key="date" :data="tableData.slice((query.pageIndex-1)*query.pageSize,query.pageIndex*query.pageSize)" border style="width: 100%">
                <el-table-column label="序号" type="index"  align="center" style="width:30%"/>
                <el-table-column prop="hotcity" label="热门城市"  align="center" style="width:10%" 
                :filters="[
                    { text: '热', value: '热' },
                    { text: '冷', value: '冷' },
                ]"
                :filter-method="filterTag"
                filter-placement="bottom-end">
                <template #default="scope">
                    <el-tag :type="scope.row.hotcity === '热' ? 'danger' : 'info'" disable-transitions>{{ scope.row.hotcity }}</el-tag>
                </template>
                </el-table-column>

                <el-table-column prop="province" label="省级"  align="center" style="width:10%" />
                <el-table-column prop="city" label="市级"  align="center" style="width:10%" />
                <el-table-column prop="county" label="区/县"  align="center" style="width:10%" />
                <!-- <el-table-column prop="hospital" label="三甲医院"  align="center" style="width:10%" /> -->
                <el-table-column prop="states" label="优先级"  align="center" sortable style="width:10%"/>

                <el-table-column label="操作"  align="center" style="width:10%" >
                    <template #default="scope">
                        <el-button size="mini" @click="handleEdit(scope.$index, scope.row)" >修改</el-button>
                        <el-button size="mini" type="danger" @click="handleDelete(scope.$index, scope.row)">隐藏</el-button>
                    </template>
                </el-table-column>
            </el-table>
            <div class="pagination">
                    <el-pagination background layout="total, sizes, prev, pager, next" :current-page="query.pageIndex"
                       :page-sizes="[10, 15, 20]" v-model:page-size="query.pageSize" :total="tableData.length" @current-change="handlePageChange"></el-pagination>
            </div>
        </div>

        <!-- 编辑弹出框 -->
        <el-dialog title="编辑" v-model="editVisible" width="30%">
            <el-form label-width="70px">
                <el-form-item label="省">
                    <el-input v-model="form.province"></el-input>
                </el-form-item>
                <el-form-item label="市">
                    <el-input v-model="form.city"></el-input>
                </el-form-item>
                <el-form-item label="区/县">
                    <el-input v-model="form.county"></el-input>
                </el-form-item>
            </el-form>
            <template #footer>
                <span class="dialog-footer">
                    <el-button @click="editVisible = false">取 消</el-button>
                    <el-button type="primary" @click="saveEdit">确 定</el-button>
                </span>
            </template>
        </el-dialog>
    </div>
</template>

<script>
import { ref, reactive } from "vue";
import { ElMessage, ElMessageBox } from "element-plus";
import { fetchData } from "../api/index";

export default {
    name: "address",
    setup() {
        const query = reactive({
            address: "",
            name: "",
            pageIndex: 1,
            pageSize: 10,
        });
        const tableData = ref([]);
        // 获取表格数据
        const getData = () => {
            fetchData(query).then((res) => {
                tableData.value = res.list;
            });
        };
        getData();
        

        // 查询操作
        const handleSearch = () => {
            query.pageIndex = 1;
            getData();
        };
        // 分页导航
        const handlePageChange = (val) => {
            query.pageIndex = val;
            getData();
        };

        // 删除操作
        const handleDelete = (index) => {
            // 二次确认删除
            ElMessageBox.confirm("确定要隐藏吗？", "提示", {
                type: "warning",
            })
                .then(() => {
                    ElMessage.success("隐藏成功");
                    tableData.value.splice(index, 1);
                })
                .catch(() => {});
        };

        // 表格编辑时弹窗和保存
        const editVisible = ref(false);
        let form = reactive({
            province: "",
            city: "",
            county: "",
        });
        let idx = -1;
        const handleEdit = (index, row) => {
            idx = index;
            Object.keys(form).forEach((item) => {
                form[item] = row[item];
            });
            editVisible.value = true;
        };
        const saveEdit = () => {
            editVisible.value = false;
            ElMessage.success(`修改第 ${idx + 1} 行成功`);
            Object.keys(form).forEach((item) => {
                tableData.value[idx][item] = form[item];
            });
        };

        const filterTag = (value, row) =>{
            return row.hotcity === value
        };

        return {
            query,
            tableData,
            editVisible,
            form,
            handleSearch,
            handlePageChange,
            handleDelete,
            handleEdit,
            saveEdit,
            filterTag,
            
        };
    },
};
</script>

<style scoped>
.handle-box {
    margin-bottom: 20px;
}

.handle-select {
    width: 120px;
}

.handle-input {
    width: 300px;
    display: inline-block;
}
.table {
    width: 100%;
    font-size: 14px;
}
.red {
    color: #ff0000;
}
.mr10 {
    margin-right: 10px;
}
.table-td-thumb {
    display: block;
    margin: auto;
    width: 40px;
    height: 40px;
}
</style>
