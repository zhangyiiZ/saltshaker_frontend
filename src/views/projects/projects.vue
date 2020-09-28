<template>
    <div>
        <common-table :cColumns="cColumns" :apiService="apiService" @getProductEvent="getProductEvent" :productShow="true" ref="childrenMethods">
            <Button slot="create" type="primary" @click="add('formValidate')">创建项目</Button>
            <Modal slot="option" v-model="formView"  :title="optionTypeName" width="650px">
                <Form ref="formValidate" :model="formValidate" :rules="ruleValidate" :label-width="60">
                    <FormItem label="项目名" prop="name">
                        <Input v-model="formValidate.name" placeholder="输入用户名"></Input>
                    </FormItem>
                    <FormItem label="描述" prop="description">
                        <Input v-model="formValidate.description" placeholder="输入描述"></Input>
                    </FormItem>
                    <FormItem label="分组" prop="group">
                        <Transfer
                            :data="originMinion"
                            :target-keys="targetMinion"
                            :list-style="listStyle"
                            :render-format="renders"
                            :titles = "titles"
                            filterable
                            @on-change="handleChange">
                        </Transfer>
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
                apiService: 'projects',
                productData: [],
                productId: '',
                groupsData: [],
                groupsId: '',
                projectsId:'',
                // 删除数据
                delId: '',
                delIndex: '',
                // 编辑数据
                formView: false,
                optionType: '',
                optionTypeName: '',
                cColumns: [
                    {
                        title: '项目名',
                        key: 'name',
                        sortable: true,
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
                        title: '描述',
                        key: 'description',
                        sortable: true
                    },
                    {
                        title: '分组',
                        key: 'group',
                        sortable: true,
                        render: (h, params) => {
                            return h('ul', params.row.group.map(item => {
                                return h('li', {
                                    style: {
                                        textAlign: 'left',
                                        padding: '0px'
                                    }
                                }, item);
                            })
                            );
                        }
                    },
                    {
                        title: '操作',
                        key: 'action',
                        width: 123,
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
                                            this.handleReset('formValidate');
                                            this.optionType = 'edit';
                                            this.optionTypeName = '编辑';
                                            // this.targetMinion = [];
                                            this.projectsId = params.row.id;
                                            this.getTargetMinion();
                                            this.formValidate.name = params.row.name;
                                            this.formValidate.description = params.row.description;
                                            this.formValidate.productId = params.row.product_id;
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
                    productId: ''
                },
                ruleValidate: {
                    name: [
                        { required: true, message: '分组名不能为空', trigger: 'blur' }
                    ],
                    description: [
                        { required: true, message: '描述不能为空', trigger: 'blur' }
                    ]
                },
                // 穿梭框
                originMinion: [],
                targetMinion: [],
                titles: ['待加分组', '当前项目'],
                listStyle: {
                    width: '250px',
                    height: '230px'
                }
            };
        },
        watch: {
            productId () {
                this.originMinion = [];
                this.getOriginMinion();
            }
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
                this.originMinion = [];
                this.targetMinion = [];
                this.getOriginMinion();
            },
            // 表单提
            handleSubmit (name) {
                this.$refs[name].validate((valid) => {
                    if (valid) {
                        // 编辑
                        let postData = {
                            'name': this.formValidate.name,
                            'description': this.formValidate.description,
                            'group': this.targetMinion,
                            'product_id': this.productId
                        };
                        if (this.optionType === 'edit') {
                            this.axios.put(this.Global.serverSrc + this.apiService + '/' + this.projectsId,
                                postData).then(
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
                                postData).then(
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
            // 表单重置
            handleReset (name) {
                this.$refs[name].resetFields();
            },
            // 穿梭框
            getOriginMinion () {
                this.axios.get(this.Global.serverSrc + 'projects?product_id=' + this.productId).then(
                    res => {
                        if (res.data['status'] === true) {
                            for (var i = 0; i < res.data['data'].length; i++) {
                                this.originMinion.push({
                                    key: res.data['data'][i]['name']
                                });
                            }
                        } else {
                            this.nError('Get Info Failure', res.data['message']);
                        };
                    },
                    err => {
                        let errInfo = '';
                        try {
                            errInfo = err.response.data['message'];
                            if (err.response.status === 404) {
                                this.originMinion = [];
                            } else {
                                this.nError('Get Info Failure', errInfo);
                            }
                        } catch (error) {
                            errInfo = err;
                            this.nError('Get Info Failure', errInfo);
                        }
                    });
            },
            groupList () {
                this.axios.get(this.Global.serverSrc + this.apiService + '?product_id=' + this.productId).then(
                    res => {
                        if (res.data['status'] === true) {
                            this.groupsData = res.data['data'];
                        } else {
                            this.nError('Get Groups Failure', res.data['message']);
                        }
                    },
                    err => {
                        let errInfo = '';
                        try {
                            errInfo = err.response.data['message'];
                        } catch (error) {
                            errInfo = err;
                        }
                        this.nError('Get Groups Failure', errInfo);
                    });
            },
            getTargetMinion () {
                this.axios.get(this.Global.serverSrc + 'projects/' + this.projectsId).then(
                    res => {
                        if (res.data['status'] === true) {
                            this.targetMinion = res.data['data']['minion'];
                        } else {
                            this.nError('Get Info Failure', res.data['message']);
                        };
                    },
                    err => {
                        let errInfo = '';
                        try {
                            errInfo = err.response.data['message'];
                            if (err.response.status === 404) {
                                this.targetMinion = [];
                            } else {
                                this.nError('Get Info Failure', errInfo);
                            }
                        } catch (error) {
                            errInfo = err;
                            this.nError('Get Info Failure', errInfo);
                        }
                    });
            },
            handleChange (newTargetMinion) {
                this.targetMinion = newTargetMinion;
            },
            renders (item) {
                return item.key;
            }
        }
    };
</script>