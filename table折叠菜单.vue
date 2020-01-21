<template>
    <div class="app-container">
        <el-collapse accordion v-model="activeName">
            <el-collapse-item name='1'>
                <template class="controller" slot="title">
                点此添加新的专题
                </template>
                <el-form ref="form" class="addInterested" :model="form" size="mini" fit label-width="80px" style="width:50%;">
                    <el-form-item label="专题名称">
                        <el-input v-model="form.title"></el-input>
                    </el-form-item>
                    
                    <el-form-item label="专题图标">
                        <el-upload
                            class="upload-demo"
                            action="/management/pictures"
                            :on-change="handleChange"
                            :auto-upload="true"
                            :http-request="handleUpload"
                            list-type="picture-card"
                            :limit=1
                            :file-list="fileList">
                            <el-button size="small" type="primary">点击上传</el-button>
                            <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
                        </el-upload>
                    </el-form-item>

                    <!-- <el-form-item label="ico背景色">
                        <el-color-picker color-format="hex" @click="changColor" v-model="form.color" size="small"></el-color-picker>
                </el-form-item>  -->

                    <el-form-item label="专题描述">
                        <el-input type="textarea" v-model="form.description"></el-input>
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" @click="onSubmit">确认</el-button>
                        <el-button @click="onReset">重置</el-button>
                    </el-form-item>
                    
                </el-form>

          </el-collapse-item>
</el-collapse>

        <el-table 
        ref="table"
        :data="specialList" 
        stripe
        style="width: 100%"
        
        >
            <!-- <el-table-column label="标识" width="100">
                <template slot-scope="scope">
                <span style="margin-left: 0px">{{ scope.row.sub_id }}</span>
                </template>
            </el-table-column> -->

            <el-table-column label="专题名" width="300">
                <template slot-scope="scope">
                <span style="margin-left: 0px">{{ scope.row.title }}</span>
                </template>
            </el-table-column>

            <el-table-column label="图标" width="120">
                <template slot-scope="scope">
                    <img :src="scope.row.cover_url" alt="" class="icon_url">
                </template>
            </el-table-column>
            <el-table-column label="文章数" width="">
                <template slot-scope="scope">
                <span style="margin-left: 0px">{{ scope.row.news_count }}篇</span>
                </template>
            </el-table-column>
            <el-table-column label="专题说明" width="">
                <template slot-scope="scope">
                <span style="margin-left: 0px">{{ scope.row.description }}</span>
                </template>
            </el-table-column>

            <el-table-column label="操作" width="400">
                <template slot-scope="scope">
                    <el-button
                    size="mini"
                    @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                    <el-button
                    size="mini"
                    @click="handleAdd(scope.$index, scope.row)">发布</el-button>
                    <el-button
                    size="mini"
                    @click="getSpecialListArticles(scope.row)">查看文章</el-button>
                    <el-button
                    size="mini"
                    type="danger"
                    @click="handleDelete(scope.$index, scope.row)">删除</el-button>
                </template>
            </el-table-column>
            


            <!-- 展开行 -->

            <el-table-column type="expand" width="1">
                <template slot-scope="props">
                    <!-- 这里显示文章列表 -->
                    <el-table
                        empty-text="当前专题暂无文章"
                         v-loading="specialListArticlesLoading"
                        :data="specialListArticles"
                        height="250"
                        border
                        style="width: 100%">
                            <el-table-column
                            prop="news_id"
                            label="文章编号"
                            >
                            </el-table-column>
                            <el-table-column
                            prop="title"
                            label="文章标题"
                            >
                            </el-table-column>
                            <el-table-column
                            prop="news_type"
                            label="文章类型"
                            >
                            </el-table-column>
                            <el-table-column
                            prop="post_time"
                            label="发布日期"
                            >
                            </el-table-column>
                            <el-table-column
                            label="操作"
                            >
                                <el-button
                                size="mini"
                                @click="handleEditSpeArt(props.row)">编辑</el-button>

                                <el-button
                                size="mini"
                                type="danger"
                                @click="handleDeleteSpeArt(props.$index, props.row)">删除</el-button>

                            </el-table-column>

                            <!-- <el-table-column
                            prop="post_time"
                            label="新闻源">
                            </el-table-column>
                            <el-table-column
                            prop="post_time"
                            label="媒体类型">
                            </el-table-column>
                            <el-table-column
                            prop="post_time"
                            label="操作">
                            </el-table-column> -->
                    </el-table>

                </template>
            </el-table-column>
        </el-table>

        <div class="block pagerr">
            <el-pagination
            background
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="currentPage4"
            :page-sizes="[10, 50, 100, 200]"
            :page-size="pageSize"
            layout="total, prev, pager, next, jumper"
            :total="total">
            </el-pagination>
        </div>


    </div>
</template>

<script>

import { uploadImage } from '@/api/article'//
import { getSpecialList ,newSpecial,updateSpecial,deleteSpecial,getSpecialArticlesFnc } from '@/api/special'//
import { constants } from 'crypto';
import { setTimeout } from 'timers';
export default {
    name: 'interested',
    data(){
        return {
            specialList: [],
            form: {
                title: ``,
                cover_url: ``,
                description: ``,
                sub_id: null,
                
            },
            fileList: [
                // {
                // name: 'food.jpeg',
                // url: 'https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100',
                // status: 'finished'
                // }
                ],
            currentPage4: 1,
            total: 0,
            pageSize: 10,
            pageIndex: 1,
            
            activeName: '',
            specialListArticles: [],//专题下文章列表
            specialListArticlesLoading: true
        }
    },
    methods: {
        //点击编辑
        handleEdit(index, row) {
            console.log( row );
            this.activeName = `1`;
            this.form = {
                title: `${row.title}`,
                cover_url: `${row.cover_url}`,
                description: `${row.description}`,
                id: `${row.sub_id}`
            }
            this.fileList = [{url: `${row.cover_url}`}]

        },
        //在该专题下发布新文章
        handleAdd(index,row){
            console.log( index, row );
            let sub_id = row.sub_id;
            this.$router.push(`/articles/newarticle?sub_id=${sub_id}`);
        },
        handleDelete(index, row) {
           deleteSpecial({
               id: row.sub_id
           }).then(e=>{
               if( e.status_code == 200 ){
                    this.$message({
                        message: `删除成功`,
                        type: "success",
                        duration: 5000
                    });
                    this.form = {
                        title: ``,
                        cover_url: ``,
                        description: ``,
                        id: null
                    }
                    this.fileList = [];
                    this.getData();
               }
           });
        },
    
        onSubmit() {
            console.log(this.form);
            if( !this.form.title || !this.form.cover_url ||! this.form.description ){
                this.$message.error('专题名称图标以及专题描述均需要填写');
                return false;   
            }
        if( this.form.id ){
            //存在id  走更新分支
            updateSpecial({
                title: this.form.title,
                cover_url: this.form.cover_url,
                description: this.form.description,
                id: this.form.id
            }).then(e=>{
                if(e && e.status_code == 200){
                    this.$message({
                        message: `添加成功`,
                        type: "success",
                        duration: 5000
                    });
                    this.form = {
                        title: ``,
                        cover_url: ``,
                        description: ``,
                        id: null
                    }
                    this.fileList = [];
                    this.getData();
                }else{
                    this.$message({
                        message: `添加失败`,
                        type: "warning",
                        duration: 5000
                    });
                };
            })
            return false;
        }

            newSpecial({
                title: this.form.title,
                cover_url: this.form.cover_url,
                description: this.form.description,
                id: this.form.id
            }).then(e=>{
                if(e && e.status_code == 200){
                    this.$message({
                        message: `添加成功`,
                        type: "success",
                        duration: 5000
                    });
                    this.form = {
                        title: ``,
                        cover_url: ``,
                        description: ``,
                        id: null
                    }
                    this.fileList = [];
                    this.getData();
                }else{
                    this.$message({
                        message: `添加失败`,
                        type: "warning",
                        duration: 5000
                    });
                };
            })
            // .catch(err=>{
            //     console.log(err);
            //     if( JSON.stringify(err).includes('400') ){
            //         this.$message({
            //             message: `已经存在`,
            //             type: "warning",
            //             duration: 5000
            //         });
            //     }else if( JSON.stringify(err).includes('405') ){
            //         this.$message({
            //             message: `PUT方法不被允许,如要修改此兴趣请删除后重新添加`,
            //             type: "warning",
            //             duration: 5000
            //         });
            //     }
            // });
        },
        onReset(){
            console.log('onReset!');
            this.form = {
                title: ``,
                cover_url: ``,
                description: ``,
                id: null
            }

            this.fileList = [];
        },

        //上传图片相关
        handleChange(file, fileList) {
            console.log(file,fileList);
            
        },

        handleUpload(item){
            let formData = new FormData();
            formData.append("file", item.file);
            uploadImage(formData).then( result =>{
                console.log("上传结果", result);
                this.form.cover_url = result.data.url_name;
            });
        },

         handleSizeChange(val) {
            console.log(`每页 ${val} 条`);
            this.pageSize = val;
            this.created;
        },
        handleCurrentChange(val) {
            console.log(`当前页: ${val}`);
            this.pageIndex = val;
            this.getData();
        },
        getData(){
            getSpecialList(this.pageSize,this.pageIndex).then(e=>{
                console.log(e);
                this.specialList = e.data;
                this.total = e.count?e.count:0;
            });
        },

        changColor(e){
            console.log(e);
        },

        //获取专题下文章列表

        getSpecialListArticles(row){
            this.specialListArticlesLoading = true;
            this.specialListArticles = [];

            let $table = this.$refs.table;
            // console.log(this.specialList );
            this.specialList.map(e=>{
                // console.log( row,e );
                if (Number( row.sub_id )!= Number( e.sub_id )) {
                   $table.toggleRowExpansion(e, false)
                 }
            });
            $table.toggleRowExpansion(row);

            setTimeout(()=>{
                //获取相应数据
                getSpecialArticlesFnc(row.sub_id).then(e=>{
                    this.specialListArticles = e.data?e.data:[];
                }).then(()=>{
                    this.specialListArticlesLoading = false;
                });
            },500);
            
            
            
        },

        handleEditSpeArt( id ){
            console.log( id );
            // let id = row.



        }






    },
    created(){
        this.getData();
        
    }
}
</script>

<style scoped>
.icon_url{
    width: 50px;
}
.addInterested{
    margin-left: auto;
    margin-right: auto;
    border: 1px solid lightblue;
    padding: 50px;
    margin-bottom: 50px;
}

/* 上传图片 */


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

.pagerr{
    padding: 30px 24px;
    border-bottom: 1px solid #eff2f6;
}

.controller{
    text-align: center;
    margin-left: auto;
    margin-right: auto;
    background-color: #409EFF;
}
  
</style>