<template>
    <div>
        <common-table
                :cColumns="cColumns"
                :apiService="apiService"
                @getHostEvent="getHostEvent"
                :hostShow="true"
                ref="childrenMethods">

            <Modal slot="option" v-model="singlePingView" :title="singlePingName">
                <Form :label-width="125">
                    <FormItem label="">
                        <Spin size="large" fix v-if="spinShow"></Spin>
                        <Alert :type="summaryType">
                            <ul>
                                <li>
                                    状态： {{result.status}}
                                </li>
                            </ul>
                            <ul>
                                <li>
                                    设备信息： {{result.sysDescr}}
                                </li>
                            </ul>
                        </Alert>
                    </FormItem>
                </Form>
                <div slot="footer">
                    <Button type="primary" @click="singlePing()">测试</Button>
                </div>
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
                /*formView: false,*/
                result: '',
                resultShow: true,
                singlePingView: false,
                salt_api_loading: false,
                gitlab_api_loading: false,
                id: '',
                singlePingName: '单设备测通',
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
                        width: 170,
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
                                            this.$parent.formView = true;
                                            this.$parent.optionType = 'edit';
                                            this.$parent.optionTypeName = '编辑';
                                            this.id = params.row.id;
                                            this.$parent.formValidate = params.row;
                                        }
                                    }
                                }, '编辑'),
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
                                            this.id = params.row.id;
                                            this.result = '';
                                            this.singlePingView = true;
                                        }
                                    }
                                }, '测通'),
                                h('Poptip', {
                                    props: {
                                        confirm: true,
                                        title: '确定要删除嘛 ' + params.row.target + ' 吗?',
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
                                ]),
                            ]);
                        }
                    }
                ],
            };
        },

        methods: {
            getHostEvent: function (hostData, hostId) {
                this.hostData = hostData;
                this.hostId = hostId;
            },
            // 调用子组件进行删除
            del() {
                this.$refs.childrenMethods.del(this.delId);
                this.$refs.childrenMethods.refresh();
            },

            // 调用子组件进行数据刷新
            tableList() {
                this.$refs.childrenMethods.tableList();
            },

            // 调用子组件消息通知
            nError(title, info) {
                this.$refs.childrenMethods.nError(title, info);
            },
            singlePing(name) {
                this.$Message.success('稍等~');
                let postData = {
                    'host_id': this.hostId,
                    'target_id': this.id
                };
                this.axios.post(this.Global.serverSrc + this.apiService + '/single', postData).then(
                    res => {
                        if (res.data['status'] === true) {
                            this.result = res.data['data'];
                            this.$Message.success(this.result);
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
            },

        }
    }
    ;
</script>