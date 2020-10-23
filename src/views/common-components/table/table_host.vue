<style lang="less">
    @import '../../../styles/common.less';
    @import '../../common-components/table/table.less';
</style>

<template>
    <div>
        <Row class="margin-top-10">
            <Col span="24">
            <Card dis-hover>
                <Row>
                    <Select style="width:200px" v-model="hostId" v-show="hostShow" filterable>
                        <Option v-for="item in hostData" :value="item.id" :key="item.id">{{ item.minion_id }}</Option>
                    </Select>
                    <div style="float: right;">
                        <slot name="create"></slot>
                        <slot name="downMenu"></slot>
                        <Button type="primary" @click="refresh()">刷新</Button>
                    </div>
                </Row>
                <Row>
                    <hr class="hr-margin" color="#e3e8ee" size="0.5">
                </Row>
                <Row>
                    <div style="float: right;">
                        <Input v-model.trim="nSearchVal" @on-change="handleSearch">
                        <Button slot="append" icon="ios-search"></Button>
                        </Input>
                    </div>
                    <div style="margin-bottom: -10px;">
                        <Button type="primary" @click="exportData(1)">导出数据</Button>
                        <Poptip placement="bottom-start">
                            <Button type="primary">自定义列</Button>
                            <div slot="content">
                                <ul>
                                    <li v-for="(c, i) in nColumns" v-if="i > 0" :key="i">
                                        <Checkbox :value="nColumnsExcept.indexOf(c.key) === -1"
                                                  @on-change="columnsExcept(c.key)">{{ c.title }}
                                        </Checkbox>
                                    </li>
                                </ul>
                            </div>
                        </Poptip>
                        <Dropdown>
                            <Button type="primary">
                                显示条数
                                <Icon type="arrow-down-b"></Icon>
                            </Button>
                            <DropdownMenu slot="list">
                                <DropdownItem>
                                    <div @click="customPage(5)">5</div>
                                </DropdownItem>
                                <DropdownItem>
                                    <div @click="customPage(10)">10</div>
                                </DropdownItem>
                                <DropdownItem>
                                    <div @click="customPage(50)">50</div>
                                </DropdownItem>
                                <DropdownItem>
                                    <div @click="customPage(100)">100</div>
                                </DropdownItem>
                                <DropdownItem divided>
                                    <div @click="customPage(pageCount)">全部</div>
                                </DropdownItem>
                            </DropdownMenu>
                        </Dropdown>
                        <Dropdown>
                            <Button type="primary">
                                常见功能
                                <Icon type="arrow-down-b"></Icon>
                            </Button>
                            <DropdownMenu slot="list">
                                <DropdownItem>
                                    <div @click="add('formValidate')">手动导入</div>
                                </DropdownItem>
                                <DropdownItem>
                                    <div @click="batchImport()">批量导入</div>
                                </DropdownItem>
                                <DropdownItem>
                                    <div @click="configGenerate()">配置生成</div>
                                </DropdownItem>
                                <DropdownItem>
                                    <div @click="pingAll()">一键测通</div>
                                </DropdownItem>
                                <DropdownItem>
                                    <div @click="truncateTable()">清空数据</div>
                                </DropdownItem>
                            </DropdownMenu>
                        </Dropdown>
                        <slot name="customFunction"></slot>
                    </div>
                    <br>
                    <Table :border="showBorder" :loading="loading" :data="tableData" :columns="filterColumns" stripe
                           ref="table" @on-selection-change="handleRowChange"></Table>
                    <div style="margin:10px 0px 10px 0px;overflow: hidden">
                        <slot name="selectAll"></slot>
                        <slot name="notSelectAll"></slot>
                        <slot name="accept"></slot>
                        <slot name="reject"></slot>
                        <slot name="delete"></slot>
                        <div style="float: right;">
                            <Page :total="pageCount" :current="pageCurrent" :page-size="pageSize" show-total
                                  show-elevator @on-change="changePage"></Page>
                        </div>
                    </div>
                </Row>
                <slot name="option"></slot>
            </Card>
            </Col>
        </Row>
        <Modal slot="option" v-model="formView" :title="optionTypeName">
            <Form ref="formValidate" :model="formValidate" :rules="ruleImportValidate" :label-width="125">
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
                <Button type="primary" @click="handleImport('formValidate')">提交</Button>
            </div>
        </Modal>
        <Modal slot="option" v-model="batchImportView" :title="batchImportName">
            <Card dis-hover>
                <Form ref="formValidate" :model="formValidate" :rules="ruleImportValidate" :label-width="46">
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
        <Modal slot="option" v-model="configGenerateView" :title="configGenerateName">
            <Form ref="formConfigValidate" :model="formConfigValidate" :rules="ruleConfigValidate" :label-width="125">
                <FormItem label="型号关键词" prop="key_word">
                    <Input v-model="formConfigValidate.key_word" placeholder="和搜索预览词一致,为空则全选"></Input>
                </FormItem>
                <FormItem label="生成地址" prop="path">
                    <Input v-model="formConfigValidate.path"
                           placeholder="生成地址，不填则为默认地址 /usr/local/prometheus/conf.d/ "></Input>
                </FormItem>
                <FormItem label="文件名" prop="file_name">
                    <Input v-model="formConfigValidate.file_name" placeholder="文件名,如 snmpconf_h3cS6900.json"></Input>
                </FormItem>
            </Form>
            <div slot="footer">
                <Button type="primary" @click="handleGenerate('formConfigValidate')">提交</Button>
            </div>
        </Modal>
        <Modal slot="option" v-model="singleImportView" :title="configGenerateName">
            <Form ref="formImportValidate" :model="formImportValidate" :rules="ruleImportValidate" :label-width="125">
                <FormItem label="target" prop="target">
                    <Input v-model="formImportValidate.target" placeholder="和搜索预览词一致,为空则全选"></Input>
                </FormItem>
            </Form>
            <div slot="footer">
                <Button type="primary" @click="handleImport('formImportValidate')">提交</Button>
            </div>
        </Modal>
    </div>

</template>

<script>
    function nCopy(data) {
        return JSON.parse(JSON.stringify(data));
    };
    export default {
        name: 'CommonTable',
        data() {
            return {
                nLocalColExcept: [],
                tableData: [],
                hostData: this.hostList(),
                groupData: this.groupList(),
                hostId: '',
                groupId: '',
                pageSize: 10,
                pageCurrent: 1,
                pageCount: this.pageCount,
                // 搜索
                nSearchVal: '',
                tmpData: [],
                showBorder: true,
                loading: true,
                nData: [],
                nColumns: this.cColumns,
                formView: false,
                batchImportView: false,
                configGenerateView: false,
                optionType: '',
                optionTypeName: '',
                batchImportName: '批量导入',
                configGenerateName: '配置生成',
                formValidate:{},
                formImportValidate: {
                    host_id: '',
                    target: '',
                    IP: '',
                    location: '',
                    model: '',
                    type: '',
                    project: '',
                    client: '',
                    pool: '',
                },
                ruleImportValidate: {
                    target: [
                        {required: true, message: 'target 不能为空', trigger: 'blur'}
                    ],
                    IP: [
                        {required: true, message: 'IP 不能为空', trigger: 'blur'}
                    ],
                    location: [
                        {required: true, message: 'location 不能为空', trigger: 'blur'}
                    ],
                    model: [
                        {required: true, message: 'model 不能为空', trigger: 'blur'}
                    ],
                    type: [
                        {required: true, message: 'type 不能为空', trigger: 'blur'}
                    ],
                    project: [
                        {required: true, message: 'project 不能为空', trigger: 'blur'}
                    ],
                    client: [
                        {required: true, message: 'client 不能为空', trigger: 'blur'}
                    ],
                    pool: [
                        {required: true, message: 'pool 不能为空', trigger: 'blur'}
                    ]
                },
                formConfigValidate: {
                    path: '',
                    file_name: '',
                    key_word: ''
                },
                ruleConfigValidate: {
                    key_word: [
                        {required: true, message: '设备关键词不能为空', trigger: 'blur'}
                    ]
                }

            };
        },
        props: {
            cColumns: {
                type: Array,
                required: true
            },
            apiService: {
                type: String,
                required: true
            },
            hostShow: {
                type: Boolean
            },
            groupShow: {
                type: Boolean
            }
        },
        watch: {
            // 监控产品线变化
            hostId() {
                this.loading = true;
                this.pageCurrent = 1;
                this.pageSize = 10;
                this.tableList();
                // 产品线变化后传递产品线信息给父组件
                this.getHost();
            }
        },
        computed: {
            nColumnsExcept() {
                return this.nColExcept || this.nLocalColExcept;
            },
            // 过滤列
            filterColumns() {
                return this.nColumns.filter(x => {
                    return this.nColumnsExcept.indexOf(x.key) === -1;
                });
            },
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
            // 更改显示条数
            customPage(num) {
                this.pageSize = num;
                let list = [];
                // 如果没有进行搜索tmpData是原始全部数据,如果进行了搜索tmpData为搜索或的数据
                list = nCopy(this.tmpData);
                list.splice(this.pageSize, this.pageCount);
                this.tableData = list;
                // 初始化到第一页
                this.pageCurrent = 1;
            },
            tableList() {
                this.axios.get(this.Global.serverSrc + this.apiService + '?host_id=' + this.hostId).then(
                    res => {
                        if (res.data['status'] === true) {
                            this.tableData = res.data['data'];
                            this.pageCount = this.tableData.length;
                            // nData 为原始数据始终不能改变
                            this.nData = nCopy(this.tableData);
                            // tmpData 初始值与nData相同,后面用于搜索,翻页,改变表格条数使用
                            this.tmpData = nCopy(this.tableData);
                            this.tableData.splice(this.pageSize, this.pageCount);
                            this.pageCurrent = 1;
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
                                this.tableData = [];
                            } else {
                                this.nError('Get Info Failure', errInfo);
                            }
                        } catch (error) {
                            errInfo = err;
                            this.nError('Get Info Failure', errInfo);
                        }
                        this.tableData = [];
                        this.loading = false;
                    });
            },
            tableListPing() {
                let postData = {
                    'host_id': this.hostId,
                };
                this.axios.post(this.Global.serverSrc + this.apiService + '/ping', postData).then(
                    res => {
                        if (res.data['status'] === true) {
                            this.tableData = res.data['data'];
                            this.pageCount = this.tableData.length;
                            // nData 为原始数据始终不能改变
                            this.nData = nCopy(this.tableData);
                            // tmpData 初始值与nData相同,后面用于搜索,翻页,改变表格条数使用
                            this.tmpData = nCopy(this.tableData);
                            this.tableData.splice(this.pageSize, this.pageCount);
                            this.pageCurrent = 1;
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
                                this.tableData = [];
                            } else {
                                this.nError('Get Info Failure', errInfo);
                            }
                        } catch (error) {
                            errInfo = err;
                            this.nError('Get Info Failure', errInfo);
                        }
                        this.tableData = [];
                        this.loading = false;
                    });
            },
            hostList() {
                this.axios.get(this.Global.serverSrc + 'host/target').then(
                    res => {
                        if (res.data['status'] === true) {
                            this.hostData = res.data['data'];
                            if (this.hostData.length > 0) {
                                this.hostId = this.hostData[0].id;
                            } else {
                                this.loading = false;
                            }
                        } else {
                            this.loading = false;
                            this.nError('Get Host Failure', res.data['message']);
                        }
                    },
                    err => {
                        let errInfo = '';
                        try {
                            errInfo = err.response.data['message'];
                        } catch (error) {
                            errInfo = err;
                        }
                        this.loading = false;
                        this.nError('Get Host Failure', errInfo);
                    });
            },
            groupList() {
                this.axios.get(this.Global.serverSrc + 'groups/target').then(
                    res => {
                        if (res.data['status'] === true) {
                            this.groupData = res.data['data'];
                            if (this.groupData.length > 0) {
                                this.groupId = this.groupData[0].id;
                            } else {
                                this.loading = false;
                            }
                        } else {
                            this.loading = false;
                            this.nError('Get group Failure', res.data['message']);
                        }
                    },
                    err => {
                        let errInfo = '';
                        try {
                            errInfo = err.response.data['message'];
                        } catch (error) {
                            errInfo = err;
                        }
                        this.loading = false;
                        this.nError('Get Host Failure', errInfo);
                    });
            },
            truncateTable(name) {
                let postData = {
                    'host_id': this.hostId,
                };
                this.axios.post(this.Global.serverSrc + this.apiService + '/truncate', postData).then(
                    res => {
                        if (res.data['status'] === true) {
                            this.$Message.success('清空完成！');
                            this.refresh();
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
                        this.nError('Truncate Failure', errInfo);
                    });
            },
            add(name) {
                this.handleReset(name);
                this.optionTypeName = '添加';
                this.singleImportView = true;
            },
            handleImport(name) {
                this.$refs[name].validate((valid) => {
                    if (valid) {
                        // 编辑
                        this.$Message.success('生成中，稍等');

                    } else {
                        this.$Message.error('请检查表单数据！');
                    }
                });
            },
            handleSubmit(name) {
                this.$refs[name].validate((valid) => {
                    if (blank) {
                        this.formValidate.host_id = this.hostId;
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
                    } else {
                        this.$Message.error('请检查表单数据！');
                    }
                });
            },
            handleReset(name) {
                this.$refs[name].resetFields();
            },
            batchImport(name) {
                this.batchImportView = true;
            },
            UploadSuccess(res) {
                if (res.message === '') {
                    this.$Message.success('上传成功' + res.message);
                    this.refresh();
                } else {
                    this.nError('上传失败', res.message);
                    this.refresh();
                }
            },
            // 上传失败
            UploadError(err) {
                this.nError('Upload Failure', '上传文件格式错误或其他异常');
            },
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
            configGenerate(name) {
                this.configGenerateView = true;
            },
            handleGenerate(name) {
                this.$refs[name].validate((valid) => {
                    if (valid) {
                        // 编辑
                        this.$Message.success('生成中，稍等');
                        let postData = {
                            'host_id': this.hostId,
                            'key_word': this.formConfigValidate.key_word,
                            'path': this.formConfigValidate.path,
                            'file_name': this.formConfigValidate.file_name,
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
                    } else {
                        this.$Message.error('请检查表单数据！');
                    }
                });
            },
            pingAll(name) {
                this.tableListPing();
                this.$Message.success('不通设备列表如下:');
            },

            // 重新定义错误消息
            nError(title, info) {
                this.$Notice.error({
                    title: title,
                    // 替换<>避免被解析为html标签
                    desc: info.toString().replace(/<|>/g, ''),
                    duration: 10
                });
            },
            // 刷新表格数据
            refresh() {
                this.loading = true;
                this.tableList();
            },
            columnsExcept(key) {
                let index = this.nColumnsExcept.indexOf(key);
                if (index === -1) {
                    this.nColumnsExcept.push(key);
                } else {
                    this.nColumnsExcept.splice(index, 1);
                }
            },
            changePage(page) {
                let list = [];
                // 如果没有进行搜索tmpData是原始全部数据,如果进行了搜索tmpData为搜索或的数据
                list = nCopy(this.tmpData);
                this.pageCurrent = page;
                this.tableData = list.splice((page - 1) * this.pageSize, this.pageSize);
            },
            // 导出表格数据
            exportData(type) {
                if (type === 1) {
                    this.$refs.table.exportCsv({
                        filename: 'saltshaker',
                        data: this.nData
                    });
                } else if (type === 2) {
                    this.$refs.table.exportCsv({
                        filename: 'saltshaker',
                        original: false
                    });
                } else if (type === 3) {
                    this.$refs.table.exportCsv({
                        filename: 'saltshaker',
                        columns: this.nColumns.filter((col, index) => index < 4),
                        data: this.tableData.filter((data, index) => index < 4)
                    });
                }
            },
            search(data, searchVal) {
                // 最终的是最字符串的搜索,达到模糊匹配的效果
                let res = data;
                let dataClone = data;
                let searchResult = [];
                if (searchVal.length > 0) {
                    for (let i = 0; i < this.nColumns.length; i++) {
                        let key = this.nColumns[i]['key'];
                        if (key !== 'action' && key !== undefined) {
                            res = dataClone.filter(d => {
                                let value = d[key];
                                // 如果是数组在进行filter
                                if (value instanceof Array) {
                                    let tmp = value.filter(v => {
                                        // 如果是数组里面是对象在搜索
                                        if (v instanceof Object) {
                                            let r = false;
                                            for (let s in v) {
                                                let z = '';
                                                if (v[s] !== undefined) {
                                                    z = v[s].toString().toLowerCase();
                                                }
                                                if (z.indexOf(searchVal.toLowerCase()) > -1) {
                                                    r = true;
                                                    return r;
                                                }
                                            }
                                        } else {
                                            let y = '';
                                            if (v !== undefined) {
                                                y = v.toString().toLowerCase();
                                            }
                                            return y.indexOf(searchVal.toLowerCase()) > -1;
                                        }
                                    });
                                    return tmp.length > 0;
                                } else {
                                    // 转换成字符串,数字没有indexOf方法
                                    let x = '';
                                    if (value !== undefined) {
                                        x = value.toString().toLowerCase();
                                    }
                                    return x.indexOf(searchVal.toLowerCase()) > -1;
                                }
                            });
                            searchResult = searchResult.concat(res);
                        }
                    }
                    // 数组去重
                    return [...new Set(searchResult)];
                } else {
                    return res;
                }
            },
            handleSearch() {
                // 获取原始数据
                this.tableData = nCopy(this.nData);
                // 获取搜索后的数据
                this.tableData = this.search(this.tableData, this.nSearchVal);
                // 获取搜索后的长度
                this.pageCount = this.tableData.length;
                // 给tmpData赋值为搜索后的数据
                this.tmpData = nCopy(this.tableData);
                // 展示默认行数的数据
                this.tableData.splice(this.pageSize, this.pageCount);
                // 切换到第一页
                this.pageCurrent = 1;
            },
            // 删除数据
            del(id) {
                this.axios.delete(this.Global.serverSrc + this.apiService + '/' + id).then(
                    res => {
                        if (res.data['status'] === true) {
                            this.tableData.splice(this.delIndex, 1);
                            this.$Message.success('删除成功！');
                            this.tableList();
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
            // 传递给父组件
            getHost() {
                this.$emit('getHostEvent', this.hostData, this.hostId);
            },
            getTable() {
                this.$emit('getTableEvent', this.tableData);
            },
            // 传递选择的行数据给父组件
            handleRowChange(currentRow) {
                this.$emit('getRowEvent', currentRow);
            }
        }
    };
</script>
<style scoped>
    .hr-margin {
        margin-top: 10px;
        margin-bottom: 10px;
    }
</style>