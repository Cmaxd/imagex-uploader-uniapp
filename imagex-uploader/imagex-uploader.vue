<template>
	<view class="imagex-uploader-container">
		<view class="imagex-uploader-files">
			<view v-for="(item, index) in fileList" :key="index">
				<view class="imagex-uploader-file"
					:class="[item.uploadPercent < 100 ? 'imagex-uploader-file-status' : '']"
					@click="previewImage(index)">
					<image class="imagex-uploader-img" :style="previewStyle" :src="item.path" mode="aspectFill" />
					<view class="imagex-img-removeicon right" @click.stop="removeImage(index)" v-show="enableMove">×
					</view>
					<view class="imagex-loader-filecontent" v-if="item.uploadPercent < 100">{{ item.uploadPercent }}%
					</view>
				</view>
			</view>
			<view v-show="fileList.length < uploadLimit" hover-class="imagex-uploader-hover"
				class="imagex-uploader-inputbox" @click="chooseImage" :style="uploadStyle">
				<view><text class="iconfont" style="color: #666;"></text></view>
			</view>
		</view>
	</view>
</template>

<script>
	import ImagexUploader from './uploader.js';
	export default {
		data() {
			return {
				tempFilePaths: [],
				upload_cache_list: [],
				fileList: []
			};
		},
		name: 'imagex-uploader',
		props: {
			// 服务ID
			serviceId: {
				type: String,
				default: ''
			},
			// stsToken
			stsToken: {
				type: Object,
				default: function() {
					return {};
				}
			},
			// 上传地区
			region: {
				type: String,
				default: 'cn',
			},
			// 当前上传用户，用于辅助定位问题，非必填
			userId: {
				type: String,
				default: '',
			},
			// appId为必填项，请填写在火山引擎创建的appId，用于辅助定位问题
			appId: {
				type: Number,
				default: undefined,
			},
			// 预览图样式
			previewStyle: {
				type: Object,
				default: function() {
					return {
						width: '162rpx',
						height: '162rpx'
					}
				}
			},
			// 上传样式
			uploadStyle: {
				type: Object,
				default: function() {
					return {
						width: '162rpx',
						height: '162rpx'
					}
				}
			},
			// 上传数量限制
			uploadLimit: {
				type: Number,
				default: 9
			},
			// 是否自动上传
			autoUpload: {
				type: Boolean,
				default: true
			},
			// 是否显示移除
			enableMove: {
				type: Boolean,
				default: true
			},
			// 服务器预览图片
			defaultImageList: {
				type: Array,
				default: () => {
					return [];
				}
			},
		},
		async created() {
			let _self = this;
			this.fileList = this.fileList.concat(this.defaultImageList);
			this.defaultImageList.map(item => {
				this.upload_cache_list.push(item.path);
			});
			this.emit();
		},
		methods: {
			uploadImage(fileInfo) {
				let _self = this;
				const promises = fileInfo.tempFiles.map(function(file, index) {
					return promisify(upload)({
						info: {
							tempFile: file,
							tempFilePath: fileInfo.tempFilePaths[index],
						},
						_self: _self
					});
				});

				uni.showLoading({
					title: `正在上传...`
				});

				Promise.all(promises)
					.then(function(data) {
						uni.hideLoading();
						_self.upload_cache_list.push(...data);
						_self.emit();
					})
					.catch(function(res) {
						uni.hideLoading();
					});
			},
			chooseImage() {
				let _self = this;
				uni.chooseImage({
					sizeType: ['compressed', 'original'],
					sourceType: ['album', 'camera'],
					success: function(res) {
						for (let i = 0, len = res.tempFiles.length; i < len; i++) {
							res.tempFiles[i]['uploadPercent'] = 0;
							_self.fileList.push(res.tempFiles[i]);
						}
						_self.tempFilePaths = res.tempFilePaths;
						_self.upload(res, _self.autoUpload);
					},
					fail: function(err) {
						console.log(err);
					}
				});
			},
			async upload(fileInfo, autoUpload) {
				let _self = this;
				autoUpload ? await _self.uploadImage(fileInfo) : console.warn(
					`传输参数:this.$refs.xx.upload(true)才可上传,默认false`);
			},
			previewImage(idx) {
				let _self = this;
				let preview = [];
				for (let i = 0, len = _self.fileList.length; i < len; i++) {
					// step3.这里修改服务器返回字段 ！！！
					preview.push(_self.fileList[i].path);
				}
				uni.previewImage({
					current: idx,
					urls: preview
				});
			},
			removeImage(idx) {
				let _self = this;
				_self.fileList.splice(idx, 1);
				_self.upload_cache_list.splice(idx, 1);
				_self.emit();
			},
			emit() {
				let _self = this;
				_self.$emit('change', _self.upload_cache_list);
			}
		}
	};

	const promisify = api => {
		return function(options, ...params) {
			return new Promise(function(resolve, reject) {
				api(
					Object.assign({}, options, {
						success: resolve,
						fail: reject
					}),
					...params
				);
			});
		};
	};

	const upload = function(options) {
		let info = options.info;
		let _self = options._self;
		const imagexUploader = new ImagexUploader({
			region: _self.region,
			appId: _self.appId,
			userId: _self.userId,
			stsToken: _self.stsToken,
			type: 'image',
			imageConfig: {
				serviceId: _self.serviceId,
			},
		});

		if (imagexUploader) {
			try {
				imagexUploader.start({
					path: info.tempFilePath,
					size: info.tempFile.size,
					type: 'image',
					stsToken: _self.stsToken,
					success: function(data) {
						if (options.success) {
							options.success(data);
						}
					},
					fail: function(data) {
						if (options.fail) {
							options.fail(data);
						}
					},
					progress: function(data) {
						let targetIndex = _self.fileList.findIndex(file => file.path === info.tempFilePath);
						if (targetIndex >= 0) {
							_self.fileList[targetIndex]['uploadPercent'] = data.progress;
							_self.$set(_self.fileList, targetIndex, _self.fileList[targetIndex]);
						}
					}
				});
			} catch (e) {
				console.log('error', e)
			}
		}
	};
</script>

<style lang="scss">

	.iconfont {
		font-size: 16px;
		font-style: normal;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
	}

	.imagex-uploader-container {
		padding: 20rpx;
	}

	.imagex-uploader-files {
		display: flex;
		flex-wrap: wrap;
	}

	.imagex-uploader-file {
		position: relative;
		margin-right: 16rpx;
		margin-bottom: 16rpx;
	}

	.imagex-uploader-file-status:after {
		content: ' ';
		position: absolute;
		top: 0;
		right: 0;
		bottom: 0;
		left: 0;
		background-color: rgba(0, 0, 0, 0.5);
	}

	.imagex-uploader-img {
		display: block;
	}

	.imagex-uploader-input {
		position: absolute;
		z-index: 1;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		opacity: 0;
	}

	.imagex-uploader-inputbox {
		position: relative;
		margin-bottom: 16rpx;
		box-sizing: border-box;
		background-color: #ededed;
		display: flex;
		flex-wrap: wrap;
		align-items: center;
		justify-content: center;
		background-image: url('data:image/svg+xml;base64,PHN2ZyB0PSIxNjE3MDM4NjEwOTgzIiBjbGFzcz0iaWNvbiIgdmlld0JveD0iMCAwIDEwMjQgMTAyNCIgdmVyc2lvbj0iMS4xIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHAtaWQ9IjEzMjk4IiB3aWR0aD0iMzIiIGhlaWdodD0iMzIiPjxwYXRoIGQ9Ik0xMjggNTU0LjQ5Nmg3NjhjMjMuNTUyIDAgNDIuNDk2LTE4Ljk0NCA0Mi40OTYtNDIuNDk2cy0xOC45NDQtNDIuNDk2LTQyLjQ5Ni00Mi40OTZIMTI4Yy0yMy41NTIgMC00Mi40OTYgMTguOTQ0LTQyLjQ5NiA0Mi40OTZzMTguOTQ0IDQyLjQ5NiA0Mi40OTYgNDIuNDk2ek00NjkuNTA0IDEyOHY3NjhjMCAyMy41NTIgMTguOTQ0IDQyLjQ5NiA0Mi40OTYgNDIuNDk2czQyLjQ5Ni0xOC45NDQgNDIuNDk2LTQyLjQ5NlYxMjhjMC0yMy41NTItMTguOTQ0LTQyLjQ5Ni00Mi40OTYtNDIuNDk2cy00Mi40OTYgMTguOTQ0LTQyLjQ5NiA0Mi40OTZ6IiBwLWlkPSIxMzI5OSIgZmlsbD0iI2JmYmZiZiI+PC9wYXRoPjwvc3ZnPg==');
		background-repeat: no-repeat;
		background-position: center;
	}

	.imagex-img-removeicon {
		position: absolute;
		color: #fff;
		width: 40upx;
		height: 40upx;
		line-height: 40upx;
		z-index: 2;
		text-align: center;
		background-color: #e54d42;

		&.right {
			top: 0;
			right: 0;
		}
	}

	.imagex-loader-filecontent {
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		color: #fff;
		z-index: 9;
	}

	.imagex-uploader-file:nth-child(4n + 0) {
		margin-right: 0;
	}

	.imagex-uploader-inputbox>view {
		text-align: center;
	}

	.imagex-uploader-hover {
		box-shadow: 0 0 0 #e5e5e5;
		background: #e5e5e5;
	}
</style>
