<template>
    <div class="about">
        <h1>{{id ? 'Edit':'New-built'}}Alarm</h1>
        <el-form label-width="120px" @submit.native.prevent="save">
            <el-form-item label="Describe">
                <el-input v-model="model.name"> </el-input>
            </el-form-item>
            <el-form-item label="Position">
                <el-input v-model="model.coordinate"> </el-input>
            </el-form-item>
            <el-form-item label="Site photos">
                <el-upload
                    class="avatar-uploader"
                    :action="$http.defaults.baseURL + '/upload'"
                    :show-file-list="false"
                    :on-success="afterUpload"
                    >
                    <img v-if="model.icon" :src="model.icon" class="avatar">
                    <i v-else class="el-icon-plus avatar-uploader-icon"></i>
                </el-upload>
            </el-form-item>
            <el-form-item>
                <el-button type="primary" native-type="submit">Save</el-button>
            </el-form-item>
        </el-form>
    </div>
</template>

<script>
export default {
    props:{
        id:{}
    },
    data(){
        return{
            model:{},
        }
    },
    methods:{
        afterUpload(res){
            this.$set(this.model, 'icon', res.url)
        },
        async save(){
            let res
            if(this.id){
                res = await this.$http.put(`rest/items/${this.id}`, this.model)
            }else{
                res = await this.$http.post('rest/items', this.model)
            }

            this.$router.push('/items/list')
            this.$message({
                type:'success',
                message: 'Saved successfully'
            })
        },
        async fetch(){
            const res = await this.$http.get(`rest/items/${this.id}`)   
            this.model = res.data
        },
    },
    created(){
        this.id && this.fetch()
    }
}
</script>

<style>
  .avatar-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }
  .avatar-uploader .el-upload:hover {
    border-color: #409EFF;
  }
  .avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 178px;
    height: 178px;
    line-height: 178px;
    text-align: center;
  }
  .avatar {
    width: 178px;
    height: 178px;
    display: block;
  }
</style>