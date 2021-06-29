<script>
import { Cascader } from 'element-ui'

export default {
	name: 'FilterCascader',
	components: {
		ElCascader: Cascader,
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
			  <el-cascader
               value={this.value}
               {
					...{
						attrs: {
							placeholder: '请选择',
							size: 'small',
							...this.elAttrs,
						},
						on: {
							input: val => this.$emit('input', this.prop, val),
						},
						props: { ...this.$attrs },
					}
				}
                options={this.options}></el-cascader>
		)
	},
}
</script>

<style lang="scss" scoped>

</style>
