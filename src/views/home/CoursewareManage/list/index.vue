<template>
    <div class="main mgT24 coursesare-manage-list">
        <div
            style="display: flex;
    justify-content: space-between;"
        >
            <div style="margin-bottom:24px">
                <el-radio-group v-model="blockId" @change="handleShipType">
                    <el-radio-button :label="item.id" type="primary" v-for="item in blockList" :key="item.id">
                        {{ item.name }}
                    </el-radio-button>
                </el-radio-group>
            </div>
            <div>
                <el-button type="primary" @click="addProject">新增</el-button>
            </div>
        </div>

        <div>
            <el-form ref="form" :model="form" label-width="80px" inline label-position="left">
                <el-form-item label="功能">
                    <el-select v-model="form.moduleId" placeholder="请选择" @change="changeFun">
                        <el-option label="全部" value="" :key="-1"></el-option>

                        <el-option :label="item.name" :value="item.id" v-for="item in funcList" :key="item.id"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="课件分类">
                    <el-select v-model="form.classId" placeholder="请选择" @change="changeClass">
                        <el-option :label="item.name" :value="item.id" v-for="item in moduleList" :key="item.id"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="状态">
                    <el-select v-model="form.status" placeholder="请选择" @change="getList">
                        <el-option :label="item.label" :value="item.value" v-for="item in statusList" :key="item.value"></el-option>
                    </el-select>
                </el-form-item>
            </el-form>
        </div>
        <el-table :data="tableData" v-loading="loading">
            <el-table-column label="课件封面" align="center" width="140" :key="Math.random()">
                <template slot-scope="{ row }">
                    <img :src="globalConfig.imagePath + row.imageUrl" alt="" height="100" @click="openPc(row.id)" />
                </template>
            </el-table-column>
            <el-table-column prop="name" label="课件名称" align="center" :key="Math.random()" />
            <el-table-column prop="id" label="课件编码" align="center" width="100" :key="Math.random()" />
            <el-table-column label="课件索引" align="center">
                <template slot-scope="{ row }">
                    <span>{{ `${row.blockName}-${row.moduleName}-${row.className}` }}</span>
                </template>
            </el-table-column>
            <el-table-column prop="detail" label="课件详情" align="center" :key="Math.random()">
                <template slot-scope="{ row }" v-if="row && row.detail">
                    <el-popover placement="top-start" title="" width="250" trigger="hover" :content="row.detail">
                        <div class="ellipsisLineTwo" slot="reference">
                            {{ row.detail }}
                        </div>
                    </el-popover>
                </template>
            </el-table-column>
            <el-table-column label="更新时间" align="center" width="180" :key="Math.random()">
                <template slot-scope="{ row }">
                    <span>{{ row.updateTime | formaData }}</span>
                </template>
            </el-table-column>
            <el-table-column label="状态" align="center" width="100" :key="Math.random()">
                <template slot-scope="{ row }">
                    <span>{{ transformText(row.status) }}</span>
                </template>
            </el-table-column>
            <el-table-column label="是否公开" align="center" width="100" :key="Math.random()">
                <template slot-scope="{ row }">
                    <span>{{ row.publicFlg === 1 ? "公开" : "非公开" }}</span>
                </template>
            </el-table-column>
            <el-table-column label="操作" fixed="right" width="220" align="center" :key="Math.random()">
                <template slot-scope="{ row }">
                    <el-button type="text" v-if="[1].indexOf(row.status) !== -1" @click="handler(row.id, 'PATCH', 'submitVerify')">
                        提交审核
                    </el-button>
                    <el-button type="text" v-if="[5].indexOf(row.status) !== -1" @click="handler(row.id, 'PATCH', 'onshelf')">
                        上架
                    </el-button>
                    <el-button type="text" v-if="[3].indexOf(row.status) !== -1" @click="handler(row.id, 'PATCH', 'offshelf')">
                        下架
                    </el-button>
                    <el-button type="text" v-if="[5].indexOf(row.status) !== -1" @click="handler(row.id, 'PATCH', 'remodify')">
                        退回编辑
                    </el-button>
                    <el-button type="text" v-if="[2].indexOf(row.status) !== -1" @click="verify(row)">
                        审核
                    </el-button>
                    <el-button type="text" v-if="row.publicFlg !== 1 && [3].indexOf(row.status) !== -1" @click="handler(row.id, 'PATCH', 'public')">
                        公开
                    </el-button>
                    <el-button type="text" v-if="row.publicFlg === 1" @click="handler(row.id, 'PATCH', 'unpublic')">
                        取消公开
                    </el-button>
                    <el-button type="text" @click="toDetail(row.id, row)">
                        查看详情
                    </el-button>
                    <el-button type="text" @click="editProject(row)">
                        编辑简介
                    </el-button>
                    <!-- <el-button type="text" v-if="[0].indexOf(row.status) !== -1" @click="dispathTask(row)">
                        下发任务
                    </el-button> -->
                    <el-button type="text" v-if="[0].indexOf(row.status) !== -1" @click="del(row.id)">
                        删除
                    </el-button>
                    <el-button type="text" @click="comment(row)">
                        处理评论
                    </el-button>
                    <el-button type="text" @click="recommend(row)" v-if="row.publicFlg === 1 && [3].indexOf(row.status) !== -1">
                        首页推荐
                    </el-button>
                </template>
            </el-table-column>
        </el-table>

        <app-pagination
            @size-change="setPagination('page_size', $event)"
            @current-change="setPagination('page', $event)"
            :current-page="pagination.page"
            :page-sizes="[10, 20, 50]"
            :page-size="pagination.page_size"
            :total="pagination.total"
        />
        <AddProject :visible.sync="isOpenAddProject" :onSuccess="getList" :projectId="projectId" :editData="editData" />
        <Verify :visible.sync="isOpenVerify" :onSuccess="getList" :id="id" />
    </div>
</template>

<script>
import { project, projectDetail, projectModule, projectClass, appConst, home, task } from "@/model/api";
import AddProject from "./Dialog/AddProject";
import Verify from "./Dialog/Verify";
export default {
    components: {
        AddProject,
        Verify
    },
    data() {
        return {
            projectId: "",
            editData: {},
            isOpenAddProject: false,
            isOpenVerify: false,
            pagination: {
                page: 1,
                page_size: 10,
                total: 0
            },
            tableData: [],
            loading: false,
            form: {
                moduleId: "",
                classId: "",
                status: ""
            },
            id: "",
            blockId: globalConfig.defaultBlocks[0].id,
            blockList: globalConfig.defaultBlocks,
            moduleList: [{ name: "全部", id: "" }],
            funcList: [],
            statusList: [
                {
                    label: "全部",
                    value: ""
                },
                {
                    label: "创建中",
                    value: 0
                },
                {
                    label: "编辑中",
                    value: 1
                },
                {
                    label: "待审核",
                    value: 2
                },
                {
                    label: "上架",
                    value: 3
                },
                {
                    label: "审核不通过",
                    value: 4
                },
                {
                    label: "下架",
                    value: 5
                }
            ]
        };
    },
    methods: {
        transformText(status) {
            const map = {
                0: "创建中",
                1: "编辑中",
                2: "待审核",
                3: "上架",
                4: "审核不通过",
                5: "下架"
            };
            return map[status];
        },
        getList() {
            this.loading = true;
            const { page, page_size } = this.pagination;
            project({
                type: "GET",
                data: {
                    page: page,
                    size: page_size,
                    blockId: this.blockId,
                    moduleId: this.form.moduleId,
                    classId: this.form.classId,
                    status: this.form.status
                }
            }).then(res => {
                if (res.suceeded) {
                    this.loading = false;
                    const { content, total, currentPage, pageSize } = res.data;
                    this.pagination.total = total;
                    this.pagination.page = currentPage;
                    this.pagination.page_size = pageSize;
                    this.tableData = content;
                }
            });
        },
        setPagination(p, v) {
            this.$set(this.pagination, p, v);
            this.getList();
        },
        toDetail(id, data) {
            this.$store.commit("SET_TIPS", `管理"${data.name}"课件详情`);
            this.$router.push(`./courseDetail/${id}`);
        },
        comment(data) {
            this.$store.commit("SET_TIPS", `对"${data.name}"课件的评论处理`);
            this.$router.push({
                path: `/home/homeManage/news/comment/PROJECT/${data.id}`
            });
        },
        verify(data) {
            this.id = "";
            this.id = data.id;
            this.isOpenVerify = true;
        },
        openPc(projectId) {
            // https://msa_pc.vr2shipping.com/my/p_ditor/0/319/2
            const url = globalConfig.APP_DEFAULT_PREVIEW[0].value;
            window.open(`${url}/my/p_ditor/0/${projectId}/2`);
        },
        handler(id, type, url) {
            const params = {};
            if (url === "onshelf") {
                params.publicFlag = false;
                url += "?publicFlag=false";
            }
            projectDetail(
                {
                    type
                },
                `${id}/${url}`
            ).then(res => {
                if (res.suceeded) {
                    this.$message.success("操作成功");
                    this.getList();
                    if (url === "remodify") {
                        this.dispatch(id);
                    }
                } else {
                    this.$message.error("操作失败");
                }
            });
        },
        dispatch(id) {
            projectDetail(
                {
                    type: "GET"
                },
                id
            ).then(res => {
                if (res.suceeded) {
                    const userIds = res.data.userList || [];
                    const detailInfo = res.data;
                    this.dispatchTask(userIds, detailInfo);
                }
            });
        },
        dispatchTask(userIds, detailInfo) {
            const { blockId, createBy, currentFlg, status, name, createTime, updateTime } = detailInfo;
            const projectId = this.$route.params.id;
            const data = {
                blockId, //模块ID
                createBy, //管理员ID
                currentFlg,
                detail: `${name}课件制作`, //项目名称+课件制作
                expireDate: updateTime + 30 * 24 * 60 * 60 * 1000, //当前日期加30天
                name: `${name}课件制作`, //项目名称+课件制作
                projectId: id, //项目ID
                startDate: createTime, //当前日期
                type: "PROJECT_MODIFY", // PROJECT_MODIFY
                userIds: userIds.map(item => item.userId)
            };

            task({
                type: "post",
                data
            }).then(res => {
                if (res.suceeded) {
                    // this.$message.success("操作成功");
                } else {
                    res.message && this.$message.error(res.message);
                }
            });
        },
        del(id) {
            projectDetail(
                {
                    type: "delete"
                },
                id
            ).then(res => {
                if (res.suceeded) {
                    this.$message.success("操作成功");
                    this.getList();
                } else {
                    this.$message.error("操作失败");
                }
            });
        },
        dispathTask(data) {
            console.log(data, "下发任务");
        },
        handleShipType() {
            this.pagination.page = 1;
            this.getFunList();
            this.getList();
            this.form.moduleId = "";
            this.form.classId = "";
        },
        changeFun() {
            this.pagination.page = 1;
            this.getClassList();
            this.form.classId = "";
            this.getList();
        },
        changeClass() {
            this.pagination.page = 1;
            this.getList();
        },
        getFunList() {
            // 获取功能列表
            this.funcList = [];
            projectModule({
                type: "get",
                data: {
                    blockId: this.blockId
                }
            }).then(res => {
                if (res.suceeded) {
                    this.funcList = res.data || [];
                }
            });
        },
        getClassList() {
            this.moduleList = [];
            projectClass({
                type: "get",
                data: {
                    moduleId: this.form.moduleId,
                    blockId: this.blockId
                }
            }).then(res => {
                if (res.suceeded) {
                    this.moduleList = this.moduleList.concat(res.data.content || []);
                }
            });
        },
        addProject() {
            this.projectId = "";
            this.isOpenAddProject = true;
        },
        editProject(data) {
            this.projectId = "";
            this.projectId = data.id;
            this.editData = data;
            this.isOpenAddProject = true;
        },
        recommend(row) {
            appConst({
                type: "post",
                data: {
                    detail: "首页推荐",
                    name: "HOME_RECOMMEND_PROJECT",
                    value: row.id
                }
            }).then(res => {
                if (res.suceeded) {
                    this.refreshAll();
                }
            });
        },
        refreshAll() {
            home(
                {
                    type: "get"
                },
                "refreshAll"
            ).then(res => {});
        }
    },
    created() {
        this.getFunList();
        this.getList();
    }
};
</script>

<style lang="less">
.coursesare-manage-list {
    table {
        .el-table__row {
            height: 125px !important;
            overflow: hidden;
        }
    }
}
</style>
