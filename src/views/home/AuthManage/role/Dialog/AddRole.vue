<template>
    <el-dialog title="新增角色名称" :visible="visible" width="30%" @open="open" @close="close">
        <el-form ref="form" :model="form" label-width="80px" label-position="left">
            <el-form-item label="船只" prop="blockId">
                <el-select v-model="form.blockId" placeholder="请选择">
                    <el-option :label="item.name" :value="item.id" v-for="item in shippList" :key="item.id"></el-option>
                </el-select>
            </el-form-item>
            <el-form-item label="角色名称" prop="name">
                <el-input v-model="form.name"></el-input>
            </el-form-item>
        </el-form>

        <span slot="footer" class="dialog-footer">
            <el-button @click="close">取 消</el-button>
            <el-button type="primary" @click="save">确 定</el-button>
        </span>
    </el-dialog>
</template>

<script>
import { roleDetail, role } from "@/model/api";

export default {
    props: {
        visible: {
            type: Boolean,
            default: false
        },
        onSuccess: {
            type: Function
        }
    },
    data() {
        return {
            form: {
                name: "",
                blockId: ""
            },
            shippList: globalConfig.defaultBlocks
        };
    },
    methods: {
        close() {
            this.$refs.form.resetFields();
            this.$emit("update:visible", false);
        },
        open() {
            console.log("打开");
        },
        save() {
            this.form.enterpriseId = globalConfig.defaultInfo.APP_DEFAULT_ENTERPRISE[0];
            role({
                type: "post",
                data: this.form
            }).then(res => {
                this.$message.success("操作成功");
                this.onSuccess && this.onSuccess();
                this.close();
            });
        }
    }
};
</script>
