<template>
	<view class="md">
		<el-alert class="alert" title="警告 切勿传输违反相关国家和国际法规的图片和个人隐私图片.本服务器记录上传者IP且为公开图床.中国网警正在审查你的数据和记录！" type="warning"
			center show-icon>
		</el-alert>
		<el-upload class="upload" drag action="" accept="image/*" :show-file-list="false" :http-request="doUpload"
			multiple>
			<i class="el-icon-upload"></i>
			<div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
			<div class="el-upload__tip" slot="tip">只能上传图片文件，且大小不超过100mb</div>
		</el-upload>
		<el-dialog title="上传列表" :visible.sync="centerDialogVisible" :close-on-click-modal="false" width="80%" center>
			<view class="list">
				<view @click="chooseImg(index)" class="box" :class="active==index?'active':''" v-for="(x,index) in list"
					:key="index">
					<image class="cover" v-if="x.status!=2" :src="x.url" mode="widthFix"></image>
					<view class="cell">
						<text class="err" v-if="x.status==2">{{x.errMsg}}</text>
						<text class="name">{{x.name}}</text>
					</view>

				</view>
			</view>
			<el-tabs v-model="activeName" v-if="active!=null">
				<el-tab-pane label="URL" name="first">
					<el-tag v-clipboard:copy="initText(1)" v-clipboard:success="onCopy" type="success">
						{{initText(1)}}
					</el-tag>
				</el-tab-pane>
				<el-tab-pane label="HTML" name="second">
					<el-tag type="success" v-clipboard:copy="initText(2)" v-clipboard:success="onCopy">
						{{initText(2)}}</el-tag>
				</el-tab-pane>
				<el-tab-pane label="BBCode" name="third">
					<el-tag type="success" v-clipboard:copy="initText(3)" v-clipboard:success="onCopy">{{initText(3)}}</el-tag>
				</el-tab-pane>
				<el-tab-pane label="MarkDown" name="fourth">
					<el-tag type="success" v-clipboard:copy="initText(4)" v-clipboard:success="onCopy">{{initText(4)}}</el-tag>
				</el-tab-pane>
				<el-tab-pane label="MarkDown with link" name="five">
					<el-tag type="success" v-clipboard:copy="initText(5)" v-clipboard:success="onCopy">{{initText(5)}}</el-tag>
				</el-tab-pane>
			</el-tabs>
		</el-dialog>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				list: [],
				centerDialogVisible: false,
				active: null,
				activeName: 'first'
			};
		},
		methods: {
			initText(val) {
				let type = val;
				val = this.list[this.active];
				let text;
				switch (type) {
					case 1:
						text = val.remoteUrl
						break;
					case 2:
						text = `<img src="${val.remoteUrl}" alt="${val.name}" title="${val.name}" />`
						break;
					case 3:
						text = `[img]${val.remoteUrl}[/img]`
						break;
					case 4:
						text = `![${val.name}](${val.remoteUrl})`
						break;
					case 5:
						text = `[![${val.name}](${val.remoteUrl})](${val.remoteUrl})`
						break;
				}

				return text;
			},
			onCopy(e) {
				this.$message({
					message: '链接已复制到剪贴板',
					type: 'success'
				});
			},
			async initUrl(file) {
				return new Promise((resolve, reject) => {
					let reader = new FileReader();
					let rs = reader.readAsArrayBuffer(file);
					let blob = null;
					reader.onload = (e) => {
						if (typeof e.target.result === 'object') {
							blob = new Blob([e.target.result])
						} else {
							blob = e.target.result
						}

						resolve(window.URL.createObjectURL(blob))
					}
				})
			},
			async doUpload(e) {
				this.centerDialogVisible = true
				let file = e.file
				if (file.type.indexOf('image') == 0) {
					let url = await this.initUrl(file)
					// 检查大小
					if (file.size < 1024 * 1024 * 100) {
						let t = {
							name: file.name,
							status: 0,
							url: url,
							remoteUrl: null,
							process: 0,
							errMsg: null
						}
						this.list.push(t);

						let result = await uniCloud.uploadFile({
							filePath: url,
							cloudPath: t.name,
							onUploadProgress: progressEvent => {
								// console.log(progressEvent);
								t.process = Math.round(
									(progressEvent.loaded * 100) / progressEvent.total
								);
							}
						});
						t.remoteUrl = result.fileID
						t.status = 1
						if (!this.active) {
							for (let index in this.list) {
								if (this.list[index].status == 1) {
									this.active = index
								}
							}
						}
						// console.log(result)
					} else {
						console.log('当前图片大小超过100Mb')
						this.list.push({
							name: file.name,
							status: 2,
							url: url,
							errMsg: "当前图片大小超过100Mb"
						})
					}
				} else {
					console.log('当前文件不是图片')
					// 不是图片
					this.list.push({
						name: file.name,
						status: 2,
						url: null,
						errMsg: "当前文件不是图片"
					})
				}
			},
			chooseImg(index) {
				console.log(this.list[index])
				if (this.list[index].status == 1) {
					this.active = index
				}
			}
		}
	}
</script>

<style lang="less">
	.md {
		width: 100vw;
		min-height: 100vh;
		box-sizing: border-box;
		background-color: #FFFFFF;
		display: flex;
		justify-content: center;
		align-items: center;
		flex-direction: column;
		background-size: cover;
		background-image: url('/static/bg.jpg');
	}

	.alert {
		position: fixed;
		top: 0;
		left: 0;
		z-index: 1;
	}

	.upload {
		margin: 0 auto;

		.el-upload__tip {
			color: #FFFFFF;
			text-align: center;
		}
	}

	.list {
		display: flex;
		flex-wrap: wrap;
		justify-content: flex-start;
		align-items: flex-start;

		.active {

			// border: 2px solid #409EFF;
			.cell {
				background-color: fade(#409EFF, 66%) !important;
			}
		}

		.box {
			margin-bottom: 32upx;
			margin-right: 32upx;
			width: 200px;
			height: 200px;
			overflow: hidden;
			display: flex;
			justify-content: center;
			align-items: center;
			position: relative;
			cursor: pointer;

			.cover {
				width: 200px;
			}

			.cell {
				width: 200px;
				height: 200px;
				background-color: fade(black, 66%);
				position: absolute;
				left: 0;
				bottom: 0;
				display: flex;
				justify-content: center;
				align-items: center;
				flex-direction: column;

				text {
					text-align: center;
					font-size: 12px;
					color: #FFFFFF;
				}

				.err {
					margin-bottom: 16upx;
				}
			}
		}
	}
</style>
