<script>
import { Input } from 'element-ui'

export default {
	name: 'FilterInput',
	components: {
		ElInput: Input,
	},
	props: {
		value: [String, Number],
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
			<el-input
				value={this.value}
				{...{
					attrs: {
						placeholder: '请输入',
						size: 'small',
						...this.elAttrs,
					},
					on: {
						input: (val) => this.$emit('input', this.prop, val),
						blur: (e) => {
							this.$emit('input', this.prop, e.target.value.trim())
							this.elAttrs.Fn && this.elAttrs.Fn(e.target.value.trim())
						},
					},
					props: { ...this.$attrs },
				}}
			></el-input>
		)
	},
}
</script>

<style lang="scss" scoped>
</style>
