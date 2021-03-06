<template>
    <div class="gulu-table-wrapper" ref="wrapper">
        <div :style="{overflow:'auto',height:`${scrollHeight}px`}" ref="tableWrapper">
            <table class="gulu-table"
                   :class="{bordered,compact,striped,cell}"
                   ref="table">
                <thead>
                <tr>
                    <th :style="{width: '50px'}" v-if="checkBox" style="text-align: center;">
                        <input type="checkbox"
                               @click="onClickAll"
                               ref="allChecked"
                               :checked="areAllItemsSelected">
                    </th>
                    <th :style="{width: '50px'}" v-if="expandDescription"></th>
                    <th v-for="column in columns" :style="{width:column.width+'px'}">
                        <span class="gulu-table-sort-head"
                              :class="{'gulu-table-unordered':!sortRules[column.field]}"
                              @click="(sortRules[column.field]!==undefined)&&OnClickSort(column.field,sortRules[column.field])">
                            {{column.title}}
                            <template v-if="sortRules[column.field]!==undefined">
                                <g-icon name="sort" v-if="!clickSortStatus||sortRules[column.field]===''"></g-icon>
                                <g-icon v-else name="sort-desc"
                                        :class="{asc:sortRules[column.field]==='asc'}"
                                ></g-icon>
                            </template>
                        </span>
                    </th>
                    <th v-if="$scopedSlots.handle" ref="actionsHeader"></th>
                </tr>
                </thead>
                <tbody>
                <template v-for="(item,index) in dataSource">
                    <tr :key="item.id" :class="{stripedItem:index%2===0}">
                        <td :style="{width: '50px'}" v-if="checkBox" class="gulu-table-checkbox">
                            <input type="checkbox" @click="onClickItem($event,item)"
                                   :checked="onChangeItem(item)">
                        </td>
                        <td :style="{width: '50px'}" v-if="expandDescription">
                            <div class="gulu-table-expand-icon"
                                 v-if="item[expandDescription]"
                                 @click="onClickExpand(item.id)">
                                <g-icon name="right"
                                        :class="{'gulu-table-expanded':inExpandIds(item.id)}">
                                </g-icon>
                            </div>
                        </td>
                        <td v-for="column in columns" :style="{width:column.width+'px'}">
                            <template v-if="column.render">
                                <vnodes :vnodes="column.render({value:item[column.field]})"></vnodes>
                            </template>
                            <template v-else>
                                {{item[column.field]}}
                            </template>
                        </td>
                        <td v-if="$scopedSlots.handle" ref="actions">
                            <div class="gulu-table-handle">
                                <slot name="handle" :item="item"></slot>
                            </div>
                        </td>
                    </tr>
                    <tr v-if="inExpandIds(item.id)" class="gulu-table-expand">
                        <td :colspan="expandColspan">
                            {{item[expandDescription]}}
                        </td>
                    </tr>
                </template>
                </tbody>
            </table>
        </div>
        <div class="gulu-table-loading" v-if="loading">
            <g-icon name="loading"></g-icon>
        </div>
    </div>
</template>

<script>
    import GIcon from "../icon.vue"

    export default {
        name: "GuluTable",
        components: {
            GIcon,
            vnodes: {
                functional: true,
                render: (h, context) => context.props.vnodes
            }
        },
        data() {
            return {
                expandIds: [],
                columns: [],
                clickSortStatus: false
            }
        },
        props: {
            expandDescription: {
                type: String
            },
            scrollHeight: {
                type: Number
            },
            dataSource: {
                type: Array,
                required: true,
                validator: (array) => {
                    return array.filter((item) => item.id === undefined).length <= 0
                }
            },
            selected: {
                type: Array,
                default: () => []
            },
            sortRules: {
                type: Object,
                default: () => ({})
            },
            bordered: {
                type: Boolean,
                default: true
            },
            compact: {
                type: Boolean,
                default: false
            },
            striped: {
                type: Boolean,
                default: true
            },
            loading: {
                type: Boolean,
                default: false
            },
            checkBox: {
                type: Boolean,
                default: false
            },
            cell: {
                type: Boolean,
                default: false
            },
        },
        mounted() {
            this.columns = this.$slots.default.filter(node =>
                node.componentOptions
            ).map((node) => {
                let {title, field, width} = node.componentOptions && node.componentOptions.propsData
                let render = node.data && node.data.scopedSlots.default
                return {title, field, width, render}
            })


            let table2 = this.$refs.table.cloneNode(false)
            table2.classList.add('gulu-table-copy')

            this.$nextTick(() => {
                let thead = this.$refs.table.getElementsByTagName('thead')[0]
                table2.appendChild(thead)

                this.$refs.wrapper.appendChild(table2)

                let {height} = thead.getBoundingClientRect()
                this.$refs.wrapper.style.marginTop = height + 'px'
                table2.style.top = -height + 'px'
            })

        },
        watch: {
            selected() {
                let {length: selectedLength} = this.selected
                let {length: dataSourceLength} = this.dataSource
                if (selectedLength === dataSourceLength) {
                    this.$refs.allChecked.indeterminate = false
                } else if (selectedLength < dataSourceLength && selectedLength > 0) {
                    this.$refs.allChecked.indeterminate = true
                } else if (selectedLength === 0) {
                    this.$refs.allChecked.indeterminate = false
                }
            }
        },
        computed: {
            areAllItemsSelected() {
                if (this.selected.length !== this.dataSource.length) return false
                let a = this.selected.map((item) => item.id).sort((a, b) => a - b)
                let b = this.dataSource.map((item) => item.id).sort((a, b) => a - b)
                for (let i = 0; i < a.length; i++) {
                    if (a[i] !== b[i]) return false
                }
                return true
            },
            expandColspan() {
                let colLength = this.columns.length
                if (this.expandDescription) {
                    colLength++
                }
                if (this.checkBox) {
                    colLength++
                }
                if (this.$scopedSlots.handle) {
                    colLength++
                }
                return colLength
            }
        },
        methods: {
            inExpandIds(id) {
                return this.expandIds.indexOf(id) >= 0
            },
            onClickExpand(id) {
                let index = this.expandIds.indexOf(id)
                if (index >= 0) {
                    this.expandIds.splice(index, 1)
                } else {
                    this.expandIds.push(id)
                }
            },
            OnClickSort(field, lastValue) {
                if (!this.clickSortStatus) {
                    this.clickSortStatus = true
                    lastValue = ''
                }
                let copy = JSON.parse(JSON.stringify(this.sortRules))
                Object.keys(copy).map((node) => {
                    copy[node] = ''
                })
                if (lastValue === 'asc') {
                    copy[field] = 'desc'
                } else if (lastValue === 'desc'||lastValue === '') {
                    copy[field] = 'asc'
                }
                this.$emit('update:sortRules', copy)
            },
            onChangeItem(item) {
                return this.selected.filter((vm) => vm.id === item.id).length > 0
            },
            onClickItem(e, item) {
                let copy = JSON.parse(JSON.stringify(this.selected))
                if (e.target.checked) {
                    copy.push(item)
                } else {
                    let itemIndex
                    copy.forEach((vm, index) => {
                        if (vm.id === item.id) {
                            itemIndex = index
                        }
                    })
                    copy.splice(itemIndex, 1)
                }
                this.$emit('update:selected', copy)
            },
            onClickAll(e) {
                this.$emit('update:selected', e.target.checked ? this.dataSource : [])
            },
        }
    }
</script>

<style scoped lang="scss">
    @import "../../styles/var";
    .gulu-table {
        border-collapse: collapse;
        width: 100%;
        table-layout: fixed;
        display: table;
        &.bordered {border: 1px solid darken($grey, 10%);}
        &.cell {
            & th {border: 1px solid darken($grey, 10%);}
            & td {border: 1px solid darken($grey, 10%);}
        }
        & thead > tr {background-color: #fefefe;box-shadow: #efefef 0 0 5px 1px inset;color: #2d2d2d;}
        & tbody > tr {border-bottom: 1px solid darken($grey, 3%);}
        & th {text-align: left;padding: 8px;}
        & td {padding: 8px;}
        &.compact {
            & th {padding: 4px;}
            & td {padding: 4px;}
        }
        &.striped {
            & .stripedItem {background-color: $grey;}
        }
        &-sort-head {
            display: inline-flex;
            align-items: center;
            cursor: pointer;
            & .asc {transform: rotate(180deg);}
            &.gulu-table-unordered {cursor: default;}
        }
        &-wrapper {position: relative;}
        &-loading {position: absolute;top: 0;left: 0;
            display: flex;justify-content: center;align-items: center;
            background-color: rgba(255, 255, 255, .8); width: 100%;height: 100%;
            svg {@include spin;width: 10%;height: 10%;}
        }
        &-copy {position: absolute;top: 0;left: 0;background-color: #fff;
        }
        &-expand {
            background-color: lighten($grey, 4%);
            text-align: center;
            transition: height 5s;
        }
        &-expand-icon {display: flex;align-items: center;justify-content: center;}
        &-expanded {transform: rotate(90deg);transition: all .3s;}
        &-handle {display: flex;}
        &-checkbox {text-align: center;}
    }
</style>