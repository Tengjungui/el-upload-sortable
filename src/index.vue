<style lang="scss" scoped>
.upload-queue{
  .el-upload-list__item:hover{
    .el-upload-list__item-actions{
      display: block;
      opacity: 1;
    }
  }
}
</style>

<template>
<div class="upload-queue" @change="updateList(list)">
  <draggable v-model="list" @start="drag=true" @end="drag=false" 
    class="el-upload-list el-upload-list--picture-card">
    <div class="el-upload-list__item" v-for="(item, index) in list" :key="index">
      <img :src="item" class="el-upload-list__item-thumbnail">

      <span class="el-upload-list__item-actions">
        <span class="el-upload-list__item-preview" @click="handlePreview(item)">
          <i class="el-icon-zoom-in"></i>
        </span>
        <span class="el-upload-list__item-delete" @click="handleRemove(item, index)">
          <i class="el-icon-delete"></i>
        </span>
      </span>
    </div>
  </draggable>
  <el-upload
    v-if="list.length < max"
    class="el-upload el-upload--picture-card"
    :action="action"
    :show-file-list="false"
    :on-success="handleSuccess"
    :before-upload="beforeUpload">
    <i class="el-icon-plus"></i>
  </el-upload>

  <el-dialog :visible.sync="dialogVisible">
    <img width="100%" :src="dialogImageUrl">
  </el-dialog>
</div>
</template>

<script>
import draggable from "vuedraggable";

export default {
  name: 'ElUploadSortable',
  components: { draggable },
  props: {
    max: {
      type: Number,
      default: 15
    },
    action: {
      type: String,
      default: 'https://jsonplaceholder.typicode.com/posts/'
    }
  },

  data () {
    return {
      list: [],
      drag: false,
      dragOptions: {
        animation: 200,
        group: "description",
        disabled: false,
        ghostClass: "ghost"
      },
      dialogImageUrl: "",
      dialogVisible: false
    }
  },
 
  methods: {
    updateList(list){
      this.$emit('change', list)
    },
    beforeUpload(file) {
      const isValidFormat = ["image/jpeg", "image/png"].indexOf(file.type) > -1;
      const isLt2M = file.size / 1024 / 1024 < 2; // 2M

      if (!isValidFormat) {
        this.$message.error("图片只能是 JPG或PNG 格式!");
      } else if (!isLt2M) {
        this.$message.error("图片大小不能超过 2MB!");
      }

      const maxLt = this.max === 1 && this.list.length > 0;
      if(maxLt){
        this.$message.error("只能上传一张图片，请删除后再上传!");
      }
      return isValidFormat && isLt2M && !maxLt;
    },

    handleSuccess(res, file) {
      this.list.push(URL.createObjectURL(file.raw));
      this.$emit('change', this.list)
    },

    handleRemove(file, index) {
      this.list.splice(index, 1);
      this.$emit('change', this.list)
    },

    handlePreview(url) {
      this.dialogImageUrl = url;
      this.dialogVisible = true;
    }
  }
}
</script>