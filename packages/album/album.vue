<template>
	<div class="sun_upload">
		<el-button class="sun_choose" size="small" type="primary" @click="dialogVisible = true">从图片空间选择</el-button>
		<el-upload
		  	class="sun_upload-demo sun_a_none"
			action="http://v20-dev2.shunliandongli.com/uploads/uploadimage" 
		  	:on-success="handlePreview"
		  	:on-remove="handleRemove"
		  	with-credentials
		  	:file-list="fileList2"
		  	show-file-list
		  	list-type="picture">
		  <el-button size="small" type="primary">点击上传</el-button>
		  <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
		</el-upload>
		

		<!-- 从相册选择 -->
		<el-dialog
			title="我的图片"
			:visible.sync="dialogVisible"
			top="3%"
			@open="getListData"
			class="sun_notPadding"
		>
			<div class="sun_top">
				<!-- <el-button type="primary" @click="clickFile()"  size="small">上传图片</el-button> -->
				<!-- <input type="file" name="file" id="in_file" style="display:none" @change="file_change($event)"> -->

				<el-upload
				  	class="sun_shangc"
					action="http://v20-dev2.shunliandongli.com/uploads/uploadimage" 
				  	with-credentials
				  	:data="img_success_data"
				  	:on-success="img_success"
				  	list-type="picture">
				  <el-button size="small" type="primary">上传图片</el-button>
				</el-upload>

				<el-button type="primary" size="small" @click="create_status = true">创建相册</el-button>
			</div>
			<div class="sun_main">
				<div class="sun_albumlist">
					<ul>
						<li :class="{active:active == index}" v-for="(item,index) in albumlist" @click="liClick(item.album_id,index)">
							<span class="sun_name">{{item.album_name}}</span>
							<i class="sun_num">{{item.num}}</i>
						</li>
					</ul>
				</div>
				<div class="sun_imglist">
					<div class="sun_img_box"  v-for="(item,index) in imgList">
						<i style="color:#20A0FF;" v-if="choose_arr.indexOf(index)!=-1" class="el-icon-circle-check"></i>
						<img :src="item.imgurl" alt="" @click="chooseImg($event,index)">
					</div>
					<el-pagination
						@current-change="handleCurrentChange"
						:current-page.sync="currentPage2"
						layout="prev, pager, next, jumper"
						:total="total">
					</el-pagination>
				</div>
			</div>
			<span slot="footer" class="dialog-footer">
				<el-button @click="dialogVisible = false">取 消</el-button>
				<el-button type="primary" @click="dialogVisible = false">确 定</el-button>
			</span>
		</el-dialog>

		<!-- 创建相册 -->
		<el-dialog
			title="我的图片"
			:visible.sync="create_status"
			size="tiny"
		>
			<el-form :model="form" :rules="rules" ref="ruleForm">
				<el-form-item label="相册名称" prop="album_name">
					<el-input v-model="form.album_name"></el-input>
				</el-form-item>
			</el-form>
			<span slot="footer" class="dialog-footer">
				<el-button @click="create_status = false">取 消</el-button>
				<el-button type="primary" @click="create_album('ruleForm')">确 定</el-button>
			</span>
		</el-dialog>
	</div>
</template>
<script>
import axios from 'axios'
export default{
	data() {
		return {
			fileList2: [],
			dialogVisible:false,
			create_status:false,
			active:0,
			albumlist:null,
			// album_id:null,
			imgList:null,
			currentPage2:1,
			pagesize:null,
			total_count:null,
			page:1,
			form:{
				album_name:null
			},
			rules:{
				album_name:[
					{required: true, message: '请输入相册名称', trigger: 'blur'}
				]
			},
			img_success_data:{
				album_id:null
			},
			choose_arr:[],
			imgArr:[]
		};
	},
	computed:{
		total:function(){
			let num = Math.ceil(this.total_count/this.pagesize)*10;
			return num;
		}
	},
	methods: {
		handleRemove(file, fileList) {
			this.fileList2=fileList
			// console.log(file, fileList);
		},
		handlePreview(response, file, fileList) {
			console.log(response)
			let url = response.data.domain+response.data.relativePath[0]
			let img_obj={
				url:response.data.domain+response.data.relativePath[0]
			}
			this.fileList2.push(img_obj)
			this.imgArr.push(url)
			this.$emit("child-data",this.imgArr)
			// console.log(response, file, fileList);
		},
		clickFile(){
			document.getElementById("in_file").click();
		},
		liClick(album_id,index){
			//console.log(album_id,index)
			this.img_success_data.album_id=album_id;
			this.active=index;
			this.getAlbumimagelist();
		},
		handleCurrentChange(val){
			this.page=val;
			this.getAlbumimagelist();
		},
		getAlbumlist(){
			axios.get("http://v20-dev2.shunliandongli.com/product/albumlist").then((response) => {
				//console.log(response)
				this.albumlist=response.data.data;
				if(!this.img_success_data.album_id){
					this.img_success_data.album_id=response.data.data[0].album_id;
				}
				this.getAlbumimagelist();
			}).catch((err) => {
				console.log(err)
			})
		},
		getAlbumimagelist(){
			console.log(this.img_success_data.album_id)
			axios.post("http://v20-dev2.shunliandongli.com/product/albumimagelist",{"page": this.page,"pagesize": 15,"album_id": this.img_success_data.album_id}).then((response) => {
				console.log(response)
				this.imgList=response.data.data.list;
				this.pagesize=response.data.data.pagesize;
				this.total_count=response.data.data.total_count;
			}).catch((err) => {
				console.log(err)
			})
		},
		getListData(){
			this.getAlbumlist()
			this.choose_arr=[];
		},
		//创建相册
		create_album(formName){
			this.$refs[formName].validate((valid) => {
				if (valid) {
					axios.post("http://v20-dev2.shunliandongli.com/product/addalbum",{album_name:this.form.album_name}).then((response) => {
						//console.log(response)
						if(response.data.success){
							this.create_status = false;
							this.getAlbumlist()
						}else{
							this.$message.error("创建失败")
						}
					}).catch((err) => {
						console.log(err)
					})
				} else {
					console.log('error submit!!');
					return false;
				}
			});
		},
		img_success(response){
			let url = response.data.domain+response.data.relativePath[0];
			this.getAlbumlist()
		},
		chooseImg(event,index){
			this.choose_arr.push(index)
			console.log(event)
			let img_obj={
				url:event.target.currentSrc
			}
			let url = event.target.currentSrc;
			this.imgArr.push(url);
			this.$emit("child-data",this.imgArr)
			this.fileList2.push(img_obj)
		}
	},
	created(){
		
	}
}
</script>
<style>
.sun_upload{
	position: relative;
}
.sun_upload .sun_choose{
	position: absolute;
	top: 0;
	left: 75px;
}
.el-upload-list--picture .el-upload-list__item{
	width: 200px!important;
	float: left!important;
	margin-right: 10px;
}
.sun_upload .sun_top{
	border-bottom: 1px solid #bfcbd9;
	padding: 20px 0 10px;
	overflow: hidden;
}
.sun_upload .sun_top button{
	float: right;
	margin-right: 20px;
}
.sun_upload .el-dialog--small{
	width: 846px!important;
}
.sun_upload .sun_notPadding .el-dialog__body{
	padding:0 0!important;
}
.sun_upload .sun_main{
	overflow: hidden;
	border-bottom: 1px solid #bfcbd9;
}
.sun_upload .sun_albumlist{
	width: 156px;
	height: 450px;
	float: left;
	box-sizing: border-box;
	border-right: 1px solid #bfcbd9;
	background: #ddd;
	overflow: scroll;
}
.sun_upload .sun_albumlist ul li{
	height: 30px;
	padding: 8px;
	line-height: 30px;
	display: flex;
	justify-content: space-between;
	align-items: center;
	cursor: pointer;
}
.sun_upload .sun_albumlist ul li.active{
	background: #fff;
}
.sun_upload .sun_albumlist ul li .sun_num{
	width: 20px;
	height: 20px;
	border-radius: 50%;
	background: #20A0FF;
	text-align: center;
	line-height: 20px;
	font-size: 12px;
	color: white;
}
.sun_upload .sun_imglist{
	width: 690px;
	height: 450px;
	box-sizing: border-box;
	padding: 20px 20px;
	float: left;
	position: relative;
}
.sun_upload .sun_imglist .sun_img_box{
	width: 100px;
	height: 100px;
	margin-right: 20px;
	margin-bottom: 20px;
	overflow: hidden;
	display: inline-block;
	position: relative;
	cursor: pointer;
}
.sun_upload .sun_imglist .sun_img_box i{
	position: absolute;
	left: 5px;
	top: 5px;
}
.sun_upload .sun_imglist .sun_img_box img{
	width: 100%;
	height: 100%;
}
.sun_upload .sun_imglist .el-pagination{
	position: absolute!important;
	bottom: 20px!important;
	left: 0!important;
	width: 100%!important;
	display: flex!important;
	justify-content: center!important;
}
.sun_upload .sun_shangc .el-upload-list{
	display: none!important;
}
.sun_upload .sun_shangc{
	float: right;
}
.sun_upload .sun_a_none .el-upload-list__item-name{
	display: none!important;
}
.sun_upload .sun_a_none .el-upload-list__item{
	width: 93px!important;
	padding-left: 10px!important;
}
.sun_upload .sun_a_none .el-upload-list__item img{
	margin-left: 0!important;
}
.sun_upload .sun_a_none .el-upload-list__item-status-label{
	z-index: 100!important;
}
.sun_upload .sun_a_none .el-upload-list--picture .el-progress{
	top: 33px!important;
}
.sun_upload .sun_a_none .el-upload-list__item .el-icon-close{
	z-index: 100!important;
}
</style>