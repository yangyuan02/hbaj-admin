<template>
    <div class="main mgT24">
        <el-table :data="tableData" v-loading="loading">
            <el-table-column prop="name" label="场景名称" align="center" width="200" />
            <el-table-column prop="id" label="场景编码" align="center" width="100" />
            <el-table-column prop="sceneTypeName" label="场景区域" align="center" width="200" />
            <el-table-column prop="detail" label="场景简介" align="center" />
        </el-table>
    </div>
</template>

<script>
import { projectDetail } from "@/model/api";
export default {
    data() {
        return {
            isOpenAddBanner: false,
            loading: false,
            tableData: []
        };
    },
    methods: {
        getList() {
            const projectId = this.$route.params.id;
            this.loading = true;
            projectDetail(
                {
                    type: "GET"
                },
                `${projectId}/panoInfo`
            ).then(res => {
                if (res.suceeded) {
                    this.loading = false;
                    this.tableData = res.data || [];
                }
            });
        }
    },
    created() {
        this.getList();
    }
};
</script>
