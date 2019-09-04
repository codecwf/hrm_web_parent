<template>
	<section>
		<!--工具条-->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :model="filters" :inline="true">
				<el-form-item>
					<el-input v-model="filters.keyword" placeholder="关键字" ></el-input>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" v-on:click="getTenantTypes">查询</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="add">新增</el-button>
				</el-form-item>
			</el-form>
		</el-col>

		<!--private Employee manager;                               // 部门经理 员工对象-->
		<!--private Department parent;                              // 上级部门 部门对象-->
		<!--private Tenant tenant;                                  // 租户-->


		<!--列表v-loading="listLoading"-->
		<el-table :data="tenantTypes" v-loading="listLoading" highlight-current-row  style="width: 100%;">
			<!--多选框-->
			<el-table-column type="selection" width="55">
			</el-table-column>
			<!--索引值,为什么不用id,id不序号-->
			<el-table-column type="index" width="60">
			</el-table-column>
			<!--其他都设置值,只有一个不设置值就自动适应了-->
			<el-table-column prop="name" label="名称">
			</el-table-column>
			<el-table-column prop="description" label="描述">
			</el-table-column>

			<el-table-column label="操作" width="150">
				<template scope="scope">
					<el-button size="small"  @click="edit(scope.row)">编辑</el-button>
					<el-button type="danger" size="small" @click="del(scope.row)">删除</el-button>
				</template>
			</el-table-column>
		</el-table>
		<!--工具条-->
		<el-col :span="24" class="toolbar">
			<el-button type="danger">批量删除</el-button>
			<el-pagination layout="prev, pager, next" @current-change="handleCurrentChange"  :page-size="10" :total="total" style="float:right;">
			</el-pagination>
		</el-col>

		<!--//多对一-->
		<!--private Employee manager;                               // 部门经理 员工对象-->
		<!--后台提供获取所有员工-->
		<!--//多对一-->
		<!--private Department parent;                              // 上级部门 部门对象-->

		<!--添加或编辑对话框-->
		<el-dialog title="添加或修改" :visible.sync="formVisible" :close-on-click-modal="false">
			<el-form :model="tenantType" label-width="80px" :rules="formRules" ref="tenantType">
				<el-form-item label="名称" prop="name">
					<el-input v-model="tenantType.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="描述" prop="description">
					<el-input v-model="tenantType.sn" auto-complete="off"></el-input>
				</el-form-item>

			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="formVisible = false">取消</el-button>
				<el-button type="primary" @click="save" >提交</el-button>
			</div>
		</el-dialog>
	</section>
</template>

<script>
	export default {
		data() {
			return {
                formVisible:false,//对话框默认不显示,只有点击添加或修改的时候显示
                listLoading:false,
				//查询对象
				filters:{
					keyword:''
				},
				page:1,//当前页,要传递到后台的
				total:0, //分页总数
			    tenantTypes:[], //当前页数据
				//初始值
                tenantType:{
                    id:null,
					name:'',
					sn:'',
					dirPath:'',
					state:0,
                    manager:null,
				},
				employees:[],
                formRules: {
                    name: [
                        { required: true, message: '请输入名称!', trigger: 'blur' }
                    ]
                }
			}
		},
		methods: {
			add(){
				//清空数据
				this.tenantType={
					id:null,
					name:'',
					sn:'',
					dirPath:'',
					state:0,
					manager:null,
				}
				//打开dialog
				this.formVisible =true;
				this.getEmployees();
			},
            stateFormatter(row, column, cellValue, index){

                if(cellValue===0){
                    return "正常";
				}else{
                    return "停用";
				}
			},
            handleCurrentChange(curentPage){
                this.page = curentPage;
                this.getTenantTypes();
			},
		    save(){
                this.$refs.tenantType.validate((valid) => {
                    //校验表单成功后才做一下操作
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            //拷贝后面对象的值到新对象,防止后面代码改动引起模型变化
                            let para = Object.assign({}, this.tenantType);
                            //判断是否有id有就是修改,否则就是添加
							this.$http.put("/sysmanage/tenantType",para).then((res) => {
								this.$message({
									message: '操作成功!',
									type: 'success'
								});
								//重置表单
								this.$refs['tenantType'].resetFields();
								//关闭对话框
								this.formVisible = false;
								//刷新数据
								this.getTenantTypes();
							});
                        });
                    }
                })
			},
            edit(row){
                //this.getTenantTypes();
                //回显
                let tenantTypeTmp = Object.assign({}, row); //解决对话框改值后列表会被改值.
                this.tenantType = tenantTypeTmp; //里面本来就有id,相当于回显了id
				//显示
                this.formVisible =true;
			},
			 //  getTenantTypes(){
             //    //发送请求到后台获取数据
             //      this.$http.patch("/employee") //$.Post(.....)
             //          .then(result=>{
             //              this.tenantTypes = result.data;
             //          });
			 //  }
		    // ,
            getTenantTypes(){
                //发送Ajax请求后台获取数据  axios
				//添加分页条件及高级查询条件
				let para = {
				    "page":this.page,
					"keyword":this.filters.keyword
				};
				this.listLoading = true; //显示加载圈
				//分页查询
                this.$http.patch("/sysmanage/tenantType",para) //$.Post(.....)
                    .then(result=>{
                        console.log(result.data); //data就是 PageListjson转换结果
                        this.total = result.data.total;
                        this.tenantTypes = result.data.rows;
                        this.listLoading = false;  //关闭加载圈
                    });

			},
			del(row){
                console.log(row);
                this.$confirm('确认删除该记录吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    var id = row.id;
                    this.listLoading = true;
                    this.$http.delete("/sysmanage/tenantType/"+id)
                        .then(result=>{
                            this.listLoading = false;
                            //做提示
                            if(result.data.success){
                                this.$message({
                                    message: '删除成功',
                                    type: 'success'
                                });
							}else{
                                this.$message({
                                    message: result.data.message,
                                    type: 'error'
                                });
							}
							//刷新数据
                            this.getTenantTypes();
                        })
				});

			}
		},
		mounted() {
		    this.getTenantTypes()
		}
	}

</script>

<style scoped>

</style>