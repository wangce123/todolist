<script>
import { Input, Button, Dialog, Table, TableColumn, Pagination } from 'element-ui'
import FilterGroup from '../form-render'

export default {
	name: 'FilterSelectPage',
	components: {
		ElInput: Input,
		ElButton: Button,
		ElDialog: Dialog,
		ElTable: Table,
		ElTableColumn: TableColumn,
		FilterGroup,
		ElPagination: Pagination,
	},
	props: {
		value: [String],
		prop: [String],
		config: {
			type: Object,
			default: () => ({
				api: {
					type: Function,
					default: () => (new Promise()),
				},
				tableConfig: [
					{
						key: '',
						label: '',
					},
				],
				filterConfig: [
				],
			}),
		},
	},
	data() {
		return {
			dialogVisible: false,
			queryState: {},
			// 选择的地块
			orignData: {},
			tableData: [],
			pagination: {
				total: 0,
				current: 1,
				size: 5,
				sizes: [5, 10, 20],
			},
		}
	},
	methods: {
		handleOpen() {
			this.dialogVisible = true
			this.queryData()
		},
		handleClose() {
			Object.assign(this.$data, this.$options.data())
			this.dialogVisible = false
		},
		handleSearch() {
			this.queryData()
		},
		async queryData() {
			const params = {
				...this.queryState,
				pageNum: this.pagination.current,
				pageSize: this.pagination.size,
			}
			const { data: { records, total } } = await this.config.api(params)
			this.tableData = records
			this.pagination.total = total
		},
		handleSizeChange(val) {
			this.pagination.size = val
			this.queryData()
		},
		handleCurrentChange(val) {
			this.pagination.current = val
			this.queryData()
		},
		handleCheck(row) {
			this.orignData = row
		},
		handleConfirm() {
			this.$emit('input', this.prop, this.orignData[this.prop], this.orignData)
			this.handleClose()
		},
	},
	computed: {
		getTableColumn() {
			return [{}].concat(this.config.tableConfig).map(
				({ label, key, elAttrs = {}, render, ...reset }, index) => {
					let scopedSlots = {}
					if (render) {
						scopedSlots = {
							default: (scope) => {
								return render(this.$createElement, scope)
							},
						}
					}
					let column
					if (index === 0) {
						scopedSlots={
						default: ({ row }) => (
						<input type="radio" 
						checked={this.orignData.id === row.id}/>
						),
						}
						column = (
						<el-table-column show-overflow-tooltip
							prop={key}
							label={label}
							width="50px"
							{...{
								attrs: {
									...elAttrs,
								},
								scopedSlots,
								props: {
									...reset,
								},
							}}
						></el-table-column>
					)
					} else {
						column = (
								<el-table-column show-overflow-tooltip
									prop={key}
									label={label}
									{...{
										attrs: {
											...elAttrs,
										},
										scopedSlots,
										props: {
											...reset,
										},
									}}
								></el-table-column>
						)
					}
					return column
				},
			)
		},
	},
	render() {
		return (
			<div class="advance-search">
				<el-input
					placeholder="请选择"
					value={this.value}
					size="mini"
					{...{
						on: {
							input: (val) => this.$emit('input', this.prop, val),
							focus: this.handleOpen,
						},
						props: { ...this.$attrs },
					}}
				></el-input>
				<el-dialog
				 visible={this.dialogVisible}
				  width="80%"
				before-close={this.handleClose}
				 close-on-click-modal={false}  
				 append-to-body
				 title={this.config.title}
				   >
					<FilterGroup
						v-model={this.queryState}
						schema={this.config.filterConfig}
						onSearch={this.handleSearch}
						onReset={() => { this.queryState = {}; this.handleSearch() }}
					/>
					<el-table
						row-key="id"
						highlight-current-row={true}
						{
							...{
								on: {
									'current-change': this.handleCheck,
								},
							}
						}
						data={this.tableData}
						style="width: 100%">
						{this.getTableColumn}
					</el-table>
					<el-pagination
						{
							...{
								on: {
									'size-change': this.handleSizeChange,
									'current-change': this.handleCurrentChange,
								},
							}
						}
						page-sizes={this.pagination.sizes}
						page-size={this.pagination.size}
						layout="total, sizes, prev, pager, next, jumper"
						total={this.pagination.total}>
					</el-pagination>
					<div class="button-group">
						<el-button onClick={this.handleClose}>取消</el-button>
						<el-button type="primary" onClick={this.handleConfirm}>确认</el-button>
					</div>
				</el-dialog>
			</div>
		)
	},
}
</script>
<style lang="scss" scoped>
.advance-search{
	display: flex;
	.el-input{
		margin-right: 15px;;
	}
}
.button-group{
	text-align: right;
}
</style>
<style lang="scss">
.el-dialog__body{
	border-top: 1px solid rgba(0, 0, 0, .1)
}
</style>