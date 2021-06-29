<script>
import { DatePicker } from 'element-ui'

export default {
	name: 'FilterDate',
	components: {
		ElDatePicker: DatePicker,
	},
	props: {
		value: [String, Number, Array, Date],
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
			<el-date-picker
				value={this.value}
				{
					...{
						attrs: {
							'value-format': 'timestamp',
							clearable: false,
							editable: false,
							placeholder: '请选择',
							size: 'small',
							'range-separator': '~',
							...{
								...this.elAttrs,
							},
						},
						on: {
							input: val => this.$emit('input', this.prop, val),
						},
						props: { ...this.$attrs },
					}
				}
			>
			</el-date-picker>
		)
	},
}
</script>
