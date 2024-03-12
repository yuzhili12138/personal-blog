### Axios 双token 无感刷新
```
// 双token 无感刷新
import axios,{AxiosRequestConfig} from 'axios';

interface PendingTask {
    config: AxiosRequestConfig
    resolve: Function
}
let refreshing = false;
const queue: PendingTask[] = [];

const axiosInstance = axios.create({
    //https://api.example.com/ 相同的前缀
    baseURL: import.meta.env.VITE_BASE_URL,
    timeout: 100 * 1000,
});
// POST传参序列化(添加请求拦截器)
axiosInstance.interceptors.request.use(function (config) {
    const accessToken = localStorage.getItem('access_token');
    if(accessToken) {
        config.headers.authorization = 'Bearer ' + accessToken;
    }
    return config;
})
axiosInstance.interceptors.response.use(
    (response) => {
        return response;
    },
    async (error) => {
        let { data, config } = error.response;

        if(refreshing) {
            return new Promise((resolve) => {
                queue.push({
                    config,
                    resolve
                });
            });
        }

        if (data.statusCode === 401 && !config.url.includes('/refresh')) {
            refreshing = true;
            
            const res = await refreshToken();

            if(res.status === 200) {

                queue.forEach(({config, resolve}) => {
                    resolve(axiosInstance(config))
                })

                return axiosInstance(config);
            } else {
                alert(data || '登录过期，请重新登录');
            }
        } else {
            return error.response;
        }
    }
)


async function refreshToken() {
    const res = await axiosInstance.get('/refresh', {
        params: {
          token: localStorage.getItem('refresh_token')
        }
    });
    localStorage.setItem('access_token', res.data.accessToken);
    localStorage.setItem('refresh_token', res.data.refreshToken);
    return res;
}

// 发送请求
export function post(url: string, params: any) {
    return new Promise((resolve, reject) => {
        axiosInstance
            .post(url, params)
            .then(
                (res) => {
                    resolve(res.data);
                },
                (err) => {
                    reject(err);
                }
            )
            .catch((err) => {
                reject(err);
            });
    });
}

export function get(url: string, params?: any) {
    return new Promise((resolve, reject) => {
        axiosInstance
            .get(url, params)
            .then((res) => {
                resolve(res.data);
            })
            .catch((err) => {
                reject(err);
            });
    });
}

export function postJson(url: string, data = {}) {
    return new Promise((resolve, reject) => {
        axiosInstance
            .post(url, data, {
                headers: {
                    'Content-Type': 'application/json',
                },
            })
            .then(
                (res) => {
                    if (res) {
                        resolve(res.data);
                    }
                },
                (err) => {
                    reject(err);
                }
            );
    });
}
export function postFormData(url: string, data = {}) {
    return new Promise((resolve, reject) => {
        axiosInstance
            .post(url, data, {
                headers: {
                    'Content-Type': 'multipart/form-data',
                    // 'Content-Type': 'application/x-www-form-urlencoded'
                },
            })
            .then(
                (res) => {
                    if (res) {
                        resolve(res.data);
                    }
                },
                (err) => {
                    reject(err);
                }
            );
    });
}

```

### 最简单Axios
```
import axios from 'axios';
const instance = axios.create({
    //https://api.example.com/ 相同的前缀
    baseURL: import.meta.env.VITE_BASE_URL,
    timeout: 100 * 1000,
    headers: {},
});
// POST传参序列化(添加请求拦截器)
instance.interceptors.request.use(
    (config) => {
        return config;
    },
    (err) => {
        return Promise.reject(err);
    }
);
// 返回状态判断(添加响应拦截器)
instance.interceptors.response.use(
    (res) => {
        if (res.status === 200) {
            return res.data;
        } else {
            return res;
        }
    },
    (err) => {
        return Promise.reject(err);
    }
);
// 发送请求
export function post(url: string, params: any) {
    return new Promise((resolve, reject) => {
        instance
            .post(url, params)
            .then(
                (res) => {
                    resolve(res.data);
                },
                (err) => {
                    reject(err);
                }
            )
            .catch((err) => {
                reject(err);
            });
    });
}

export function get(url: string, params?: any) {
    return new Promise((resolve, reject) => {
        instance
            .get(url, params)
            .then((res) => {
                resolve(res.data);
            })
            .catch((err) => {
                reject(err);
            });
    });
}

export function postJson(url: string, data = {}) {
    return new Promise((resolve, reject) => {
        instance
            .post(url, data, {
                headers: {
                    'Content-Type': 'application/json',
                },
            })
            .then(
                (res) => {
                    if (res) {
                        resolve(res.data);
                    }
                },
                (err) => {
                    reject(err);
                }
            );
    });
}
export function postFormData(url: string, data = {}) {
    return new Promise((resolve, reject) => {
        instance
            .post(url, data, {
                headers: {
                    'Content-Type': 'multipart/form-data',
                    // 'Content-Type': 'application/x-www-form-urlencoded'
                },
            })
            .then(
                (res) => {
                    if (res) {
                        resolve(res.data);
                    }
                },
                (err) => {
                    reject(err);
                }
            );
    });
}
```