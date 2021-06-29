<script>
import { Radio } from 'element-ui'

export default {
	name: 'FilterRadio',
	components: {
		ElRadio: Radio,
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
		options: {
			type: Array,
			default: () => ([])
		},
	},
	render(h) {
		return (<div>
		{
			this.options.map(({ label = '', value = 0 }) => (
			<el-radio
				label={value}
				value={this.value}
				{...{
					attrs: {
						...this.elAttrs,
					},
					on: {
						input: (val) => this.$emit('input', this.prop, val),
					},
					props: { ...this.$attrs },
				}}
			>{label}</el-radio>
			))
		}
		</div>)
	},
}
</script>

<style lang="scss" scoped>
</style>
