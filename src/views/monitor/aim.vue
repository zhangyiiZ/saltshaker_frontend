<template>
    <div>
        <common-table
                :cColumns="cColumns"
                :apiService="apiService"
                @getProductEvent="getProductEvent"
                :productShow="false"
                ref="childrenMethods">
            <Button slot="create" type="primary" @click="add('formValidate')">导入监控信息</Button>
            <Button slot="create" type="primary" @click="add('formValidate')">一键测通</Button>
            <Modal slot="option" v-model="formView"  :title="optionTypeName">
                <Form ref="formValidate" :model="formValidate" :rules="ruleValidate" :label-width="125">
                    <FormItem label="项目名" prop="name">
                        <Input v-model="formValidate.name" placeholder="输入用户名"></Input>
                    </FormItem>
                    <FormItem label="描述" prop="description">
                        <Input v-model="formValidate.description" placeholder="输入描述"></Input>
                    </FormItem>
                    <FormItem label="Master ID" prop="salt_master_id">
                        <Input v-model="formValidate.salt_master_id" placeholder="输入Master ID"></Input>
                    </FormItem>
                    <FormItem label="Master API 地址" prop="salt_master_url">
                        <Input v-model="formValidate.salt_master_url" placeholder="输入Master API 地址"></Input>
                    </FormItem>
                    <FormItem label="Master API 用户名" prop="salt_master_user">
                        <Input v-model="formValidate.salt_master_user" placeholder="输入Master API 用户名"></Input>
                    </FormItem>
                    <FormItem label="Master API 密码" prop="salt_master_password">
                        <Input v-model="formValidate.salt_master_password" placeholder="输入Master API 密码"></Input>
                    </FormItem>
                    <FormItem label="" prop="check_salt_api">
                        <Button type="primary" :loading="salt_api_loading" @click="handleCheckAPI('salt_api')">
                            <span v-if="!salt_api_loading">测试 Salt API</span>
                            <span v-else>测试 Salt API</span>
                        </Button>
                    </FormItem>
                    <FormItem label="文件服务器">
                        <RadioGroup v-model="formValidate.file_server">
                            <Radio label="gitfs">GitLab</Radio>
                            <Radio label="rsync">Rsync</Radio>
                        </RadioGroup>
                    </FormItem>
                    <FormItem v-if="this.formValidate.file_server === 'gitfs'" label="GitLab 地址" prop="gitlab_url">
                        <Input v-model="formValidate.gitlab_url" placeholder="输入GitLab 地址"></Input>
                    </FormItem>
                    <FormItem v-if="this.formValidate.file_server === 'gitfs'" label="GitLab API 版本" prop="api_version">
                        <Input v-model="formValidate.api_version" placeholder="输入GitLab API 版本"></Input>
                    </FormItem>
                    <FormItem v-if="this.formValidate.file_server === 'gitfs'" label="GitLab Token" prop="private_token">
                        <Input v-model="formValidate.private_token" placeholder="输入GitLab Token"></Input>
                    </FormItem>
                    <FormItem v-if="this.formValidate.file_server === 'gitfs'" label="GitLab State 项目" prop="state_project">
                        <Input v-model="formValidate.state_project" placeholder="输入GitLab State 项目"></Input>
                    </FormItem>
                    <FormItem v-if="this.formValidate.file_server === 'gitfs'" label="GitLab Pillar 项目" prop="pillar_project">
                        <Input v-model="formValidate.pillar_project" placeholder="输入GitLab Pillar 项目"></Input>
                    </FormItem>
                    <FormItem v-if="this.formValidate.file_server === 'gitfs'" label="" prop="check_gitlab_api">
                        <Button type="primary" :loading="gitlab_api_loading" @click="handleCheckAPI('gitlab_api')">
                            <span v-if="!gitlab_api_loading">测试 GitLab API</span>
                            <span v-else>测试 GitLab API</span>
                        </Button>
                    </FormItem>
                </Form>
                <div slot="footer">
                    <Button type="ghost" @click="handleReset('formValidate')" style="margin-left: 8px">重置</Button>
                    <Button type="primary" @click="handleSubmit('formValidate')">提交</Button>
                </div>
            </Modal>
        </common-table>
    </div>
</template>

<script>
    import CommonTable from '../common-components/table/table.vue';
    export default {
        components: {
            CommonTable
        },
        data () {
            return {
                apiService: 'product',
                productData: [],
                productId: '',
                // 删除数据
                delId: '',
                delIndex: '',
                // 编辑数据
                formView: false,
                salt_api_loading: false,
                gitlab_api_loading: false,
                id: '',
                optionType: '',
                optionTypeName: '',
                cColumns: [
                    {
                        title: 'targets',
                        key: 'targets',
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
                                }, params.row.name)
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
                    name: '',
                    description: '',
                    salt_master_id: '',
                    salt_master_url: '',
                    salt_master_user: '',
                    salt_master_password: '',
                    file_server: 'gitfs',
                    gitlab_url: '',
                    api_version: '',
                    private_token: '',
                    state_project: '',
                    pillar_project: '',
                    oauth_token: '',
                    http_password: '',
                    http_username: '',
                    password: '',
                    email: ''
                },
                ruleValidate: {
                    name: [
                        { required: true, message: '项目名不能为空', trigger: 'blur' }
                    ],
                    description: [
                        { required: true, message: '描述不能为空', trigger: 'blur' }
                    ],
                    salt_master_id: [
                        { required: true, message: 'Master ID不能为空', trigger: 'blur' }
                    ],
                    salt_master_url: [
                        { required: true, message: 'Master API 地址不能为空', trigger: 'blur' }
                    ],
                    salt_master_user: [
                        { required: true, message: 'Master API 用户名不能为空', trigger: 'blur' }
                    ],
                    salt_master_password: [
                        { required: true, message: 'Master API 密码不能为空', trigger: 'blur' }
                    ],
                    gitlab_url: [
                        { required: true, message: 'GitLab 地址不能为空', trigger: 'blur' }
                    ],
                    api_version: [
                        { required: true, message: 'GitLab API 版本不能为空', trigger: 'blur' }
                    ],
                    private_token: [
                        { required: true, message: 'GitLab Token不能为空', trigger: 'blur' }
                    ],
                    state_project: [
                        { required: true, message: 'GitLab State 项目不能为空', trigger: 'blur' }
                    ],
                    pillar_project: [
                        { required: true, message: 'GitLab Pillar 项目不能为空', trigger: 'blur' }
                    ]
                }
            };
        },
        methods: {
            getProductEvent: function (productData, productId) {
                this.productData = productData;
                this.productId = productId;
            },
            // 调用子组件进行删除
            del () {
                this.$refs.childrenMethods.del(this.delId);
            },
            // 调用子组件进行数据刷新
            tableList () {
                this.$refs.childrenMethods.tableList();
            },
            // 调用子组件消息通知
            nError (title, info) {
                this.$refs.childrenMethods.nError(title, info);
            },
            // 添加展示
            add (name) {
                this.handleReset(name);
                this.optionType = 'add';
                this.optionTypeName = '添加';
                this.formView = true;
            },
            // 检查salt api 可用性
            handleCheckAPI (name) {
                if (name === 'salt_api') {
                    this.salt_api_loading = true;
                } else {
                    this.gitlab_api_loading = true;
                }
                this.axios.post(this.Global.serverSrc + this.apiService + '/check/' + name,
                    this.formValidate).then(
                    res => {
                        if (res.data['status'] === true) {
                            this.formView = true;
                            this.$Message.success('成功！');
                        } else {
                            this.nError('Check Failure', res.data['message']);
                        }
                        if (name === 'salt_api') {
                            this.salt_api_loading = false;
                        } else {
                            this.gitlab_api_loading = false;
                        }
                    },
                    err => {
                        this.formView = true;
                        let errInfo = '';
                        try {
                            errInfo = err.response.data['message'];
                        } catch (error) {
                            errInfo = err;
                        }
                        this.nError('Check Failure', errInfo);
                        if (name === 'salt_api') {
                            this.salt_api_loading = false;
                        } else {
                            this.gitlab_api_loading = false;
                        }
                    });
            },
            // 表单提
            handleSubmit (name) {
                this.$refs[name].validate((valid) => {
                    if (valid) {
                        if (this.formValidate.file_server === 'rsync') {
                            this.formValidate.gitlab_url = '';
                            this.formValidate.api_version = '';
                            this.formValidate.private_token = '';
                            this.formValidate.state_project = '';
                            this.formValidate.pillar_project = '';
                        };
                        // 编辑
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
            },
            handleReset (name) {
                this.$refs[name].resetFields();
            }
        }
    };
</script>