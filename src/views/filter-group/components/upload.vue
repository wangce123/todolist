<script>
import { Upload, Button } from 'element-ui'
import _ from 'lodash'

export default {
	name: 'FilterUpload',
	components: {
		ElUpload: Upload,
		ElButton: Button,
	},
	props: {
		value: Array,
		prop: String,
		elAttrs: {
			type: Object,
			default() {
				return {}
			},
		},
    },
    methods: {
      handleExceed(files, fileList) {
			this.$message.warning(`当前限制选择 ${this.elAttrs.limit} 个文件，本次选择了 ${files.length} 个文件，共选择了 ${files.length + fileList.length} 个文件`)
      },
      handleUpload({ type, size }) {
		  let isSupportFormat = true
		  let isSupportSize = true
		  if (this.$attrs.format) {
			if (_.isArray(this.$attrs.format)) {
				isSupportFormat = this.$attrs.format.some(item => type.indexOf(item) > -1)
			} else {
				isSupportFormat = type.indexOf(this.$attrs.format) > -1
			}
			if (!isSupportFormat) {
				this.$message({
					type: 'error',
					message: '请上传正确格式文件',
				})
			} 
		 	}
			if (this.$attrs.size) {
				if (size >= this.$attrs.size * 1024 * 1024) {
					isSupportSize = false
					this.$message({
						type: 'error',
						message: `单个文件限制${this.$attrs.size}MB`,
					})
				}
			}
		return isSupportFormat && isSupportSize
	  },
	},
	computed: {
		getCover() {
			let cover = null
			switch (this.$attrs.cover) {
				case 'avatar':
					if (this.value && this.value.length) {
						let src = ''
						if (this.value[0].url) {
							src = this.value[0].url
						} else {
							if (this.value[0].raw) {
								src = URL.createObjectURL(this.value[0].raw)
							} else {
								cover = <i class="el-icon-plus"></i>
								break;
							}
						}
						cover = <img class="el-icon-plus" src={src} />
					} else {
						cover = <i class="el-icon-plus"></i>
					}
					break;
			
				default:
					cover = <el-button size="mini" type="primary">点击上传</el-button>
					break;
			}
			return cover
		},
	},
	render(h) {
		return (
			<el-upload
				file-list={this.value}
				{...{
					attrs: {
                        'on-success': async (res, file,val) => {
							if (this.$attrs.cover === 'avatar') {
								this.$emit('input', this.prop, [val[val.length - 1]])
							} else {
								this.$emit('input', this.prop, val)
							}
							/**
							 * 应该是此版本element的坑,其他版本数组校验的好好的这个版本第二个问题了
							*/
							this.$emit('validateCurField', this.prop)
						},
                        'on-remove': (res, file,val) => this.$emit('input', this.prop, val),
                        'on-exceed': this.handleExceed,
                         multiple: false,
                         'before-upload': this.handleUpload,
						...this.elAttrs,
					},
					props: { ...this.$attrs },
				}}
			>{this.getCover}</el-upload>
		)
	},
}
</script>

<style lang="scss" scoped>
.el-icon-plus{
	font-size: 28px;
    color: #8c939d;
    width: 178px;
    height: 178px;
    line-height: 178px;
    text-align: center;
	background-color: #f0f2f5;
}
</style>
<style lang="scss">
.el-upload--text{
	padding-top: 15px;
}
</style>