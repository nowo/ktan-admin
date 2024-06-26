<template>
    <my-box v-loading="defData.loading">
        <my-form-tool :data="searchData" inline @submit.prevent="onSearch">
            <el-button type="success" @click="onOpenAdd()">
                <i class="i-ep-plus mr5px" />
                新增{{ props.title }}
            </el-button>
        </my-form-tool>
        <div class="min-h-0 flex-1">
            <CoTable v-model:option="tableData" class="table-box" row-key="id" border
                @pagination="onHandleCurrentChange">
                <!-- <template #title="{ row }">
                    {{ row?.classify?.title }}
                </template> -->
                <template #status="{ row }">
                    <el-tag v-if="row.status === 1" type="">
                        是
                    </el-tag>
                    <el-tag v-else type="info">
                        否
                    </el-tag>
                </template>
                <template #operate="{ row }">
                    <el-button size="small" text type="primary" @click.stop="onOpenEdit(row)">
                        修改
                    </el-button>
                    <el-button size="small" text type="primary" @click.stop="onRowDel(row)">
                        删除
                    </el-button>
                </template>
            </CoTable>
        </div>

        <NewsModel ref="modelRef" v-bind="props" :list="tableData.data" @update="initTableData" />
    </my-box>
</template>

<script lang="ts" setup>
import NewsModel from './ClassifyModel.vue'
import { PAGINATION } from '@/config/global'
import { getGoodsClassifyList, setGoodsClassifyDelete } from '@/api/list'

const props = defineProps<{
    type: number
    title: string
}>()

const modelRef = ref<InstanceType<typeof NewsModel>>()

const defData = reactive({
    loading: false,
    menuData: [] as MenuApi_MenuItem[],
    expand: false, // 是否默认展开
})

// form表单数据类型
interface FormSearchData {
    name: string
    type: string | number
}
const searchData = reactive<SearchDataType<FormSearchData>>({
    data: {
        name: '',
        type: '',
    },
    config: [
        { itemProp: { label: '分类名称', prop: 'name' }, placeholder: '请输入分类名称', width: '200' },
    ],
    hideBtn: false,
    // showAll: true,
    searchFunc: () => initTableData(),
})

type TableDataItem = IGoodsClassifyGetListResponse['list'][0]
const tableData = reactive<CoTableProps<TableDataItem>>({
    data: [],
    tableHeader: [
        { property: 'id', label: 'id', width: 150 },
        // { property: 'img', label: '图片', width: 100, align: 'center', slot: true },

        { property: 'title', label: '分类名称', minWidth: 150 },
        // { property: 'type', label: '分类名称', width: 180, slot: true },
        { property: 'sort', label: '排序', width: 100, align: 'center' },
        { property: 'status', label: '是否显示', width: 150, align: 'center' },
        { property: 'operate', label: '操作', width: 130, fixed: 'right', align: 'center' },
    ],
    pagination: {
        ...PAGINATION,
        page_size: 50,
    },
})

const initDefaultData = async () => {
    // const res = await MenuApi.getList()
    // if (res.code !== 200) return ElMessage.error(res.msg)

    // const list = res.data.list.filter(item => item.id === 3)

    // defData.menuData = list[0]?.children || []
}

// 初始化菜单数据
const initTableData = async () => {
    const data: IGoodsGetList = {
        isPage: 0,
        // page: tableData.pagination.page,
        // pageSize: tableData.pagination.page_size,
        title: searchData.data.name?.trim() ?? '',
        type: Number(props.type) || 1,
    }
    // if (searchData.data.type) data.type = Number(searchData.data.type)

    const res = await getGoodsClassifyList(data)
    if (res.code !== 200) return ElMessage.error(res.msg)

    // console.log(res)

    tableData.data = res.data.list
    // tableData.pagination.total = res.data.total
}

// 打开新增菜单弹窗
const onOpenAdd = () => {
    modelRef.value?.openDialog()
}
// 打开编辑菜单弹窗
const onOpenEdit = (row: TableDataItem) => {
    modelRef.value?.openDialog(row)
}
// 删除当前行
const onRowDel = (row: TableDataItem) => {
    ElMessageBox.confirm('此操作将永久删除该条内容, 是否继续?', '提示', {
        confirmButtonText: '删除',
        cancelButtonText: '取消',
        type: 'warning',
        buttonSize: 'default',
    }).then(async () => {
        const res = await setGoodsClassifyDelete({ id: row.id })
        if (res.code !== 200) return ElMessage.error(res.msg)

        ElMessage.success('删除成功')
        initTableData() // 更新列表数据
    }).catch(() => { })
}
// 查询
const onSearch = () => {
    initTableData()
}

// 分页改变
const onHandleCurrentChange = () => {
    initTableData()
}

// 加载时
onBeforeMount(() => {
    initDefaultData()
    initTableData()
})
</script>

<style lang="scss" scoped>
.table-box {
    :deep(.el-table__body-wrapper) {
        .row-info-bg {
            --el-table-tr-bg-color: var(--el-color-info-light-8);
            opacity: 0.8;
        }
    }
}
</style>
