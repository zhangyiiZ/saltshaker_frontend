<template>
    <div>
        <common-table
                :cColumns="cColumns"
                :apiService="apiService"
                @getHostEvent="getHostEvent"
                :hostShow="true"
                ref="childrenMethods">
            <Button slot="create" type="primary" @click="add('formValidate')">手动导入</Button>
            <Button slot="create" type="primary" @click="batchImport('formValidate')">批量导入</Button>
            <Button slot="create" type="primary" @click="configGenerate('formValidate')">配置生成</Button>
            <Button slot="create" type="primary" @click="pingAll()">一键测通</Button>
            <Modal slot="option" v-model="formView" :title="optionTypeName">
                <Form ref="formValidate" :model="formValidate" :rules="ruleValidate" :label-width="125">
                    <FormItem label="target" prop="target">
                        <Input v-model="formValidate.target" placeholder="输入 target"></Input>
                    </FormItem>
                    <FormItem label="IP" prop="IP">
                        <Input v-model="formValidate.IP" placeholder="输入 IP"></Input>
                    </FormItem>
                    <FormItem label="location" prop="location">
                        <Input v-model="formValidate.location" placeholder="输入 location"></Input>
                    </FormItem>
                    <FormItem label="model" prop="model">
                        <Input v-model="formValidate.model" placeholder="输入 model"></Input>
                    </FormItem>
                    <FormItem label="type" prop="type">
                        <Input v-model="formValidate.type" placeholder="输入 type"></Input>
                    </FormItem>
                    <FormItem label="project" prop="project">
                        <Input v-model="formValidate.project" placeholder="输入 project"></Input>
                    </FormItem>
                    <FormItem label="client" prop="client">
                        <Input v-model="formValidate.client" placeholder="输入 client"></Input>
                    </FormItem>
                    <FormItem label="pool" prop="pool">
                        <Input v-model="formValidate.pool" placeholder="输入 pool"></Input>
                    </FormItem>
                </Form>
                <div slot="footer">
                    <Button type="ghost" @click="handleReset('formValidate')" style="margin-left: 8px">重置</Button>
                    <Button type="primary" @click="handleSubmit('formValidate')">提交</Button>
                </div>
            </Modal>
            <Modal slot="option" v-model="configGenerateView" :title="configGenerateName">
                <Form ref="formValidate" :model="formValidate" :rules="ruleValidate" :label-width="125">
                    <FormItem label="型号关键词" prop="key_word">
                        <Input v-model="formValidate.key_word" placeholder="和搜索预览词一致,为空则全选"></Input>
                    </FormItem>
                    <FormItem label="生成地址" prop="path">
                        <Input v-model="formValidate.path" placeholder="生成地址，不填则为默认地址 /usr/local/prometheus/conf.d/ "></Input>
                    </FormItem>
                    <FormItem label="文件名" prop="file_name">
                        <Input v-model="formValidate.file_name" placeholder="文件名,如 snmpconf_h3cS6900.json"></Input>
                    </FormItem>
                </Form>
                <div slot="footer">
                    <Button type="primary" @click="handleGenerate('formValidate')">提交</Button>
                </div>
            </Modal>
            <Modal slot="option" v-model="batchImportView" :title="batchImportName">
                <Card dis-hover>
                    <Form ref="formValidate" :model="formValidate" :rules="ruleValidate" :label-width="46">
                        <FormItem label="" prop="code">
                            <Tabs v-model="tab" :style="[h]">
                                <TabPane label="目标设备execl文件" name="upload" :disabled="uploadDisabled">
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
                                                <Icon type="ios-cloud-upload" size="52" style="color: #3399ff"></Icon>
                                                <p>点击或者拖拽上传</p>
                                            </div>
                                        </Upload>
                                    </div>
                                </TabPane>
                            </Tabs>
                        </FormItem>
                    </Form>
                </Card>
            </Modal>
        </common-table>
    </div>
</template>

<script>
    import CommonTable from '../common-components/table/table_host.vue';

    export default {
        components: {
            CommonTable
        },
        data() {
            return {
                apiService: 'target',
                hostData: [],
                hostId: '',
                // 删除数据
                delId: '',
                delIndex: '',
                // 编辑数据
                formView: false,
                batchImportView: false,
                configGenerateView: false,
                salt_api_loading: false,
                gitlab_api_loading: false,
                id: '',
                optionType: '',
                optionTypeName: '',
                batchImportName: '批量导入',
                configGenerateName: '配置生成',
                cColumns: [
                    {
                        title: 'target',
                        key: 'target',
                        sortable: true,
                        width: 160,
                        fixed: 'left',
                        render: (h, params) => {
                            return h('div', [
                                h('Tooltip', {
                                    props: {
                                        content: params.row.id,
                                        transfer: true,
                                        placement: 'top-start'
                                    }
                                }, params.row.target)
                            ]);
                        }
                    },
                    {
                        title: 'IP',
                        key: 'IP',
                        width: 160,
                        sortable: true
                    },
                    {
                        title: 'location',
                        key: 'location',
                        width: 160,
                        sortable: true
                    },
                    {
                        title: 'model',
                        key: 'model',
                        width: 180,
                        sortable: true
                    },
                    {
                        title: 'type',
                        key: 'type',
                        width: 160,
                        sortable: true
                    },
                    {
                        title: 'project',
                        key: 'project',
                        width: 140,
                        sortable: true
                    },
                    {
                        title: 'client',
                        key: 'client',
                        width: 180,
                        sortable: true
                    },
                    {
                        title: 'pool',
                        key: 'pool',
                        width: 160,
                        sortable: true
                    },
                    {
                        title: '操作',
                        key: 'action',
                        width: 123,
                        fixed: 'right',
                        align: 'center',
                        render: (h, params) => {
                            return h('div', [
                                h('Button', {
                                    props: {
                                        type: 'primary',
                                        size: 'small'
                                    },
                                    style: {
                                        marginRight: '5px'
                                    },
                                    on: {
                                        click: () => {
                                            this.formView = true;
                                            this.optionType = 'edit';
                                            this.optionTypeName = '编辑';
                                            this.id = params.row.id;
                                            this.formValidate = params.row;
                                        }
                                    }
                                }, '编辑'),
                                h('Poptip', {
                                    props: {
                                        confirm: true,
                                        title: '确定要删除 ' + params.row.name + ' 吗?',
                                        transfer: true,
                                        placement: 'top-end'
                                    },
                                    on: {
                                        'on-ok': () => {
                                            this.delId = params.row.id;
                                            this.delIndex = params.index;
                                            this.del();
                                        }
                                    }
                                }, [
                                    h('Button', {
                                        props: {
                                            type: 'error',
                                            size: 'small'
                                        }
                                    }, '删除')
                                ])
                            ]);
                        }
                    }
                ],
                // 表单验证
                formValidate: {
                    host_id: '',
                    target: '',
                    IP: '',
                    location: '',
                    model: '',
                    type: '',
                    project: '',
                    client: '',
                    pool: '',
                    path: '',
                    file_name:'',
                    key_word: ''
                },
                ruleValidate: {
                    target: [
                        {required: true, message: '项目名不能为空', trigger: 'blur'}
                    ],
                    IP: [
                        {required: true, message: '描述不能为空', trigger: 'blur'}
                    ],
                    model: [
                        {required: true, message: 'Master ID不能为空', trigger: 'blur'}
                    ],
                    type: [
                        {required: true, message: 'Master API 地址不能为空', trigger: 'blur'}
                    ],
                    project: [
                        {required: true, message: 'Master API 用户名不能为空', trigger: 'blur'}
                    ],
                    client: [
                        {required: true, message: 'Master API 密码不能为空', trigger: 'blur'}
                    ],
                    pool: [
                        {required: true, message: 'GitLab 地址不能为空', trigger: 'blur'}
                    ]
                }
            };
        },
        computed: {
            // 文件上传附带的额外参数
            uploadParameter: function () {
                let postData = {
                    'host_id': this.hostId,
                    'action': 'upload'
                };
                return postData;
            },
            // 上传的地址
            action: function () {
                return this.Global.serverSrc + 'target/upload';
            }
        },
        methods: {
            getHostEvent: function (hostData, hostId) {
                this.hostData = hostData;
                this.hostId = hostId;
            },
            // 调用子组件进行删除
            del() {
                this.$refs.childrenMethods.del(this.delId);
            },
            // 调用子组件进行数据刷新
            tableList() {
                this.$refs.childrenMethods.tableList();
            },
            // 调用子组件消息通知
            nError(title, info) {
                this.$refs.childrenMethods.nError(title, info);
            },
            // 添加展示
            add(name) {
                this.handleReset(name);
                this.optionType = 'add';
                this.optionTypeName = '添加';
                this.formView = true;
            },
            // 添加展示
            batchImport(name) {
                this.batchImportView = true;
            },
            configGenerate(name) {
                this.configGenerateView = true;
            },
            pingAll(name){
                this.$Message.success('ping！');
            },
            // 表单提
            handleSubmit(name) {
                this.$refs[name].validate((valid) => {
                    if (valid) {
                        // 编辑
                        this.formValidate.host_id = this.hostId;
                        if (this.optionType === 'edit') {
                            this.axios.put(this.Global.serverSrc + this.apiService + '/' + this.id,
                                this.formValidate).then(
                                res => {
                                    if (res.data['status'] === true) {
                                        this.formView = false;
                                        this.$Message.success('成功！');
                                        this.tableList();
                                    } else {
                                        this.nError('Edit Failure', res.data['message']);
                                    }
                                },
                                err => {
                                    let errInfo = '';
                                    try {
                                        errInfo = err.response.data['message'];
                                    } catch (error) {
                                        errInfo = err;
                                    }
                                    this.nError('Edit Failure', errInfo);
                                });
                        } else {
                            // 添加
                            this.axios.post(this.Global.serverSrc + this.apiService,
                                this.formValidate).then(
                                res => {
                                    if (res.data['status'] === true) {
                                        this.formView = false;
                                        this.$Message.success('成功！');
                                        this.tableList();
                                    } else {
                                        this.nError('Add Failure', res.data['message']);
                                    }
                                },
                                err => {
                                    let errInfo = '';
                                    try {
                                        errInfo = err.response.data['message'];
                                    } catch (error) {
                                        errInfo = err;
                                    }
                                    this.nError('Add Failure', errInfo);
                                });
                        }
                    } else {
                        this.$Message.error('请检查表单数据！');
                    }
                });
            }
            ,
            handleReset(name) {
                this.$refs[name].resetFields();
            }
            ,
            handleGenerate(name) {
                this.$refs[name].validate((valid) => {
                    if (valid) {
                        // 编辑
                        let postData = {
                            'host_id': this.hostId,
                            'key_word': this.formValidate.key_word,
                            'path': this.formValidate.path,
                            'file_name':this.formValidate.file_name,
                            'action': 'configGenerate'
                        };
                        this.axios.post(this.Global.serverSrc + this.apiService + '/config', postData).then(
                            res => {
                                if (res.data['status'] === true) {
                                    this.configGenerateView = false;
                                    this.$Message.success('配置生成成功！');
                                } else {
                                    this.nError('生成失败！', res.data['message']);
                                }
                            },
                            err => {
                                let errInfo = '';
                                try {
                                    errInfo = err.response.data['message'];
                                } catch (error) {
                                    errInfo = err;
                                }
                                this.nError('Generate Failure', errInfo);
                            });
                    }
                });
            }
            ,
            UploadSuccess() {
                this.$Message.success('上传成功！请刷新');
            }
            ,
            // 上传失败
            UploadError() {
                this.nError('Upload Failure', 'The file path is incorrect or file formats are not supported');
            }
            ,
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
            }
        }
    }
    ;
</script>