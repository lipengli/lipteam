<template>
    <drawer-tabs-container>
        <div class="project-my-manage">
            <!-- 列表 -->
            <Table class="tableFill" ref="tableRef" :columns="columns" :data="lists" :loading="loadIng > 0" :no-data-text="noDataText" stripe></Table>
            <!-- 分页 -->
            <Page class="pageBox" :total="listTotal" :current="listPage" :disabled="loadIng > 0" @on-change="setPage" @on-page-size-change="setPageSize" :page-size-opts="[10,20,30,50,100]" placement="top" show-elevator show-sizer show-total transfer></Page>
        </div>
    </drawer-tabs-container>
</template>

<style lang="scss" scoped>
    .project-my-manage {
        .tableFill {
            margin: 12px 12px 20px;
        }
    }
</style>
<script>
    import Project from "../../../mixins/project";
    import DrawerTabsContainer from "../../DrawerTabsContainer";

    export default {
        name: 'ProjectMyManage',
        components: {DrawerTabsContainer},
        props: {
            canload: {
                type: Boolean,
                default: true
            },
        },
        mixins: [
            Project
        ],
        data () {
            return {
                loadYet: false,

                loadIng: 0,

                columns: [],

                lists: [],
                listPage: 1,
                listTotal: 0,
                noDataText: "",
            }
        },
        created() {
            this.noDataText = this.$L("数据加载中.....");
            this.columns = [{
                "title": this.$L("项目名称"),
                "key": 'title',
                "minWidth": 100,
                render: (h, params) => {
                    return h('a', {
                        attrs: {
                            href: 'javascript:void(0)',
                        },
                        on: {
                            click: () => {
                                this.openProject(params.row.id);
                            }
                        }
                    }, params.row.title);
                },
            }, {
                "title": this.$L("创建时间"),
                "minWidth": 160,
                render: (h, params) => {
                    return h('span', $A.formatDate("Y-m-d H:i:s", params.row.indate));
                }
            }, {
                "title": this.$L("操作"),
                "key": 'action',
                "width": 80,
                "align": 'center',
                render: (h, params) => {
                    return h('Button', {
                        props: {
                            type: 'primary',
                            size: 'small'
                        },
                        style: {
                            fontSize: '12px'
                        },
                        on: {
                            click: () => {
                                this.deleteProject(params.row.id, () => {
                                    this.getLists();
                                });
                            }
                        }
                    }, this.$L('删除'));
                }
            }];
        },
        mounted() {
            if (this.canload) {
                this.loadYet = true;
                this.getLists(true);
            }
        },

        watch: {
            canload(val) {
                if (val && !this.loadYet) {
                    this.loadYet = true;
                    this.getLists(true);
                }
            }
        },

        methods: {
            setPage(page) {
                this.listPage = page;
                this.getLists();
            },

            setPageSize(size) {
                if (Math.max($A.runNum(this.listPageSize), 10) != size) {
                    this.listPageSize = size;
                    this.getLists();
                }
            },

            getLists(resetLoad) {
                if (resetLoad === true) {
                    this.listPage = 1;
                }
                this.loadIng++;
                this.noDataText = this.$L("数据加载中.....");
                $A.aAjax({
                    url: 'project/lists',
                    data: {
                        act: 'manage',
                        page: Math.max(this.listPage, 1),
                        pagesize: Math.max($A.runNum(this.listPageSize), 10),
                    },
                    complete: () => {
                        this.loadIng--;
                    },
                    error: () => {
                        this.noDataText = this.$L("数据加载失败！");
                    },
                    success: (res) => {
                        if (res.ret === 1) {
                            this.lists = res.data.lists;
                            this.listTotal = res.data.total;
                            this.noDataText = this.$L("没有相关的数据");
                        } else {
                            this.lists = [];
                            this.listTotal = 0;
                            this.noDataText = res.msg;
                        }
                    }
                });
            },
        }
    }
</script>
