<!--created By Lvyl-->
<!--功能：
1、根据slot内容填充列表，即列表显示的列由调用该组件的父组件决定；
2、该组件处理了翻页功能，翻页相关的参数_page、_page_size在该组件内设置；
3、该组件根据地址栏参数来筛选列表，除了翻页相关的参数在该组件内设置，其余的筛选条件都应该在父组件中设置；
    父组件中可以随意设置url的查询参数，该组件都能做出响应刷新列表。
-->
<template>
    <div>
        <slot v-bind:list="list"></slot>
        <div class="mt10 clean-float">
            <el-pagination
                    v-if="!hidePage"
                    :class="{'pull-right':position=='right'}"
                    @size-change="sizeChange"
                    @current-change="pageChange"
                    :current-page="meta.currentPage"
                    :page-sizes="pageSizes"
                    :page-size="meta.perPage"
                    :layout="layoutShow"
                    :total="meta.totalCount">
            </el-pagination>
        </div>
    </div>
</template>

<script>
    export default {
        name: "pageList",
        props:[
            'pageListApi', //列表的api
            'ignore', //检查查询参数，并去掉ignore中设置的参数；如果不设置，则不检查
            'need', //检查查询参数，如果参数中包含了need中设置的所有参数，才查询；如果不设置，则不检查
            'range', //查询范围，设置了此参数，则只在该范围内查询，不在此范围内的查询参数不作为查询条件
            'layout',   //显示哪些
            'position', //显示在哪儿，默认left,可传的值right
            'hidePage', // 隐藏翻页
            'noAuto',
            'expand',  // 从后端获取的额外的字段
        ],
        data() {
            return {
                search: {},
                list: [],
                meta: {
                    _page_size: 20,
                    currentPage: 1
                },
                pageSizes: [20, 50, 100],
            }
        },
        computed:{
            layoutShow:function () {
                return this.layout || 'total, sizes, prev, pager, next, jumper';
            }
        },
        methods: {
            // 每页显示条数更改触发
            sizeChange(size) {
                this.$router.push({
                    name: this.$route.name,
                    query: Object.assign({}, this.search, {_page_size: size, _page: 1})
                })
            },
            // 翻页
            pageChange(page) {
                this.$router.push({
                    name: this.$route.name,
                    query: Object.assign({}, this.search, {_page: page})
                })
            },
            // 获取列表数据
            getList(cb) {
                var that = this;
                var data;
                // 如果有需要忽略的字段，则去掉查询参数里面的这些字段
                if(this.ignore&&this.ignore.length>0){
                    data = myTool.deleteProps(this.search,this.ignore)
                }else{
                    data = Object.assign({},this.search);
                }

                // 设置了范围的话，只在该范围内取参数
                if(this.range&&this.range.length>0){
                    data = myTool.trimObj(data,this.range);
                }

                if(this.hidePage){
                    data._page_size = -1;
                }
                if(this.expand){
                    data.expand = this.expand;
                }
                if(this.need&&this.need.length>0){
                    // 如果设置了need,则data中必须包含need中设置的所有参数，才开始查询
                    //使用场景：进入某个页面必须携带某个参数才显示列表，否则不显示列表
                    let isPass = 1;
                    for(let i=0;i<this.need.length;i++){
                        if(!data.hasOwnProperty(this.need[i])){
                            isPass = 0;
                            break;
                        }
                    }
                    if(isPass){
                        queryData();
                    }
                }else{
                    queryData();
                }

                function queryData() {
                    that.pageListApi.query({params: data}).then(function (res) {
                        that.list = res.data.items;
                        that.meta = res.data['_meta'];
                        cb && cb(that.list);
                    });
                }

            }
        },
        created: function () {
            // 初始化页面
            this.search = Object.assign({},this.$route.query);
            if(!this.noAuto){
                this.getList();
            }
        },
        watch: {
            // 监听路由，刷新列表
            $route: {
                handler(n, o) {
                    this.search = Object.assign({},this.$route.query);
                    if(!this.noAuto){
                        this.getList();
                    }
                },
                deep: true
            }
        }
    }
</script>

<style lang="less" scoped>

</style>