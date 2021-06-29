<script>
import { Select, Option } from 'element-ui'

export default {
	name: 'FilterSelect',
	components: {
		ElSelect: Select,
		ElOption: Option,
	},
	props: {
		value: [String, Number, Array],
		options: {
			type: Array,
			default() {
				return []
			},
		},
		prop: String,
		elAttrs: {
			type: Object,
			default() {
				return {}
			},
		},
	},
	render(h) {
		return (
			<el-select
				value={this.value}
				{
					...{
						attrs: {
							placeholder: '请选择',
							...this.elAttrs,
							size: 'small',
						},
						on: {
							input: val => this.$emit('input', this.prop, val),
						},
						props: { ...this.$attrs },
					}
				}
			>
			  {
				  this.options.map(option => (
					  <el-option key={option.value} value={option.value} label={option.label}></el-option>
				  ))
			  }
			</el-select>
		)
	},
}
</script>

<style lang="scss" scoped>

</style>
