<style lang="less">
    @import '../../styles/common.less';
</style>

<template>
    <div>
        <Row class="margin-top-10">
            <Col span="24">
            <Card dis-hover>
                <Row>
                    <Select style="width:110px" v-model="productId" v-show="productShow">
                        <Option v-for="item in productData" :value="item.id" :key="item.id">{{ item.name }}</Option>
                    </Select>
                    <Select style="width:200px" v-model="projectId" v-show="projectShow">
                        <Option v-for="item in projectData" :value="item.id" :key="item.id">{{ item.name }}</Option>
                    </Select>
                    <div style="float: right;">
                        <slot name="create"></slot>
                        <slot name="downMenu"></slot>
                        <Button type="primary" @click="refresh()">刷新</Button>
                        <Button type="primary" @click="synchronize()">备份</Button>
                        <Modal slot="option" v-model="synchronizeView" :title="optionTypeName" width="600px">
                            <Form ref="formSynchronizeValidate" :model="formSynchronizeValidate"
                                  :rules="ruleSynchronizeValidate" :label-width="65">
                                <FormItem label="目标地址" prop="desc_path">
                                    <Input v-model="formSynchronizeValidate.desc_path"
                                           placeholder="输入备份的目标地址,默认到 /usr/local/"></Input>
                                </FormItem>
                                <FormItem label="目标" prop="target">
                                    <Select v-model="formSynchronizeValidate.target" multiple>
                                        <Option v-for="item in hostData" :value="item.id" :key="item.id"
                                                placeholder="选择目标">{{ item.minion_id }}
                                        </Option>
                                    </Select>
                                </FormItem>
                            </Form>
                            <div slot="footer">
                                <Button type="primary" @click="handleSynchronize('formValidate')">提交</Button>
                            </div>
                        </Modal>
                        <Modal slot="option" v-model="distributeView" :title="optionTypeName" width="600px">
                            <Form ref="formDistributeValidate" :model="formDistributeValidate"
                                  :rules="ruleDistributeValidate" :label-width="65">
                                <FormItem label="目标地址" prop="desc_path">
                                    <Input v-model="formDistributeValidate.desc_path"
                                           placeholder="输入分发的目标地址文件夹,如 /usr/local/prometheus/, 会覆盖该目录下同名文件"></Input>
                                </FormItem>
                                <FormItem label="目标" prop="target">
                                    <Select v-model="formDistributeValidate.target" multiple>
                                        <Option v-for="item in targetData" :value="item.id" :key="item.id"
                                                placeholder="选择目标">{{ item.name }}
                                        </Option>
                                    </Select>
                                </FormItem>
                            </Form>
                            <div slot="footer">
                                <Button type="primary" @click="handleDistribute('formValidate')">提交</Button>
                            </div>
                        </Modal>
                    </div>
                </Row>
                <Row>
                    <hr class="hr-margin" color="#e3e8ee" size="0.5">
                </Row>
                <Row :gutter="5">
                    <Col span="5">
                    <Card dis-hover :padding="6">
                        <div style="margin-bottom: -10px;">
                            <Select v-model="branchName">
                                <Option v-for="item in branchData" :value="item" :key="item">{{ item }}</Option>
                            </Select>
                        </div>
                        <br>
                        <Tree :data="fileTree" :load-data="loadData" @on-select-change="handleContent"></Tree>
                    </Card>
                    </Col>
                    <Col span="19">
                    <Card dis-hover>
                        <Form ref="formValidate" :model="formValidate" :rules="ruleValidate" :label-width="46">
                            <FormItem label="文件" prop="fileDir">
                                <Input v-model="formValidate.fileDir" :disabled="inputDisabled"
                                       placeholder="点击左侧树型结构获取目录，输入文件名或者路径，如：a/b/top.sls，目录不存在自动创建"></Input>
                            </FormItem>
                            <FormItem label="内容" prop="code">
                                <Tabs v-model="tab" :style="[h]">
                                    <TabPane label="从文本输入框创建" name="text">
                                        <MonacoEditor
                                                height="500"
                                                width="100%"
                                                language="yaml"
                                                srcPath="dist"
                                                :code="fileContent"
                                                :options="options"
                                                :highlighted="highlightLines"
                                                :changeThrottle="100"
                                                theme="vs-dark"
                                                @mounted="onMounted"
                                                @codeChange="onCodeChange"
                                                ref="vscode"
                                        >
                                        </MonacoEditor>
                                        <br>
                                        <Button type="primary" @click="handleCreate('formValidate')"
                                                :disabled="createDisabled">创建
                                        </Button>
                                        <Button type="primary" @click="handleEdit('formValidate')"
                                                :disabled="editDisabled">更新
                                        </Button>
                                        <Button type="primary" @click="handleDistributeView('formValidate')"
                                                :disabled="editDisabled">分发
                                        </Button>
                                        <Poptip
                                                confirm
                                                :title="title"
                                                @on-popper-show="PopperShow()"
                                                @on-ok="handleDelete('formValidate')">
                                            <Button type="error" :disabled="deleteDisabled">删除</Button>
                                        </Poptip>
                                    </TabPane>
                                    <TabPane label="从文件创建" name="upload" :disabled="uploadDisabled">
                                        <div style="padding: 1px">
                                            <Upload
                                                    multiple
                                                    type="drag"
                                                    :action="action"
                                                    :data="uploadParameter"
                                                    :with-credentials="true"
                                                    :on-success="UploadSuccess"
                                                    :on-error="UploadError"
                                                    :before-upload="beforeUpdate">
                                                <div style="padding: 10px 0px">
                                                    <Icon type="ios-cloud-upload" size="52"
                                                          style="color: #3399ff"></Icon>
                                                    <p>点击或者拖拽上传</p>
                                                </div>
                                            </Upload>
                                        </div>
                                    </TabPane>
                                </Tabs>
                            </FormItem>
                        </Form>
                    </Card>
                    </Col>
                </Row>
            </Card>
            </Col>
        </Row>
    </div>

</template>

<script>
    import MonacoEditor from 'vue-monaco-editor';

    function nCopy(data) {
        return JSON.parse(JSON.stringify(data));
    };
    export default {
        components: {
            MonacoEditor
        },
        name: 'GitLab',
        data() {
            const validateName = (rule, value, callback) => {
                if (value === '') {
                    callback(new Error('名称不能为空'));
                } else {
                    // 进行中文的验证
                    if (/[\u4E00-\u9FA5]/i.test(value)) {
                        callback(new Error('名称不能为中文'));
                    }
                    let status = true;
                    this.steps.map(item => {
                        if (value === item.id) {
                            callback(new Error('同一个SLS文件,名称不能重复'));
                            status = false;
                        }
                    });
                    if (status) {
                        callback();
                    }
                }
            };
            return {
                productData: this.productList(),
                projectData: this.projectList(),
                productId: '',
                projectId: '',
                productStateProject: '',
                branchData: [],
                branchName: 'master',
                fileTreeData: [],
                fileTree: [],
                fileListPathData: [],
                targetData: [],
                hostData: [],
                deleteDisabled: true,
                editDisabled: true,
                createDisabled: false,
                inputDisabled: false,
                uploadDisabled: false,
                fileContent: '',
                path: '',
                filePath: [''],
                code: '',
                optionTypeName: '分发',
                options: {
                    selectOnLineNumbers: false,
                    // 启用该编辑器将安装一个时间间隔来检查其容器dom节点大小是否已更改,启用此功能可能会对性能造成严重影响
                    automaticLayout: true
                },
                highlightLines: [
                    {
                        number: 0,
                        class: 'primary-highlighted-line'
                    },
                    {
                        number: 0,
                        class: 'secondary-highlighted-line'
                    }
                ],
                formValidate: {
                    path: '',
                    fileDir: '',
                    code: ''
                },
                ruleValidate: {
                    path: [
                        {required: true, message: '请输选择要执行的SLS', trigger: 'blur'}
                    ],
                    fileDir: [
                        {required: true, message: '点击左侧树型结构获取目录，创建请输入文件名、上传请输入文件路径', trigger: 'blur'}
                    ]
                },
                formDistributeValidate: {
                    desc_path: '',
                    target: []
                },
                ruleDistributeValidate: {
                    desc_path: [
                        {required: true, message: '路径不能为空', trigger: 'blur'}
                    ],
                    target: [
                        {required: true, type: 'array', min: 1, message: '请选择目标', trigger: 'change'}
                    ]
                },
                formSynchronizeValidate: {
                    desc_path: '',
                    target: []
                },
                ruleSynchronizeValidate: {
                    desc_path: [
                        {required: true, message: '路径不能为空', trigger: 'blur'}
                    ],
                    target: [
                        {required: true, type: 'array', min: 1, message: '请选择目标', trigger: 'change'}
                    ]
                },
                steps: [],
                stepIndex: 0,
                title: '',
                tab: 'text',
                h: {
                    height: '620px'
                },
                fileManaged: [],
                cmdRun: [],
                fileDirectory: [],
                pkgInstalled: [],
                fileManagedFormView: false,
                cmdRunFormView: false,
                fileDirectoryFormView: false,
                pkgInstalledFormView: false,
                distributeView: false,
                synchronizeView: false
            };
        },
        props: {
            apiService: {
                type: String,
                required: true
            },
            productShow: {
                type: Boolean
            },
            projectShow: {
                type: Boolean
            },
            projectType: {
                type: String,
                required: true
            },
            slsURI: {
                type: String,
                required: true
            }
        },
        computed: {
            // 文件上传附带的额外参数
            uploadParameter: function () {
                let postData = {
                    'path': this.formValidate.fileDir,
                    'project_id': this.projectId,
                    'branch': this.branchName,
                    'action': 'upload'
                };
                return postData;
            },
            // 上传的地址
            action: function () {
                return this.Global.serverSrc + 'config/upload?product_id=' + this.productId;
            }
        },
        watch: {
            // 监控产品线变化
            projectId() {
                this.getGroups();
                this.getHosts();
                this.branch();
            },
            branchName() {
                if (this.branchName !== '') {
                    // 获取第一级GitLab数据
                    this.fileList();
                } else {
                    this.fileTreeData = [];
                    this.fileTree = [];
                    this.fileListPathData = [];
                    this.path = '';
                }
            },
            fileContent() {
                if (this.fileContent !== '') {
                    this.editDisabled = false;
                }
                // 重新加载 MonacoEditor
                this.reload();
            },
            filePath() {
                if (this.filePath.length !== 0) {
                    if (this.filePath[0].type !== 'tree') {
                        this.deleteDisabled = false;
                        this.editDisabled = false;
                        this.createDisabled = true;
                        this.inputDisabled = true;
                        this.uploadDisabled = true;
                    } else {
                        this.deleteDisabled = true;
                        this.editDisabled = true;
                        this.createDisabled = false;
                        this.inputDisabled = false;
                        this.uploadDisabled = false;
                        this.code = '';
                        this.fileContent = '';
                    }
                } else {
                    this.deleteDisabled = true;
                    this.editDisabled = true;
                    this.createDisabled = false;
                    this.inputDisabled = false;
                    this.uploadDisabled = false;
                }
            },
            tab() {
                if (this.tab === 'text') {
                    this.h.height = '620px';
                } else if (this.tab === 'sls') {
                    this.h.height = '620px';
                }
            }
        },
        methods: {
            productList() {
                this.axios.get(this.Global.serverSrc + 'product  ').then(
                    res => {
                        if (res.data['status'] === true) {
                            this.productData = res.data['data'];
                            if (this.productData.length > 0) {
                                this.productId = this.productData[0].id;
                                this.productStateProject = this.productData[0].state_project;
                            }
                        } else {
                            this.nError('Get Product Failure', res.data['message']);
                        }
                    },
                    err => {
                        let errInfo = '';
                        try {
                            errInfo = err.response.data['message'];
                        } catch (error) {
                            errInfo = err;
                        }
                        this.nError('Get Product Failure', errInfo);
                    });
            },
            projectList() {
                this.axios.get(this.Global.serverSrc + 'projects').then(
                    res => {
                        if (res.data['status'] === true) {
                            this.projectData = res.data['data'];
                            if (this.projectData.length > 0) {
                                this.projectId = this.projectData[0].id;
                            }
                        } else {
                            this.nError('Get Product Failure', res.data['message']);
                        }
                    },
                    err => {
                        let errInfo = '';
                        try {
                            errInfo = err.response.data['message'];
                        } catch (error) {
                            errInfo = err;
                        }
                        this.nError('Get Product Failure', errInfo);
                    });
            },
            branch() {
                this.branchData = [];
                this.branchName = '';
                this.fileTree = [];
                this.fileContent = '';
                this.axios.get(this.Global.serverSrc + this.apiService + '/branch?product_id=' + this.productId + '&project_id=' + this.projectId).then(
                    res => {
                        if (res.data['status'] === true) {
                            this.branchData = res.data['data'];
                            this.branchName = this.branchData[0];
                        } else {
                            this.nError('Get Branch Failure', res.data['message']);
                        }
                    },
                    err => {
                        let errInfo = '';
                        try {
                            errInfo = err.response.data['message'];
                        } catch (error) {
                            errInfo = err;
                        }
                        this.nError('Get Branch Failure', errInfo);
                    });
            },
            fileList() {
                this.fileContent = '';
                this.path = '';
                this.axios.get(this.Global.serverSrc + this.apiService + '/file?product_id=' + this.productId + '&project_id=' + this.projectId + '&path=/&branch=' + this.branchName).then(
                    res => {
                        if (res.data['status'] === true) {
                            this.fileTree = res.data['data'];
                        } else {
                            this.fileTree = [];
                            this.nError('Get File Tree Failure', res.data['message']);
                        }
                    },
                    err => {
                        let errInfo = '';
                        try {
                            errInfo = err.response.data['message'];
                        } catch (error) {
                            errInfo = err;
                        }
                        this.fileTree = [];
                        this.nError('Get File Tree Failure', errInfo);
                    });
            },
            // 传入path获取gitlab对应数据
            fileListPath(path,callback) {
                this.fileContent = '';
                this.axios.get(this.Global.serverSrc + this.apiService + '/file?product_id=' + this.productId + '&project_id=' + this.projectId + '&path=' + path + '&branch=' + this.branchName).then(
                    res => {
                        if (res.data['status'] === true) {
                            callback(res.data['data'])
                        } else {
                            this.fileListPathData = [];
                            this.nError('Get File Tree Failure', res.data['message']);
                            callback(this.fileListPathData)
                        }
                    },
                    err => {
                        let errInfo = '';
                        try {
                            errInfo = err.response.data['message'];
                        } catch (error) {
                            errInfo = err;
                        }
                        this.fileListPathData = [];
                        callback(this.fileListPathData)
                        this.nError('Get File Tree Failure', errInfo);
                    });
            },
            handleContent(filePath) {
                this.filePath = filePath;
                if (filePath.length !== 0) {
                    this.formValidate.fileDir = filePath[0].path;
                }
                if (filePath.length !== 0 && filePath[0]['type'] !== 'tree') {
                    // 如果点击的是文件切回Tab 为 text
                    this.tab = 'text';
                    this.fileContent = '';
                    this.path = filePath[0]['path'];
                    this.axios.get(this.Global.serverSrc + this.apiService + '/content?product_id=' + this.productId + '&project_id=' + this.projectId + '&branch=' + this.branchName + '&path=' + this.path).then(
                        res => {
                            if (res.data['status'] === true) {
                                this.fileContent = res.data['data'];
                            } else {
                                this.nError('Get File Content Failure', res.data['message']);
                            }
                        },
                        err => {
                            let errInfo = '';
                            try {
                                errInfo = err.response.data['message'];
                            } catch (error) {
                                errInfo = err;
                            }
                            this.nError('Get File Content Failure', errInfo);
                        });
                }
            },
            // 展开树型结构获取gitlab数据
            loadData(item, callback) {
                this.fileListPath(item['path'],callback);
                // fileListPath为异步方法,等待300ms
            },
            // 重新定义错误消息
            nError(title, info) {
                this.$Notice.error({
                    title: title,
                    desc: info,
                    duration: 10
                });
            },
            refresh() {
                this.projectList();
                this.path = '';
                this.filePath = [''];
                this.branch();
                this.fileList();
            },
            handleEdit() {
                let postData = {
                    'path': this.path,
                    'project_id': this.projectId,
                    'branch': this.branchName,
                    'action': 'update',
                    'content': this.code
                };
                this.axios.post(this.Global.serverSrc + 'config/commit?product_id=' + this.productId, postData).then(
                    res => {
                        if (res.data['status'] === true) {
                            this.result = res.data['data'];
                            this.edit = false;
                            this.$Message.success('更新成功！');
                            // 调用hook进行更新
                            this.handleHook();
                            // this.fileList();
                        } else {
                            this.nError('Update Failure', res.data['message']);
                        }
                    },
                    err => {
                        let errInfo = '';
                        try {
                            errInfo = err.response.data['message'];
                        } catch (error) {
                            errInfo = err;
                        }
                        this.nError('Update Failure', errInfo);
                    });
            },

            handleDistributeView() {
                this.distributeView = true;
            },
            handleDistribute() {
                this.$Message.success('分发中，稍等~');
                let postData = {
                    'file_path': this.formValidate.fileDir,
                    'desc_path': this.formDistributeValidate.desc_path,
                    'target': this.formDistributeValidate.target,
                    'product_id': this.productId,
                    'project_id': this.projectId
                };
                this.axios.post(this.Global.serverSrc + 'config/distribute', postData).then(
                    res => {
                        if (res.data['status'] === true) {
                            this.result = res.data['data'];
                            this.edit = false;
                            this.$Message.success('分发完成');
                            // 调用hook进行更新
                            this.handleHook();
                            // this.fileList();
                        } else {
                            this.nError('Update Failure', res.data['message']);
                        }
                    },
                    err => {
                        let errInfo = '';
                        try {
                            errInfo = err.response.data['message'];
                        } catch (error) {
                            errInfo = err;
                        }
                        this.nError('Update Failure', errInfo);
                    });
            },
            handleSynView() {
                this.$Message.success("???");
                this.synchronizeView = true;

            },
            handleSynchronize() {
                this.axios.post(this.Global.serverSrc + 'config/synchronize', this.formSynchronizeValidate).then(
                    res => {
                        if (res.data['status'] === true) {
                            this.$Message.success('备份成功！');
                        } else {
                            this.nError('Update Failure', res.data['message']);
                        }
                    },
                    err => {
                        let errInfo = '';
                        try {
                            errInfo = err.response.data['message'];
                        } catch (error) {
                            errInfo = err;
                        }
                        this.nError('Update Failure', errInfo);
                    });
            },
            // 删除提示
            PopperShow() {
                this.title = '你确定删除 ' + this.formValidate.fileDir + ' 这个文件吗?';
            },
            handleDelete() {
                let postData = {
                    'path': this.formValidate.fileDir,
                    'project_id': this.projectId,
                    'branch': this.branchName,
                    'action': 'delete'
                };
                this.axios.post(this.Global.serverSrc + 'config/commit?product_id=' + this.productId, postData).then(
                    res => {
                        if (res.data['status'] === true) {
                            this.result = res.data['data'];
                            this.edit = false;
                            this.$Message.success('删除成功！');
                            // 刷新gitlab file list
                            this.fileList();
                            this.filePath = [];
                            this.formValidate.fileDir = '';
                            this.fileContent = '';
                        } else {
                            this.nError('Delete Failure', res.data['message']);
                        }
                    },
                    err => {
                        let errInfo = '';
                        try {
                            errInfo = err.response.data['message'];
                        } catch (error) {
                            errInfo = err;
                        }
                        this.nError('Delete Failure', errInfo);
                    });
            },
            handleCreate(name) {
                this.$refs[name].validate((valid) => {
                    if (valid) {
                        let postData = {
                            'path': this.formValidate.fileDir,
                            'project_id': this.projectId,
                            'branch': this.branchName,
                            'action': 'create',
                            'content': this.code
                        };
                        this.axios.post(this.Global.serverSrc + 'config/commit?product_id=' + this.productId, postData).then(
                            res => {
                                if (res.data['status'] === true) {
                                    this.result = res.data['data'];
                                    this.edit = false;
                                    this.$Message.success('创建成功！');
                                    // 刷新gitlab file list
                                    this.fileList();
                                    this.filePath = [];
                                    this.formValidate.fileDir = '';
                                    this.reload();
                                } else {
                                    this.nError('Create Failure', res.data['message']);
                                }
                            },
                            err => {
                                let errInfo = '';
                                try {
                                    errInfo = err.response.data['message'];
                                } catch (error) {
                                    errInfo = err;
                                }
                                this.nError('Create Failure', errInfo);
                            });
                    } else {
                        this.$Message.error('请检查表单数据！');
                    }
                });
            },
            handleHook() {
                let postData = {
                    'tag': 'gitfs/update'
                };
                this.axios.post(this.Global.serverSrc + 'hook?product_id=' + this.productId, postData).then(
                    res => {
                        if (res.data['status'] === true) {
                            this.result = res.data['data'];
                        } else {
                            this.nError('Web Hook Failure', res.data['message']);
                        }
                    },
                    err => {
                        let errInfo = '';
                        try {
                            errInfo = err.response.data['message'];
                        } catch (error) {
                            errInfo = err;
                        }
                        this.nError('Web Hook Failure', errInfo);
                    });
            },
            getGroups() {
                this.axios.get(this.Global.serverSrc + 'config/group').then(
                    res => {
                        if (res.data['status'] === true) {
                            this.targetData = res.data['data'];
                        } else {
                            this.nError('Get Info Failure', res.data['message']);
                        }
                        ;
                        this.loading = false;
                    },
                    err => {
                        let errInfo = '';
                        try {
                            errInfo = err.response.data['message'];
                            if (err.response.status === 404) {
                                this.targetData = [];
                            } else {
                                this.nError('Get Info Failure', errInfo);
                            }
                        } catch (error) {
                            errInfo = err;
                            this.nError('Get Info Failure', errInfo);
                        }
                        this.loading = false;
                    });
            },
            getHosts() {
                this.axios.get(this.Global.serverSrc + 'config/host').then(
                    res => {
                        if (res.data['status'] === true) {
                            this.hostData = res.data['data'];
                        } else {
                            this.nError('Get Info Failure', res.data['message']);
                        }
                        ;
                        this.loading = false;
                    },
                    err => {
                        let errInfo = '';
                        try {
                            errInfo = err.response.data['message'];
                            if (err.response.status === 404) {
                                this.hostData = [];
                            } else {
                                this.nError('Get Info Failure', errInfo);
                            }
                        } catch (error) {
                            errInfo = err;
                            this.nError('Get Info Failure', errInfo);
                        }
                        this.loading = false;
                    });
            },

            onMounted(editor) {
                this.editor = editor;
            },
            onCodeChange(editor) {
                this.code = this.editor.getValue();
            },
            // 重载编辑框
            reload() {
                clearTimeout(time);
                let time = setTimeout(() => {
                    this.$refs.vscode.destroyMonaco();
                    this.$refs.vscode.createMonaco();
                }, 1);
            },
            // 上传成功
            UploadSuccess() {
                this.$Message.success('上传成功！');
                this.fileList();
            },
            // 上传失败
            UploadError() {
                this.nError('Upload Failure', 'The file path is incorrect or file formats are not supported');
            },
            // 上传前检查表单
            beforeUpdate() {
                let form = false;
                this.$refs['formValidate'].validate((valid) => {
                    if (valid) {
                        form = true;
                    } else {
                        form = false;
                    }
                });
                return form;
            },
            // 表单重置

        }
    };
</script>
<style scoped>
    .hr-margin {
        margin-top: 10px;
        margin-bottom: 10px;
    }
</style>