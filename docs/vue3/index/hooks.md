#### 倒计时
```
/**
 * 倒计时，主要用于发送短信验证码倒计时
 * 利用到期时间做倒计时，解决倒计时在页面关闭后就被不会更新的痛点
 */
import { onMounted, ref } from "vue";
export function useCountDown() {
  const timeLeft = ref<number>(0);
  onMounted(() => {
    // 计算当前时间到可以重新获取时间中间相差多少秒
    if (getRetrieveTime()) {
      const diff = getRetrieveTime() - new Date().getTime();
      if (diff <= 0) {
        // 倒计时结束了，可以获取验证码
        timeLeft.value = 0;
      } else {
        // 倒计时没有结束，继续倒计时
        timeLeft.value = Math.ceil(diff / 1000);
        start();
      }
    }
  });
  /**
   * 存储可以重新获取的时间
   */
  const setRetrieveTime = (count: number) => {
    localStorage.setItem(
      "retrieveTime",
      new Date(new Date().getTime() + count * 1000).getTime() + ""
    );
  };
  /**
   * 获取可以重新获取的时间
   */
  const getRetrieveTime = () => {
    return Number(localStorage.getItem("retrieveTime") || "0");
  };
  /**
   * 启动循环
   */
  const start = () => {
    setTimeout(() => {
      if (timeLeft.value > 0) {
        timeLeft.value--;
        start();
      }
    }, 1000);
  };

  /**
   * 点击开启倒计时
   * @param count 倒计时的时间，单位秒（s）
   */
  const startCountDown = (count: number) => {
    timeLeft.value = count;
    setRetrieveTime(count);
    start();
  };
  return {
    timeLeft,
    startCountDown,
  };
}
```

#### 拖动元素 moveDom.hook.ts
```
import { ref, onMounted, onUnmounted } from 'vue'

/**
 * 示例
 * <div  ref="chatbox"></div>
 * <div  @mousedown="dragx($event)"></div>
 * 拖动第二个div移动第一个div元素
 */
export const moveDom = () => {
	// 被拖动的包围盒
	const chatbox = ref()
	// 拖动的元素
	const dragx = (el: any) => {
		const oDiv = chatbox.value //当前元素
		const disX = el.clientX - oDiv.offsetLeft
		const disY = el.clientY - oDiv.offsetTop
		document.onmousemove = function (e) {
			//通过事件委托，计算移动的距离
			let l = e.clientX - disX
			let t = e.clientY - disY
			if (l < 0) {
				//如果左侧的距离小于0，就让距离等于0.不能超出屏幕左侧。如果需要磁性吸附，把0改为100或者想要的数字即可
				l = 0
			} else if (l > document.documentElement.clientWidth - oDiv.offsetWidth) {
				//如果左侧的距离>屏幕的宽度-元素的宽度。也就是说元素的右侧超出屏幕的右侧，就让元素的右侧在屏幕的右侧上
				l = document.documentElement.clientWidth - oDiv.offsetWidth
			}
			if (t < 0) {
				//和左右距离同理
				t = 0
			} else if (t > document.documentElement.clientHeight - oDiv.offsetHeight) {
				t = document.documentElement.clientHeight - oDiv.offsetHeight
			}
			//移动当前元素
			oDiv.style.left = l + 'px'
			oDiv.style.top = t + 'px'
		}
		document.onmouseup = function (e) {
			document.onmousemove = null
			document.onmouseup = null
		}
		// 解决有些时候,在鼠标松开的时候,元素仍然可以拖动;
		document.ondragstart = function (ev) {
			ev.preventDefault()
		}
		document.ondragend = function (ev) {
			ev.preventDefault()
		}
		return false
	}

	return {
		chatbox,
		dragx
	}
}

```





#### 屏幕适配

<!-- tabs:start -->
#### **hook/index.ts**
``` ts
export * from '@/hooks/moveDom.hook'
export * from '@/hooks/usePreviewScale.hook'
export * from '@/hooks/usePoints.hooks'

```
#### **useScale.hook.ts**
``` ts
import { ref, onMounted, onUnmounted } from 'vue'
import {
	usePreviewFitScale,
	usePreviewScrollYScale,
	usePreviewScrollXScale,
	usePreviewFullScale
} from '@/hooks/index'

export const useScale = (fScaleType: any) => {
	const entityRef = ref()
	const previewRef = ref()
	const width = ref(3840)
	const height = ref(2160)

	// 屏幕适配
	onMounted(() => {
		switch (fScaleType) {
			case 'fit':
				;(() => {
					const { calcRate, windowResize, unWindowResize } = usePreviewFitScale(
						width.value as number,
						height.value as number,
						previewRef.value
					)
					calcRate()
					windowResize()
					onUnmounted(() => {
						unWindowResize()
					})
				})()
				break
			case 'scrollY':
				;(() => {
					const { calcRate, windowResize, unWindowResize } = usePreviewScrollYScale(
						width.value as number,
						height.value as number,
						previewRef.value,
						(scale) => {
							const dom = entityRef.value
							dom.style.width = `${width.value * scale.width}px`
							dom.style.height = `${height.value * scale.height}px`
						}
					)
					calcRate()
					windowResize()
					onUnmounted(() => {
						unWindowResize()
					})
				})()

				break
			case 'scrollX':
				;(() => {
					const { calcRate, windowResize, unWindowResize } = usePreviewScrollXScale(
						width.value as number,
						height.value as number,
						previewRef.value,
						(scale) => {
							const dom = entityRef.value
							dom.style.width = `${width.value * scale.width}px`
							dom.style.height = `${height.value * scale.height}px`
						}
					)
					calcRate()
					windowResize()
					onUnmounted(() => {
						unWindowResize()
					})
				})()

				break
			case 'full':
				;(() => {
					const { calcRate, windowResize, unWindowResize } = usePreviewFullScale(
						width.value as number,
						height.value as number,
						previewRef.value
					)
					calcRate()
					windowResize()
					onUnmounted(() => {
						unWindowResize()
					})
				})()
				break
		}
	})

	return {
		entityRef,
		previewRef
	}
}

```
#### **usePoints.hooks.ts**
``` ts
import { throttle } from 'lodash'

// 拆出来是为了更好的分离单独复用

// * 屏幕缩放适配（两边留白）
export const usePreviewFitScale = (
	width: number,
	height: number,
	scaleDom: HTMLElement | null,
	callback?: (scale: { width: number; height: number }) => void
) => {
	// * 画布尺寸（px）
	const baseWidth = width
	const baseHeight = height

	// * 默认缩放值
	const scale = {
		width: 1,
		height: 1
	}

	// * 需保持的比例
	const baseProportion = parseFloat((baseWidth / baseHeight).toFixed(5))
	const calcRate = () => {
		// 当前屏幕宽高比
		const currentRate = parseFloat((window.innerWidth / window.innerHeight).toFixed(5))
		if (scaleDom) {
			if (currentRate > baseProportion) {
				// 表示更宽
				scale.width = parseFloat(((window.innerHeight * baseProportion) / baseWidth).toFixed(5))
				scale.height = parseFloat((window.innerHeight / baseHeight).toFixed(5))
				scaleDom.style.transform = `scale(${scale.width}, ${scale.height})`
			} else {
				// 表示更高
				scale.height = parseFloat((window.innerWidth / baseProportion / baseHeight).toFixed(5))
				scale.width = parseFloat((window.innerWidth / baseWidth).toFixed(5))
				scaleDom.style.transform = `scale(${scale.width}, ${scale.height})`
			}
			if (callback) callback(scale)
		}
	}

	const resize = throttle(() => {
		calcRate()
	}, 200)

	// * 改变窗口大小重新绘制
	const windowResize = () => {
		window.addEventListener('resize', resize)
	}

	// * 卸载监听
	const unWindowResize = () => {
		window.removeEventListener('resize', resize)
	}

	return {
		calcRate,
		windowResize,
		unWindowResize
	}
}

// *  X轴撑满，Y轴滚动条
export const usePreviewScrollYScale = (
	width: number,
	height: number,
	scaleDom: HTMLElement | null,
	callback?: (scale: { width: number; height: number }) => void
) => {
	// * 画布尺寸（px）
	const baseWidth = width
	const baseHeight = height

	// * 默认缩放值
	const scale = {
		width: 1,
		height: 1
	}

	// * 需保持的比例
	const baseProportion = parseFloat((baseWidth / baseHeight).toFixed(5))
	const calcRate = () => {
		if (scaleDom) {
			scale.height = parseFloat((window.innerWidth / baseProportion / baseHeight).toFixed(5))
			scale.width = parseFloat((window.innerWidth / baseWidth).toFixed(5))
			scaleDom.style.transform = `scale(${scale.width}, ${scale.height})`
			if (callback) callback(scale)
		}
	}

	const resize = throttle(() => {
		calcRate()
	}, 200)

	// * 改变窗口大小重新绘制
	const windowResize = () => {
		window.addEventListener('resize', resize)
	}

	// * 卸载监听
	const unWindowResize = () => {
		window.removeEventListener('resize', resize)
	}

	return {
		calcRate,
		windowResize,
		unWindowResize
	}
}

// *  Y轴撑满，X轴滚动条
export const usePreviewScrollXScale = (
	width: number,
	height: number,
	scaleDom: HTMLElement | null,
	callback?: (scale: { width: number; height: number }) => void
) => {
	// * 画布尺寸（px）
	const baseWidth = width
	const baseHeight = height

	// * 默认缩放值
	const scale = {
		height: 1,
		width: 1
	}

	// * 需保持的比例
	const baseProportion = parseFloat((baseWidth / baseHeight).toFixed(5))
	const calcRate = () => {
		if (scaleDom) {
			scale.width = parseFloat(((window.innerHeight * baseProportion) / baseWidth).toFixed(5))
			scale.height = parseFloat((window.innerHeight / baseHeight).toFixed(5))
			scaleDom.style.transform = `scale(${scale.width}, ${scale.height})`
			if (callback) callback(scale)
		}
	}

	const resize = throttle(() => {
		calcRate()
	}, 200)

	// * 改变窗口大小重新绘制
	const windowResize = () => {
		window.addEventListener('resize', resize)
	}

	// * 卸载监听
	const unWindowResize = () => {
		window.removeEventListener('resize', resize)
	}

	return {
		calcRate,
		windowResize,
		unWindowResize
	}
}

// * 变形内容，宽高铺满
export const usePreviewFullScale = (
	width: number,
	height: number,
	scaleDom: HTMLElement | null,
	callback?: (scale: { width: number; height: number }) => void
) => {
	// * 默认缩放值
	const scale = {
		width: 1,
		height: 1
	}

	const calcRate = () => {
		if (scaleDom) {
			scale.width = parseFloat((window.innerWidth / width).toFixed(5))
			scale.height = parseFloat((window.innerHeight / height).toFixed(5))
			scaleDom.style.transform = `scale(${scale.width}, ${scale.height})`
			if (callback) callback(scale)
		}
	}

	const resize = throttle(() => {
		calcRate()
	}, 200)

	// * 改变窗口大小重新绘制
	const windowResize = () => {
		window.addEventListener('resize', resize)
	}

	// * 卸载监听
	const unWindowResize = () => {
		window.removeEventListener('resize', resize)
	}

	return {
		calcRate,
		windowResize,
		unWindowResize
	}
}


```

#### **使用方法**
``` vue
<!-- scaleScreen/index.vue -->
<template>
	<!-- 实体层 -->
	<div class="fit" :style="fitStyle" ref="entityRef">
		<!-- 缩放层 -->
		<div class="scaleFit" id="scaleFit" ref="previewRef">
			<slot></slot>
		</div>
	</div>
</template>

<script lang="ts" setup>
import { useScale } from '@/hooks/useScale.hook'
const SCREEN_ZOOM = import.meta.env.VITE_SCREEN_ZOOM


const { entityRef, previewRef } =
	SCREEN_ZOOM === 'true'
		? useScale('full')
		: {
				entityRef: null,
				previewRef: null
			}

const fitStyle = reactive<any>(
	SCREEN_ZOOM == 'true'
		? {
				height: '100vh',
				display: 'flex'
			}
		: {
				height: 'auto',
				display: 'block'
			}
)
</script>
<style lang="scss" scoped>
.fit {
	display: flex;
	align-items: center;
	justify-content: center;
	pointer-events: none;
}

.scaleFit {
	transform-origin: center center;
	pointer-events: none;
	/* overflow: hidden; */
}
</style>



<!-- App.vue -->
<template>
	<NConfigProvider
		:locale="zhCN"
		:date-locale="dateZhCN"
		:theme="darkTheme"
		:theme-overrides="themeOverrides"
	>
		<n-message-provider>
			<n-dialog-provider>
				<ScaleScreen>
					<RouterView />
				</ScaleScreen>
				<Common />
			</n-dialog-provider>
		</n-message-provider>
	</NConfigProvider>
</template>
```
<!-- tabs:end -->



