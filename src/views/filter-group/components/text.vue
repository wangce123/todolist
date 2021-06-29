<script>

export default {
    name: 'FilterText',
    functional: true,
	render(h, { props }) {
		if (props.options && props.options.length) {
			props.value = (props.options.find(item => item.value === props.value) || { label: '' }).label
		}
		return (
            <div class="text">{props.value ? props.value : null}</div>
		)
	},
}
</script>

<style lang="scss" scoped>
.text{
	opacity: 0.6;
}
</style>
