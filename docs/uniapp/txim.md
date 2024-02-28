### uni-app兼容安卓 ios 对接腾讯im聊天(代码不完善)
 
#### 1. 使用含ui继承方案(https://cloud.tencent.com/document/product/269/79111)
- 详细步骤按照 https://cloud.tencent.com/document/product/269/64506 执行

#### 2. 解决收到消息滚动条问题，获取历史数据闪烁问题
- 将聊天元素翻转180度，然后里面的内容翻转180(历史数据问题)
- 接收消息自动滚动到底部(收消息问题)

<!-- tabs:start -->

#### **TUIChat.scss**
``` scss
.TUIChat {
  width: 100%;
    /* #ifdef APP-PLUS */
  	height: calc(100vh - var(--status-bar-height) - 44px );
  /* #endif */

  /* #ifdef H5 */
  	height: calc(100vh - 88upx);
  /* #endif */
  display: flex;
  flex-direction: column;
   &-header {
    padding: 20px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    h1 {
      font-family: PingFangSC-Medium;
      font-weight: 500;
      font-size: 16px;
      color: #000000;
      letter-spacing: 0;
      line-height: 24px;
    }
  }
	&-container {
		width: 100%;
		overflow: auto;
		transform: rotate(180deg);
	}
```
#### **TUIChat/index.vue**
``` vue
// 首次获取聊天记录数据
const getList = () => {
	TUIServer.getMessageList({
		conversationID: timStore.conversation.conversationID,
		count: 15,
		beginIndex: data.messageList.length
	}).then(res => {
		console.log('聊天记录', res)
		data.isCompleted = res.isCompleted
		let list = res.messageList
		if (list.length > 0) {
			// 获取老时间
			data.oldMessageTime = list[0].time;
			// 不展示已删除消息
			let filterList = list.filter((item: any) => {
				return !item.isDeleted;
			});
			data.messageList.push(...filterList)
			// 第一次进页面滚动到底部
			nextTick(() => {
				handleShowTime();
			})
		}
	}).catch(err => {
		// 没有在群里
		console.log(err)

	})
}

// 监听数据变更 （包括收到的消息，和自己发送的消息）
watch(() => timStore.sendNowMessage, (n) => {
	try {
		if (n.payload.operationType == 255) {
			if (n.payload.userDefinedField.indexOf('邀请进群') != -1) {
				inGroup.value = true
			}
			if (n.payload.userDefinedField.indexOf('踢出群聊') != -1) {
				inGroup.value = false
			}
		}
	} catch (err) {

	}
	n.isAnonymousValue = false
	let pick = groupUserList.find((jtem) => {
		return n.from == jtem.userID
	})
	if (pick) {
		let isAnonymousValue = pick.memberCustomField.find((item) => {
			return item.key == 'IsAnonymous' && item.value == 'True'
		})

		if (pick) n.isAnonymousValue = true
	}
	data.messageList.push(n)
	if (!pageHide) isRead()
	// if (scrollBottom) initScrollBar()
})

```
#### **TUICore/server/chat/index.ts**
我使用了本地数据库sqlite可以不用管，只需要知道这里是获取其他用户是否已读的回调
``` ts
    private async handleMessageReadByPeer(event : any) {
		// 获取已读消息的回调
		store.commit("timStore/setReadList", event.data)
		// 更新数据库中的表
		// #ifdef APP-PLUS
		event.data.forEach(item => {
			sqlite.insertIMMessage(item)
		})
		// #endif
	}
```

#### **TUIKit/TUICore/server/chat/index.ts**
我使用了本地数据库sqlite可以不用管，只需要知道这里是获取其他用户是否已读的回调
``` ts
// 接受消息处
    private async handleMessageReceived(event : any) {
		for (let i = 0; i < event.data.length; i++) {
			if (event.data[i].conversationID === this.store.conversationID) {
				// console.log(event.data[i])
				// uni.$TUIKit.TUIConversationServer.setMessageRead(event.data[i].conversationID);
				this.sendMessageLocal(event.data[i])
			}
			if (event.data[i].conversationID == '@TIM#SYSTEM') {
				event.data[i].conversationID = 'GROUP' + event.data[i].to
				this.sendMessageLocal(event.data[i])
			}


		}
	}

    /**
	 * 保存本地信息
	 *
	 * @param {any} message 发送的消息
	 * @returns {Promise}
	 */
	public sendMessageLocal(message : any, isCommit = true) : Promise<any> {
		return new Promise<void>(async (resolve, reject) => {
			try {
				// 将发送的数据保存，便于监听
				if (isCommit) store.commit("timStore/sendMessage", message);
				// 并且是app保存该条数据到本地数据库
				// console.log(message)
				// #ifdef APP-PLUS
				sqlite.insertUser(message)
				sqlite.insertIMMessage(message)
				// #endif

			} catch (error) {

			}
		});
	}
```
<!-- tabs:end -->



#### 3. 安卓和ios聊天input框高度问题解决
- 目前是使用每次计算软键盘高度来修改聊天窗口的高度
- 使用inputpadding来顶聊天高度（未实现）待处理中

<!-- tabs:start -->

#### **TUIChatInput**

``` vue 
<view class="TUIChat-container" @tap="closeBottom" @touchstart="touchstart" @touchmove="touchmove"
			:style="{ height: stockListHeight }">
<!-- 组件 -->
<TUIChatInput ref="TUIChatInputRef" :text="text" :conversationData="conversation" @openUtil="openUtil">
			</TUIChatInput>
// 软键盘弹起
const openUtil = (e) => {
	// setScrollHeight(e)
	scrollBottom = true
	setScrollHeight(e);
};
const setScrollHeight = (descHeight = "0px") => {
	// #ifdef APP-PLUS
	stockListHeight.value = `calc(100vh - 116upx - 44px  - ${statusBar}px - ${descHeight} )`;
	// #endif
	// #ifdef H5
	stockListHeight.value = `calc(100vh - 116upx - 44px -  ${descHeight} )`;
	// #endif
	// 要发送消息情况下，瞬间滚动到底部  并且
	scrollAnimation.value = false
	initScrollBar();
	// stockListHeight.value = `calc(100vh - 44px - ${statusBar}px )`
};

// 初始化到底部
const initScrollBar = () => {
	nextTick(() => {
		data.scrollTop = data.scrollTop == 0 ? -1 : 0
		scrollViewId.value = ""
	})
};


```

#### **TUIChatInput内部**
```
<template>
	<view class="">
		<view class="TUI-message-input-container">
			<view class="TUI-message-input">
				<!-- #ifdef H5 -->
				<image class="TUI-icon" v-if="!isAudio" @tap.stop="handleSwitchAudio"
					src="../../../../assets/icon/keyboard.png"></image>
				<image class="TUI-icon" v-else @tap.stop="handleSwitchAudio"
					src="../../../../assets/icon/voice-circle.png">
				</image>
				<!-- #endif -->
				<!-- #ifndef H5 -->
				<image class="TUI-icon" v-if="!isAudio" @tap.stop="handleSwitchAudio"
					src="../../../../assets/icon/keyboard.png"></image>
				<image class="TUI-icon" v-else @tap.stop="handleSwitchAudio"
					src="../../../../assets/icon/voice-circle.png">
				</image>
				<!-- #endif -->
				<view v-if="!isAudio" class="TUI-message-input-main">
					<input class="TUI-message-input-area emojifont" :adjust-position="false" cursor-spacing="20"
						v-model="inputText" confirm-type="send" confirm-hold="true" @confirm="handleSendTextMessage"
						maxlength="140" type="text" placeholder-class="input-placeholder" placeholder="请输入聊天内容"
						@input="textInput" />
				</view>
				<view v-else class="TUI-message-input-main">
					<AudioMessage></AudioMessage>
				</view>
				<view class="TUI-message-input-functions" hover-class="none">
					<image class="TUI-icon" @tap.stop="handlechange('emoji')" src="../../../../assets/icon/emoji.svg">
					</image>
					<view @tap.stop="handlechange('extension')">
						<image class="TUI-icon" src="../../../../assets/icon/more.svg"></image>
					</view>
				</view>
			</view>
			<view>
				<view class="TUI-Emoji-area" v-if="displayFlag === 'emoji'">
					<Face :show="displayFlag === 'emoji'" @send="handleSend" @handleSendEmoji="handleSendTextMessage">
					</Face>
				</view>
				<view v-if="displayFlag === 'extension'" class="TUI-Extensions">
					<!-- TODO: 这里功能还没实现
				<!       <camera device-position="back" flash="off" binderror="error" style="width: 100%; height: 300px;"></camera>-->
					<view class="TUI-Extension-slot" @tap.prevent.stop="handleSendImageMessage('camera')">
						<image class="TUI-Extension-icon" src="../../../../assets/icon/take-photo.svg"></image>
						<view class="TUI-Extension-slot-name">拍照</view>
					</view>
					<view class="TUI-Extension-slot" @tap.prevent.stop="handleSendImageMessage('album')">
						<image class="TUI-Extension-icon" src="../../../../assets/icon/send-img.svg"></image>
						<view class="TUI-Extension-slot-name">图片</view>
					</view>
					<view class="TUI-Extension-slot" @tap.prevent.stop="handleSendVideoMessage('album')">
						<image class="TUI-Extension-icon" src="../../../../assets/icon/take-video.svg"></image>
						<view class="TUI-Extension-slot-name">视频</view>
					</view>
					<view class="TUI-Extension-slot" @tap.prevent.stop="handleSendVideoMessage('camera')">
						<image class="TUI-Extension-icon" src="../../../../assets/icon/take-photo.svg"></image>
						<view class="TUI-Extension-slot-name">录像</view>
					</view>
					<!-- #ifndef H5 -->
					<!-- <view class="TUI-Extension-slot" @tap.stop="handleCalling(1)">
					<image class="TUI-Extension-icon" src="../../../../assets/icon/audio-calling.svg"></image>
					<view class="TUI-Extension-slot-name">语音通话</view>
				</view>
				<view class="TUI-Extension-slot" @tap.stop="handleCalling(2)">
					<image class="TUI-Extension-icon" src="../../../../assets/icon/take-video.svg"></image>
					<view class="TUI-Extension-slot-name">视频通话</view>
				</view> -->
					<!-- #endif -->
				</view>
			</view>
		</view>

		<uni-popup ref="popup" type="bottom">
			<view class="popBox">
				<view class="flex-center-bt  btns">
					<view class="sureBtn" @tap="closePop">
						取消
					</view>
					<view class="sureBtn" :class="{isOK:isOK}" @tap="popSure">
						确定
					</view>
				</view>
				<uni-search-bar radius="5" placeholder="搜索" clearButton="auto" cancelButton="none" @input="search" />
				<view class="contents">
					<uni-indexed-list :options="popList" :show-select="true" @click="bindClick"
						:isAnonymou="isAnonymous" />
				</view>
			</view>
		</uni-popup>
	</view>


</template>

<script lang="ts">
	import {
		defineComponent,
		watchEffect,
		reactive,
		toRefs,
		onMounted,
		computed,
		shallowRef,
		onUnmounted,
		ref,
		watch
	} from "vue";
	import Face from "./message/face.vue";
	import AudioMessage from "./message/audio.vue";
	import store from "../../../../TUICore/store";
	import { onReachBottom, onShow, onHide } from "@dcloudio/uni-app";
	import uniIndexedList from './uni-indexed-list/uni-indexed-list.vue'
	import {
		convertList
	} from '@/utils/pinyin.js'

	import {
		getGroupInfo,
	} from '@/Apis/im.js'

	const TUIChatInput = defineComponent({
		components: {
			Face,
			AudioMessage,
			uniIndexedList
		},
		props: {
			text: {
				type: String,
				default: () => {
					return "";
				},
			},
			conversationData: {
				type: Object,
				default: () => {
					return "";
				},
			},
		},
		setup(props, context) {
			const TUIServer : any = uni.$TUIKit.TUIChatServer;
			const data = reactive({
				firstSendMessage: true,
				inputText: "",
				extensionArea: false,
				sendMessageBtn: false,
				displayFlag: "", //控制显示的是那个
				isOpenDisplay: false, //是否已经打开了下方工具栏(包括 软键盘，表情，图片)
				isAudio: false,
				displayServiceEvaluation: false,
				displayCommonWords: false,
				displayOrderList: false,

				conversation: computed(() => store.state.timStore.conversation),
			});

			watchEffect(() => {
				data.inputText = props.text;
			});

			const handleSwitchAudio = () => {
				data.isAudio = !data.isAudio;
				if (data.isAudio) {
					close()
				}
			};

			// 打开下方类型
			const handlechange = (type) => {
				if (data.displayFlag == "softWare") {
					data.displayFlag = type;
				} else {
					data.displayFlag = data.displayFlag === type ? "" : type;
					if (data.displayFlag) data.isAudio = false
					// 跳转到底部
					let h = data.displayFlag ? uni.upx2px(450) : 0

					context.emit("openUtil", h + "px");

				}

			};

			// 发送消息
			const handleSendTextMessage = (e : any) => {
				if (data.inputText.trimEnd()) {
					let arr = []
					// 并且要判断字符名字一样
					atPersonList.forEach((item => {
						if (data.inputText.indexOf(item.nickname) != -1) {
							arr.push(item)
						}
					}))

					if (arr.length) {
						let atUserList = arr.map((item) => item.uid)
						console.log(atUserList)
						uni.$TUIKit.TUIChatServer.sendTextAtMessage({
							text: JSON.parse(JSON.stringify(data.inputText)),
							atUserList: atUserList
						})
					} else {
						uni.$TUIKit.TUIChatServer.sendTextMessage(
							JSON.parse(JSON.stringify(data.inputText))
						);
					}
				}
				data.inputText = "";
			};

			// 处理需要合并的数据
			const handleSend = (emo : any) => {
				data.inputText += emo;
				// inputEle.value.focus();
			};

			const handleSendImageMessage = (type : any) => {
				uni.chooseImage({
					sourceType: [type],
					count: 1,
					success: (res) => {
						uni.getImageInfo({
							src: res.tempFilePaths[0],
							success(image) {
								TUIServer.sendImageMessage(res, image);
							},
						});
					},
				});
			};

			const handleSendVideoMessage = (type : any) => {
				uni.chooseVideo({
					sourceType: [type],
					// 来源相册或者拍摄
					maxDuration: 60,
					// 设置最长时间60s
					camera: "back",
					// 后置摄像头
					success: (res) => {
						if (res) {
							TUIServer.sendVideoMessage(res);
						}
					},
				});
			};

			const handleCalling = (value : any) => {
				// todo 目前支持单聊
				if (data.conversation.type === "GROUP") {
					uni.showToast({
						title: "群聊暂不支持",
						icon: "none",
					});
					return;
				}
				const { userID } = data.conversation.userProfile;

				// #ifdef APP-PLUS
				if (typeof uni.$TUICallKit === "undefined") {
					console.error(
						"请集成 TUICallKit 插件，使用真机运行并且自定义基座调试，请参考官网文档：https://cloud.tencent.com/document/product/269/83857"
					);
					uni.showToast({
						title: "请集成 TUICallKit 插件哦 ～ ",
						icon: "none",
						duration: 3000,
					});
				} else {
					uni.$TUICallKit.call(
						{
							userID: userID,
							callMediaType: value,
						},
						(res) => {
							console.log(JSON.stringify(res));
						}
					);
				}
				// #endif
				// #ifdef MP-WEIXIN
				if (
					typeof uni.$TUICallKit !== "undefined" &&
					uni.$TUICallKit.value !== null
				) {
					uni.$TUICallKit.value.call({
						userID: userID,
						type: value,
					});
				} else {
					console.error(
						"请集成 TUICallKit 插件哦～，请参考官网文档：https://cloud.tencent.com/document/product/269/83858"
					);
					uni.showToast({
						title: "请集成 TUICallKit 插件哦 ～ ",
						icon: "none",
						duration: 3000,
					});
				}
				// #endif
				// #ifdef H5
				uni.showToast({
					title: "暂不支持",
					icon: "none",
				});
				// #endif
			};

			const close = (isEmit = true) => {
				data.displayFlag = "";
				context.emit("openUtil");
			};

			const resetDisplayFlay = () => {
				data.displayFlag = "";
			}

			let firstTime = false;
			const showBottom = computed(() => {
				if (data.displayFlag == "emoji" || data.displayFlag == "extension") {
					firstTime = true;
					return "aniDispaly";
				} else {
					if (firstTime) return "disDisplay";
					return "";
				}
			});
			onHide(() => {
				firstTime = false;
				//   close();
				// 业务逻辑
			});
			// #ifdef APP-PLUS
			uni.onKeyboardHeightChange((res) => {
				context.emit("openUtil", res.height + "px");
				if (res.height != 0) {
					// 打开了软键盘
					data.displayFlag = 'softWare'
					// handlechange("softWare");
				}
			});
			// #endif

			const popup = ref()

			const popList = reactive([])
			const isAnonymous = ref(false)
			let oldList = []
			let select = reactive([])
			let atPersonList = []
			// 光标位置
			const inputCursor = ref(0)

			const isOK = computed(() => {
				return select.length ? true : false
			})
			const search = (e) => {
				select.length = 0
				select = []
				let nowList = oldList.filter((item => {
					return item.nickname.indexOf(e) != -1
				}))
				let data = convertList(nowList)
				if (!isAnonymous.value) {
					data.unshift({ letter: '*', data: [{ photo: '', nickname: '所有人' }] })
				}
				popList.length = 0
				popList.push(...data)
			}

			const textInput = (e) => {
				if (store.state.timStore.conversationID.indexOf('GROUP') == -1) return

				if (!isAdd) return
				let { cursor, value } = e.detail

				let singleText = value.charAt(cursor - 1)
				// 打开群组聊天
				if (singleText == '@') {
					uni.hideKeyboard()
					select.length = 0
					popup.value.open()
					// let gid = store.state.timStore.conversationID.split('GROUP')[1]
					// getGroupInfo({
					// 	gid: gid
					// }).then(res => {
					// 	let id = uni.getStorageSync('id')
					// 	let list = res.groupMember.filter((item) => {
					// 		return item.userID != id
					// 	})
					// 	oldList = res.groupMember
					// 	let list2 = convertList(list)
					// 	// isAnonymous.value = res.myGroupMemberInfo.isAnonymous
					// 	if (!isAnonymous.value) {
					// 		list2.unshift({ letter: '*', data: [{ photo: '', nickname: '所有人', uid: uni.$TIM.TYPES.MSG_AT_ALL }] })
					// 	}
					// 	popList.length = 0
					// 	popList.push(...list2)
					// })

					let promise = uni.$TUIKit.getGroupMemberList({ groupID: store.state.timStore.conversationID.split('GROUP')[1], count: 999, offset: 0 }); // 从 0 开始拉取 15 个群成员
					promise.then(function (imResponse) {
						let id = uni.getStorageSync('id')
						// 获取除自己之外的群成员
						let list = imResponse.data.memberList.filter((item) => {
							item.nickname = item.nick
							item.uid = item.userID
							// item.uid=item.
							return item.userID != id
						})

						let list2 = convertList(list)
						// isAnonymous.value = res.myGroupMemberInfo.isAnonymous
						list2.unshift({ letter: '*', data: [{ photo: '', nickname: '所有人', uid: uni.$TIM.TYPES.MSG_AT_ALL }] })
						popList.length = 0
						popList.push(...list2)
						console.log(popList)



					}).catch(function (imError) {
						console.warn('getGroupMemberList error:', imError);
					});

				}
			}

			let isAdd = true
			watch(() => data.inputText, (newVal, oldVal) => {
				if (newVal.length >= oldVal.length) isAdd = true
				else isAdd = false

				if (!newVal) isAdd = true
				delAtUser(newVal, oldVal)
			})

			// 删除@的用户
			const delAtUser = (newVal, oldVal) => {
				if (oldVal == '') return
				//获取光标位置
				let cursorPosition = 0
				for (let i = 0; i < oldVal?.length; i++) {
					if (newVal[i] !== oldVal[i]) {
						cursorPosition = i
						break
					}
				}
				//光标前面的文字
				const cursorFirstText = oldVal.slice(0, cursorPosition + 1)
				//获取@符号位置
				const atIndex = cursorFirstText.lastIndexOf('@')
				// 获取@符号到光标的位置
				const atToEndstr = cursorFirstText.slice(atIndex, cursorPosition + 1)


				// 正在@的用户
				for (let i = 0; i < atPersonList.length; i++) {
					const text = atPersonList[i].nickname
					// 匹配@+用户与@符号后面的文字相等则处理
					if ('@' + text + ' ' == atToEndstr) {
						// 获取@前面的位置，然后没有则空
						const firstText = atIndex - 1 <= 0 ? '' : newVal.slice(0, atIndex)
						// 从@符号到光标的文字后面截取
						const lastText = oldVal.slice(atIndex + atToEndstr?.length)
						// 重新设置input框值
						data.inputText = firstText + lastText
						atPersonList.splice(i, 1)
						// 设置光标位置
						setTimeout(() => {
							// inputCursor.value = atIndex
						}, 200)

					}
				}
			}

			const bindClick = (e) => {
				select.length = 0
				select.push(...e.select)
			}

			const closePop = () => {
				popup.value.close()
			}

			// 弹窗确定  选择@的人
			const popSure = () => {
				atPersonList = select
				atPersonList.forEach((item, index) => {
					if (index == 0) {
						data.inputText += `${item.nickname} `
					} else {
						data.inputText += `@${item.nickname} `
					}
				})
				// data.inputText
				closePop()

			}

			return {
				...toRefs(data),
				handleSendImageMessage,
				handleSendTextMessage,
				handleSendVideoMessage,
				handleSend,
				handleSwitchAudio,
				handleCalling,
				close,
				showBottom,
				handlechange,
				resetDisplayFlay,
				textInput,
				popup,
				popList,
				bindClick,
				isAnonymous, search, isOK, closePop, popSure, inputCursor
			};
		},
	});
	export default TUIChatInput;
</script>
<style lang="scss" scoped>
	.TUI-message-input-container {
		// padding-bottom: 20rpx;
		border-top: 1rpx solid #dddddd;
		// height: 116rpx;
		padding-bottom: constant(safe-area-inset-bottom);
		/*兼容 IOS<11.2*/
		padding-bottom: env(safe-area-inset-bottom);
		/*兼容 IOS>11.2*/
	}

	.TUI-message-input {
		display: flex;
		padding: 16rpx;
		background-color: white;
		width: 100vw;
		height: 116rpx;
		align-items: center;
		flex-direction: row;
	}

	.TUI-commom-function {
		display: flex;
		flex-wrap: nowrap;
		width: 750rpx;
		height: 106rpx;
		background-color: #f1f1f1;
		align-items: center;
	}

	.TUI-commom-function-item {
		display: flex;
		width: 136rpx;
		justify-content: center;
		align-items: center;
		font-size: 24rpx;
		color: #ffffff;
		height: 48rpx;
		margin-left: 16rpx;
		border-radius: 24rpx;
		background-color: #00c8dc;
	}

	.TUI-commom-function-item:first-child {
		margin-left: 48rpx;
	}

	.TUI-message-input-functions {
		display: flex;
		flex-direction: row;
	}

	.TUI-message-input-main {
		background-color: #F4F5F6FF;
		flex: 1;
		height: 66rpx;
		margin: 0 10rpx;
		padding: 0 5rpx;
		border-radius: 5rpx;
		display: flex;
		align-items: center;
	}

	.TUI-message-input-area {
		width: 100%;
		height: 100%;
		background: #F4F5F6;
		border-radius: 16rpx;
		padding-left: 20rpx;
	}

	.TUI-icon {
		width: 56rpx;
		height: 56rpx;
		margin: 0 16rpx;
	}

	.TUI-Extensions {
		display: flex;
		flex-wrap: wrap;
		width: 100vw;
		height: 450rpx;
		margin-left: 14rpx;
		margin-right: 14rpx;
		flex-direction: row
	}

	.TUI-Extension-slot {
		width: 128rpx;
		height: 170rpx;
		margin-left: 26rpx;
		margin-right: 26rpx;
		margin-top: 24rpx;
	}

	.TUI-Extension-icon {
		width: 128rpx;
		height: 128rpx;
	}

	.TUI-sendMessage-btn {
		display: flex;
		align-items: center;
		margin: 0 10rpx;
	}

	.TUI-Emoji-area {
		width: 100vw;
		height: 450rpx;
		//   height: 0px;
	}

	.aniDispaly {
		animation: display 0.3s linear forwards;
	}

	@keyframes display {
		from {
			height: 0;
			opacity: 0;
		}

		to {
			height: 450rpx;
			opacity: 1;
		}
	}

	.disDisplay {
		animation: disdisplay 0.3s linear forwards;
		height: 0;
	}

	@keyframes disdisplay {
		from {
			height: 450rpx;
			opacity: 1;
		}

		to {
			height: 0;
			opacity: 0;
		}
	}

	.emoDispaly {
		display: block;
	}

	.emodisDisplay {
		animation: hidess 0.3s linear forwards;
	}

	@keyframes hidess {
		from {
			// display: block;
			opacity: 1;
		}

		to {
			opacity: 0;
		}
	}

	.TUI-Extension-slot-name {
		line-height: 34rpx;
		font-size: 24rpx;
		color: #333333;
		letter-spacing: 0;
		text-align: center;
	}

	.record-modal {
		height: 300rpx;
		width: 60vw;
		background-color: #000;
		opacity: 0.8;
		position: fixed;
		top: 670rpx;
		z-index: 9999;
		left: 20vw;
		border-radius: 24rpx;
		display: flex;
		flex-direction: column;
	}

	.record-modal .wrapper {
		display: flex;
		height: 200rpx;
		box-sizing: border-box;
		padding: 10vw;
	}

	.record-modal .wrapper .modal-loading {
		opacity: 1;
		width: 40rpx;
		height: 16rpx;
		border-radius: 4rpx;
		background-color: #006fff;
		animation: loading 2s cubic-bezier(0.17, 0.37, 0.43, 0.67) infinite;
	}

	.modal-title {
		text-align: center;
		color: #fff;
	}

	@keyframes loading {
		0% {
			transform: translate(0, 0);
		}

		50% {
			transform: translate(30vw, 0);
			background-color: #f5634a;
			width: 40px;
		}

		100% {
			transform: translate(0, 0);
		}
	}

	.emojiTrans-enter-from,
	.emojiTrans-leave-to {
		opacity: 0;
	}

	.emojiTrans-enter-to,
	.emojiTrans-leave-from {
		opacity: 1; //这个其实是每个元素的默认值，因此可以省略
		display: none;
	}

	.emojiTrans-enter-active,
	.emojiTrans-leave-active {
		transition: opacity 0.2s linear;
	}

	.popBox {
		height: 80vh;
		background-color: white;
		padding-top: 20upx;

		.btns {
			padding: 0 10upx;
			height: 60upx;
		}

		::v-deep .uni-indexed-list__menu {
			height: calc(80vh - 44px - 56px) !important;
		}
	}

	.sureBtn {
		width: 120upx;
		height: 60upx;
		border-radius: 30rpx;
		font-size: 28rpx;
		line-height: 60upx;
		text-align: center;
		border: 1upx solid #f4f4f4;
	}

	.isOK {
		background: #5677fc !important;
		box-shadow: 0 5px 7px 0 rgba(86, 119, 252, 0.2) !important;
		color: white;
	}
</style>
```
<!-- tabs:end -->
