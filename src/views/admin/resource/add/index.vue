<template>
  <section>
    <el-drawer
      :title="title"
      :size="size"
      :append-to-body="true"
      :visible.sync="drawerVisible"
      :direction="direction"
      :wrapperClosable="wrapperClosable"
      class="drawer"
      @close="onClosePanl"
    >
      <section class="drawer-body">
        <!-- <a-form
          label-width="80px"
          :fields="fields"
          :data="data"
          :rules="formRules"
        >
        </a-form> -->

        <!-- el-form 是修改的内容 其他复制到表单组件原则上不做修改 -->
        <el-form
          ref="refForm"
          :model="dataItem"
          label-width="80px"
          :inline="false"
          size="small"
          :rules="formRules"
        >
          <el-form-item label="标题" prop="title">
            <el-input v-model="dataItem.title" />
          </el-form-item>

          <el-form-item label="描述" prop="description">
            <el-input v-model="dataItem.description" type="textarea" rows="2" />
          </el-form-item>
          <el-form-item label="编码" prop="code">
            <el-input v-model="dataItem.code" autocomplete="off" />
          </el-form-item>
          <el-form-item label="ICON" prop="icon">
            <el-input v-model="dataItem.icon" />
          </el-form-item>
          <el-divider content-position="left"></el-divider>
          <el-form-item label="父级" prop="parentId">
            <el-cascader
              v-model="dataItem.parentId"
              :options="parentOptions"
              :props="{
                checkStrictly: true,
                value: 'id',
                label: 'title',
                emitPath: false
              }"
              style="width:100%"
              filterable
              clearable
            ></el-cascader>
          </el-form-item>
          <el-form-item label="归属组" prop="groupId">
            <group-select v-model="dataItem.groupId"></group-select>
          </el-form-item>
          <el-divider content-position="left"></el-divider>
          <el-form-item label="资源类型" prop="resourceType">
            <el-radio-group v-model="dataItem.resourceType">
              <el-radio :label="1">资源分组</el-radio>
              <el-radio :label="2">资源菜单</el-radio>
              <el-radio :label="3">功能点</el-radio>
            </el-radio-group>
          </el-form-item>
          <el-form-item
            label="菜单类型"
            prop="linkType"
            v-if="dataItem.resourceType === 2"
          >
            <el-radio-group
              v-model="dataItem.linkType"
              v-if="dataItem.resourceType === 2"
            >
              <el-radio :label="1">视图组件</el-radio>
              <el-radio :label="2">外部链接</el-radio>
            </el-radio-group>
          </el-form-item>
          <el-form-item
            label="打开模式"
            prop="openMode"
            v-if="dataItem.resourceType === 2"
          >
            <el-radio-group v-model="dataItem.openMode">
              <el-radio :label="1">内部窗口</el-radio>
              <el-radio :label="2">外部窗口</el-radio>
            </el-radio-group>
          </el-form-item>

          <el-form-item
            label="Path"
            prop="path"
            v-if="
              dataItem.resourceType === 2 &&
                dataItem.linkType &&
                dataItem.resourceType === 2
            "
          >
            <el-input v-model="dataItem.path" />
          </el-form-item>
          <el-form-item
            :label="dataItem.linkType === 2 ? '外链URL' : '视图路径'"
            prop="viewPath"
            v-if="dataItem.resourceType === 2"
          >
            <el-input v-model="dataItem.viewPath" />
          </el-form-item>
          <el-form-item
            label="视图名称"
            prop="viewName"
            v-if="dataItem.resourceType === 2 && dataItem.linkType === 1"
          >
            <el-input v-model="dataItem.viewName">
              <el-button
                slot="append"
                icon="el-icon-bottom"
                @click="generateViewName"
              ></el-button>
            </el-input>
          </el-form-item>
          <el-form-item
            label="是否缓存"
            prop="viewCache"
            v-if="dataItem.resourceType === 2 && dataItem.linkType === 1"
          >
            <el-switch v-model="dataItem.viewCache" />
          </el-form-item>
          <el-form-item
            label="可否关闭"
            prop="closable"
            v-if="
              dataItem.resourceType === 2 &&
                (dataItem.linkType === 1 ||
                  (dataItem.linkType === 2 && dataItem.openMode === 1))
            "
          >
            <el-switch v-model="dataItem.closable" />
          </el-form-item>
          <el-divider content-position="left"></el-divider>

          <el-form-item label="隐藏" prop="isHidden">
            <el-switch v-model="dataItem.isHidden" />
          </el-form-item>
          <el-form-item label="禁用" prop="isDisabled">
            <el-switch v-model="dataItem.isDisabled" />
          </el-form-item>
        </el-form>
      </section>
      <div class="drawer-footer">
        <el-button @click.native="drawerVisible = false">
          {{ this.$t("common.cancel") }}
        </el-button>
        <confirm-button
          :validate="formValidate"
          :loading="loading"
          type="submit"
          @click="onSubmit"
        />
      </div>
    </el-drawer>
  </section>
</template>

<script>
// 组件
import ConfirmButton from "@/components/confirm-button";
import GroupSelect from "@/components/group-select";
// 工具
import { cloneDeep, merge } from "lodash";
// 接口
import { execCreate } from "@/api/admin/resource";
// 模块配置
import { model } from "../module-config";

export default {
  name: "admin-resource-add",
  components: {
    // AForm,
    ConfirmButton,
    GroupSelect
  },
  props: {
    title: {
      type: String,
      default: "新增"
    },
    visible: {
      type: Boolean,
      default: false
    },
    direction: {
      type: String,
      default: "rtl"
    },
    size: {
      type: String,
      default: "700px"
    },
    wrapperClosable: {
      type: Boolean,
      default: false
    },
    parentOptions: {
      type: Array
    }
  },
  computed: {
    drawerVisible: {
      get() {
        return this.visible;
      },
      set(v) {
        this.$emit("onChangeDrawer", v);
      }
    }
  },

  data() {
    //console.log(2, this.data);
    return {
      loading: false,
      dataItem: {},
      formRules: {
        title: [{ required: true, message: "请输入标题", trigger: "blur" }],
        code: [{ required: true, message: "请输入Code", trigger: "blur" }]
      }
    };
  },
  mounted() {
    this.setData();
  },
  methods: {
    setData(item) {
      if (item) {
        this.dataItem = merge(cloneDeep(model), item);
      } else {
        this.dataItem = cloneDeep(model);
      }
    },
    // 验证表单
    formValidate: function() {
      let isValid = false;
      this.$refs.refForm.validate(valid => {
        isValid = valid;
      });
      return isValid;
    },
    // 关闭表单面板重置表单控件
    onClosePanl() {
      this.$refs.refForm.resetFields();
    },
    async onSubmit() {
      this.loading = true;
      const para = cloneDeep(this.dataItem);
      const res = await execCreate(para);
      this.loading = false;

      if (res.success) {
        this.$message({ message: this.$t("common.addOk"), type: "success" });
        this.$refs.refForm.resetFields();
        this.drawerVisible = false;
        // 成功后钩子，共父级调用
        this.$emit("onSuccess", para, res);
      } else {
        this.$message({ message: res.message, type: "error" });
        // 失败后钩子，共父级调用
        this.$emit("onError", para, res);
      }
    },
    generateViewName() {
      if (!this.dataItem.viewPath || this.dataItem.viewPath === "") {
        this.$message({
          message: "视图路径为空，不能生成视图名称",
          type: "warning"
        });
        return;
      }

      let str = this.dataItem.viewPath;
      str = str.startsWith("/") ? str.replace(/\//, "") : str;
      str = str.replace(/\//g, "--");
      this.dataItem.viewName = str;
    }
  }
};
</script>
