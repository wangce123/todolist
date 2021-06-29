<script>
import { Button, Row, Col, Form, FormItem } from 'element-ui'
import _ from 'lodash'

const rtx = require.context('./components', true, /\.(vue|js)$/)
const formItemList = rtx.keys().reduce((pre, cur) => {
	const pkg = rtx(cur).default || rtx(cur)
	const { name } = pkg || {}
	if (!name) return pre
	return {
		...pre,
		[name.toLocaleLowerCase()]: pkg,
	}
}, {})

const TYPETOMESSAGE = {
	input: '请输入',
	select: '请选择',
	upload: '请上传',
	cascader: '请选择',
	radio: '请选择',
	date: '请选择',
	selectpage: '请选择',
}

export default {
	name: 'FormRender',
	props: {
		schema: {
			type: Array,
			default() {
				return []
			},
		},
		value: {
			type: Object,
			default() {
				return {}
			},
        },
        config: {
            type: Object,
            default: () => ({ gutter: 0, max: 24 }),
        },
	},
	data: () => ({
		typeListenerMap: {
			select: 'input',
		},
		linkageList: [],
	}),
	components: {
		ElButton: Button,
		ElForm: Form,
		ElFormItem: FormItem,
		ElRow: Row,
		ElCol: Col,
	},
	async mounted() {
		await this.$nextTick()
		this.setLinkage()
	},
	methods: {
		async handleValidateField(prop) {
			/**
			 * 应该是此版本element的坑,其他版本数组校验的好好的这个版本第二个问题了,此处由表单组件来发起校验
			*/
			await this.$refs.form.validateField(prop)
		},
		handleInput(key, val, orign) {
			this.$emit(
				'input',
				Object.assign({}, this.value, {
					[key]: val,
				}), orign
			)
		},
		/**
		 * 必填规则补全
		 */
		getRules(rules, label, type) {
			if (_.isArray(rules)) {
				if (rules.some(item => (item.required === true || item === true) && !item.message && !item.validator)) {
					const index = rules.findIndex(item => item.required)
					rules.splice(index, 1, this.getDefRules(label, type))
					return rules
				} else {
					return rules
				}
			} else if (rules !== undefined) {
				//只传布尔值的情况
				return [this.getDefRules(label, type)]
			}
		},
		getDefRules(label, type) {
			if (type === 'text') {
				return { required: true, message: '此选项必填' }
			} else {
				return {
					required: true,
					message: TYPETOMESSAGE[type] + label
				}
			}
		},
		/**
		 * @param {linkage: 联动规则,Object}
		 * @param {prop: 联动字段,String}
		 */
		setLinkage() {
			this.getPureLinkageList.forEach(({ linkage, prop }) => {
				this.$watch(() => this.value[prop], (nv) => {
					const linkData = linkage.find(item => item.when.value === this.value[prop])
					if (!linkData.then || !linkData.then.key) {
						throw new Error('联动目标参数不存在!')
					}
					if (linkData) {
						this.$emit(
								'input',
							Object.assign({}, this.value, {
								[linkData.then.key]: linkData.then.value || '',
							}))
						if (_.isBoolean(linkData.then.hidden)) {
							/**
							 * 写法不合vue规范,但是因为是引用数据,可全数据流更改更新,先用着吧
							 */
							this.schema.forEach(item => {
								const curSchema = item.find(ele => ele.prop === linkData.then.key)
								if (curSchema) {
									this.$set(curSchema, 'hidden', linkData.then.hidden)
								}
							})
						}
					} else {
						throw new Error('联动字段值不存在!')
					}
				},
				{
					immediate: true,
				}
				)
			})
		},
		getCols(row) {
			return row.map(({ type, prop, label, rules, colspan, linkage, hidden,  ...rest }) => {
				const Form = formItemList[`filter${type}`]
				const listenerKey = this.typeListenerMap[type] || 'input'
				const value = this.value[prop]
				if (linkage) {
					if (this.linkageList.every(item => item.prop !== prop)) {
						this.linkageList.push({ prop, linkage, })	
					}
				}
				if (Form && !hidden) {
					return (
						<el-col span={colspan || this.getColSpan}>
							<el-form-item label={label} prop={prop} rules={this.getRules(rules, label, type)}>
								<Form
									{...{
										attrs: {
											value,
											prop,
											...rest,
										},
										on: {
											[listenerKey]: this.handleInput,
											validateCurField: this.handleValidateField
										},
									}}
								/>
							</el-form-item>
						</el-col>
					)
				}
				return null
			})
		},
	},
	computed: {
		getRows() {
			return this.schema.map(row => (<el-row gutter={this.config.gutter}>{this.getCols(row)}</el-row>))
		},
		getColSpan() {
			const maxLen = Math.max(...this.schema.map(item => item.length))
			return Math.floor(this.config.max / maxLen)
		},
		getPureLinkageList() {
			return _.uniqWith(this.linkageList, _.isEqual)
		},
		/**
		 * 优化label-width宽度算法,模拟dom计算宽度
		 * label-width = *宽度(5px) + margin-right: 4px; + 实际文字宽度(动态计算) + padding-right: 12px;
		 * 此属性无特殊情况,无需再从外部传入
		*/
		getLabelWidth() {
			if (this.$attrs['label-wdith']) {
				return this.$attrs['label-wdith']
			}
			let schemaList = []
			this.schema.forEach(item => {
				schemaList = [...schemaList, ...item]
			})
			const labelLenList = []
			schemaList.map(item => {
				const dom = document.createElement('div');
				dom.style.visibility = 'hidden'
				dom.style.position = 'fixed'
				dom.textContent = '*' + item.label + ':'
				document.body.appendChild(dom)
				labelLenList.push(dom.offsetWidth)
				document.body.removeChild(dom)
			})
			return (Math.max(...labelLenList) < 100 ? 100 :  Math.max(...labelLenList)) + 'px'
		},
	},
	render(h) {
		return (
			<el-form class="label-right" ref="form" model={this.value} {...{ props: { ...this.$attrs } }} label-width={this.getLabelWidth} label-suffix=":">
				{this.getRows}
			</el-form>
		)
	},
}
</script>