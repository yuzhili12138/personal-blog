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


